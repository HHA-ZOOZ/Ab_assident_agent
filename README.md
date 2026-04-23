# Ab_assident_agent

> Ableton Live 전문 음악 프로듀서 AI 에이전트.  
> NullClaw + OpenClaw + pi-mono + Hermes 4개 프레임워크의 우수 패턴을 융합하여 구축.  
> **Kimi-K2.6** (Moonshot AI)을 기본 LLM으로 사용. **Claude Code CLI**로 단계적 구축.

---

## 빠른 시작

```bash
# 1. 환경변수 설정
cp .env.example .env
# .env 파일에서 MOONSHOT_API_KEY 입력

# 2. 의존성 설치 (uv 사용)
uv venv venv --python 3.11 && source venv/bin/activate
uv pip install -e .

# 3. 실행
python main.py cli        # 터미널 인터페이스
python main.py telegram   # 텔레그램 봇
python main.py hub        # 컴포넌트 관리 콘솔
```

---

## 문서

| 문서 | 설명 |
|---|---|
| [docs/architecture.md](docs/architecture.md) | 4개 프레임워크 분석 + 융합 아키텍처 설계 |
| [docs/build-order.md](docs/build-order.md) | Phase 0~14 단계적 구축 순서 |
| [docs/claude-code-instructions.md](docs/claude-code-instructions.md) | **Claude Code CLI 단계별 구축 지시 프롬프트** |
| [docs/env-setup.md](docs/env-setup.md) | Kimi-K2.6 API 환경변수 설정 가이드 |
| [docs/repos.md](docs/repos.md) | 참조 저장소 전체 목록 |

---

## 참조 프레임워크

| 프레임워크 | 저장소 | 이식 포인트 |
|---|---|---|
| **NullClaw** | [nullclaw/nullclaw](https://github.com/nullclaw/nullclaw) | 워크플로우 노드, 체크포인트, KV 스토어 |
| **OpenClaw** | [openclaw.ai](https://openclaw.ai) | SOUL.md 페르소나, agentskills.io 스킬 표준 |
| **pi-mono** | [badlogic/pi-mono](https://github.com/badlogic/pi-mono) | LLM 추상화, 에이전트 코어, TUI |
| **Hermes** | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | 자기개선 루프, 메모리, cron, 게이트웨이 |