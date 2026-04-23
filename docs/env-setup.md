# 환경변수 설정 가이드

> Ab_assident_agent는 **Kimi-K2.6**을 기본 LLM으로 사용한다.  
> Claude Code CLI가 에이전트를 오케스트레이션하고,  
> Kimi-K2.6 API가 실제 음악 생성·분석 추론을 담당한다.

---

## 1. 역할 분리

```
Claude Code CLI  →  에이전트 구축·코드 작성·도구 호출 오케스트레이션
Kimi-K2.6 API   →  실제 추론 (음악 생성, 분석, 스킬 생성 등)
```

Kimi-K2.6은 Moonshot AI의 OpenAI 호환 엔드포인트를 통해 접근한다.

---

## 2. 필수 환경변수

### 2-1. `.env` 파일 생성

프로젝트 루트에 `.env` 파일을 생성한다 (절대 커밋하지 않는다):

```bash
cp .env.example .env
```

### 2-2. 환경변수 목록

```bash
# ─────────────────────────────────────────
# Kimi-K2.6 (기본 LLM)
# ─────────────────────────────────────────
# Moonshot AI API 키 (https://platform.moonshot.ai)
MOONSHOT_API_KEY=sk-...

# Kimi API 엔드포인트 (OpenAI 호환)
KIMI_BASE_URL=https://api.moonshot.cn/v1

# 사용할 모델명
KIMI_MODEL=kimi-k2-0528

# ─────────────────────────────────────────
# 보조 LLM (선택)
# ─────────────────────────────────────────
ANTHROPIC_API_KEY=sk-ant-...       # Claude (복잡한 코드 분석용)
OPENAI_API_KEY=sk-...              # GPT-4o (대안)
OPENROUTER_API_KEY=sk-or-...       # OpenRouter (200+ 모델 라우팅)

# ─────────────────────────────────────────
# 메시징 게이트웨이 (선택)
# ─────────────────────────────────────────
TELEGRAM_BOT_TOKEN=...
DISCORD_BOT_TOKEN=...

# ─────────────────────────────────────────
# 메모리 백엔드
# ─────────────────────────────────────────
# SQLite (기본값 — 별도 설정 불필요)
# Qdrant 벡터 DB (선택)
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=

# Honcho (사용자 모델링, 선택)
HONCHO_API_KEY=

# ─────────────────────────────────────────
# Ableton / DSP
# ─────────────────────────────────────────
ABLETON_OSC_HOST=127.0.0.1
ABLETON_OSC_SEND_PORT=11000        # Ab→Ableton
ABLETON_OSC_RECEIVE_PORT=11001     # Ableton→Ab

# ─────────────────────────────────────────
# 에이전트 설정
# ─────────────────────────────────────────
AB_HOME=~/.ab_assident             # 에이전트 데이터 디렉토리
AB_LOG_LEVEL=INFO
AB_DEFAULT_PROVIDER=kimi           # kimi | anthropic | openai | openrouter
```

---

## 3. `.env.example` 파일 (커밋용 템플릿)

> 실제 키 없이 키 이름만 기재된 템플릿. 저장소에 커밋한다.

```bash
# Kimi-K2.6 (필수)
MOONSHOT_API_KEY=
KIMI_BASE_URL=https://api.moonshot.cn/v1
KIMI_MODEL=kimi-k2-0528

# 보조 LLM (선택)
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
OPENROUTER_API_KEY=

# 메시징 (선택)
TELEGRAM_BOT_TOKEN=
DISCORD_BOT_TOKEN=

# 메모리 백엔드 (선택)
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=
HONCHO_API_KEY=

# Ableton OSC
ABLETON_OSC_HOST=127.0.0.1
ABLETON_OSC_SEND_PORT=11000
ABLETON_OSC_RECEIVE_PORT=11001

# 에이전트
AB_HOME=~/.ab_assident
AB_LOG_LEVEL=INFO
AB_DEFAULT_PROVIDER=kimi
```

---

## 4. API 키 발급 방법

### 4-1. Kimi-K2.6 (Moonshot AI) — 필수

1. https://platform.moonshot.ai 에서 가입
2. **API Keys** → **Create new key**
3. `MOONSHOT_API_KEY`에 입력
4. 요금 확인: https://platform.moonshot.ai/docs/pricing

```bash
# 연결 테스트
curl https://api.moonshot.cn/v1/chat/completions \
  -H "Authorization: Bearer $MOONSHOT_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "kimi-k2-0528",
    "messages": [{"role": "user", "content": "ping"}],
    "max_tokens": 10
  }'
```

### 4-2. Anthropic Claude — 선택 (복잡한 코드 작성용)

1. https://console.anthropic.com 에서 가입
2. **API Keys** → **Create Key**
3. `ANTHROPIC_API_KEY`에 입력

### 4-3. OpenRouter — 선택 (모델 라우팅)

1. https://openrouter.ai 에서 가입
2. **Keys** → **Create Key**
3. `OPENROUTER_API_KEY`에 입력

---

## 5. 환경변수 로딩 방법

### 5-1. Python (python-dotenv)

```python
from dotenv import load_dotenv
load_dotenv()

import os
MOONSHOT_API_KEY = os.environ["MOONSHOT_API_KEY"]
```

### 5-2. Shell

```bash
source .env  # 또는
set -a && source .env && set +a
```

### 5-3. Claude Code CLI에서 사용 시

```bash
# Claude Code를 실행하기 전에 환경변수 로드
set -a && source .env && set +a
claude
```

---

## 6. Kimi-K2.6 OpenAI 호환 설정

Kimi API는 OpenAI SDK와 완전 호환된다. `packages/ai/providers/kimi.py`에서:

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["MOONSHOT_API_KEY"],
    base_url=os.environ.get("KIMI_BASE_URL", "https://api.moonshot.cn/v1"),
)

model = os.environ.get("KIMI_MODEL", "kimi-k2-0528")
```

LangChain을 통한 접근:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model=os.environ.get("KIMI_MODEL", "kimi-k2-0528"),
    openai_api_key=os.environ["MOONSHOT_API_KEY"],
    openai_api_base=os.environ.get("KIMI_BASE_URL", "https://api.moonshot.cn/v1"),
    temperature=0.7,
)
```

---

## 7. `.gitignore` 확인

`.env` 파일이 절대 커밋되지 않도록:

```gitignore
.env
.env.local
*.env
__pycache__/
venv/
.venv/
*.pyc
~/.ab_assident/
```
