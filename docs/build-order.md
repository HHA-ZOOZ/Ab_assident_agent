# Ab_assident_agent — 단계적 구축 순서

> NullClaw + OpenClaw + pi-mono + Hermes 융합 아키텍처를  
> 기반으로 Ab_assident_agent를 단계적으로 구축하는 순서.

---

## Phase 0 — 환경 준비 (Prerequisites)

### 0-1. 시스템 요구사항
- Python 3.11+
- Node.js 20+ (pi-tui, 모노레포 도구)
- uv (Python 패키지 관리)
- Ableton Live 11/12 + Max for Live
- Git

### 0-2. 저장소 클론 및 의존성

```bash
# 본 저장소
git clone https://github.com/HHA-ZOOZ/Ab_assident_agent.git
cd Ab_assident_agent

# Python 환경 (Hermes 방식)
curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv venv --python 3.11
source venv/bin/activate

# 참조 저장소 (로컬 학습용, 빌드 불필요)
git clone https://github.com/NousResearch/hermes-agent.git /tmp/hermes-agent
git clone https://github.com/badlogic/pi-mono.git /tmp/pi-mono
git clone https://github.com/nullclaw/nullboiler.git /tmp/nullboiler
git clone https://github.com/nullclaw/nulltickets.git /tmp/nulltickets
```

### 0-3. 핵심 Python 의존성 설치

```bash
uv pip install \
  langgraph \
  langchain-anthropic \
  langchain-openai \
  python-telegram-bot \
  discord.py \
  honcho-ai \
  pythonosc \
  sqlalchemy \
  qdrant-client \
  rich \
  typer \
  pydantic-settings \
  httpx \
  apscheduler
```

---

## Phase 1 — 에이전트 정체성 파일 구성 (OpenClaw 패턴)

### 1-1. SOUL.md 작성 — 에이전트 페르소나

> OpenClaw의 SOUL.md 패턴 적용.  
> 에이전트가 세션 시작 시 자신의 정체성을 확인하는 파일.

```markdown
# SOUL.md
## 정체성
나는 Ab_assident — Ableton Live 전문 음악 프로듀서 AI 에이전트.
...
```

**참조:** `docs/architecture.md` § 1-2

### 1-2. MEMORY.md — 초기 영속 메모리 템플릿

### 1-3. USER.md — 사용자 프로파일 템플릿

### 1-4. AGENTS.md — 에이전트 작업 규칙 (pi-mono/NullClaw 패턴)

---

## Phase 2 — 멀티프로바이더 LLM 추상화 (pi-ai 패턴)

**목표:** 런타임에 LLM 제공자를 교체 가능하게 만든다.

### 2-1. `packages/ai/` 구조 생성

```bash
mkdir -p packages/ai/providers
```

### 2-2. 프로바이더 구현

- `packages/ai/providers/kimi.py` — Kimi-K2.6 (음악 생성 특화)
- `packages/ai/providers/anthropic.py` — Claude Opus (복잡한 분석)
- `packages/ai/providers/openai.py` — GPT-4o (보조)
- `packages/ai/base.py` — 추상 인터페이스 (pi-ai 패턴)

### 2-3. 프로바이더 팩토리

```python
# packages/ai/factory.py
def get_llm(provider: str = "kimi") -> BaseLLM:
    ...
```

**참조:** `badlogic/pi-mono` `packages/ai/`

---

## Phase 3 — 에이전트 코어 + 상태 머신 (pi-agent-core + NullClaw)

**목표:** tool calling 루프와 상태 관리를 구현한다.

### 3-1. `packages/agent-core/` 구조 생성

```bash
mkdir -p packages/agent-core
```

### 3-2. 상태 관리

- `packages/agent-core/state.py` — 현재 세션/트랙/컨텍스트 상태 (TypedDict)
- `packages/agent-core/tool_loop.py` — ReAct 스타일 도구 호출 루프

### 3-3. 인터럽트 + 체크포인트 (NullBoiler 패턴)

- `packages/agent-core/interrupt.py` — human-in-the-loop 브레이크포인트
- `packages/agent-core/checkpoint.py` — 실행 상태 저장/복원/포킹

**참조:**
- `badlogic/pi-mono` `packages/agent/src/`
- `nullclaw/nullboiler` — interrupt, checkpoint forking

---

## Phase 4 — 워크플로우 오케스트레이터 (NullBoiler → LangGraph)

**목표:** NullBoiler의 워크플로우 그래프 노드 개념을 LangGraph로 구현한다.

### 4-1. `packages/orchestrator/` 구조 생성

```bash
mkdir -p packages/orchestrator/nodes
```

### 4-2. 노드 타입 구현

| 노드 | 파일 | NullBoiler 원형 |
|---|---|---|
| 분석/생성 노드 | `nodes/task_node.py` | `task` |
| 서브에이전트 노드 | `nodes/agent_node.py` | `agent` |
| 조건 분기 | `nodes/route_node.py` | `route` |
| 인터럽트 | `nodes/interrupt_node.py` | `interrupt` |
| 병렬 fan-out | `nodes/send_node.py` | `send` |
| 출력 변환 | `nodes/transform_node.py` | `transform` |
| 서브그래프 | `nodes/subgraph_node.py` | `subgraph` |

### 4-3. 워크플로우 그래프 조립

- `packages/orchestrator/workflow_graph.py` — LangGraph StateGraph 조립
- `packages/orchestrator/store_updates.py` — 워크플로우 결과 → 영속 메모리 쓰기

**참조:** `nullclaw/nullboiler` README — Workflow Graph Features

---

## Phase 5 — 태스크 트래커 + 영속 KV 스토어 (NullTickets 패턴)

**목표:** 세션 간 태스크와 상태를 영속화한다.

### 5-1. `packages/tracker/` 구조 생성

```bash
mkdir -p packages/tracker
```

### 5-2. 구현 항목

- `packages/tracker/task_queue.py` — 태스크 생성/완료/우선순위 관리
- `packages/tracker/kv_store.py` — SQLite 기반 영속 KV 스토어
- `packages/tracker/fts.py` — FTS5 전문 검색 (Hermes 세션 검색 패턴)

**참조:** `nullclaw/nulltickets`

---

## Phase 6 — 메모리 시스템 (Hermes + OpenClaw)

**목표:** 에피소드 메모리, 의미 메모리, 사용자 모델을 구현한다.

### 6-1. `packages/memory/` 구조 생성

```bash
mkdir -p packages/memory
```

### 6-2. 구현 항목

- `packages/memory/episodic.py` — FTS5 기반 세션 기억 + LLM 요약 (Hermes 패턴)
- `packages/memory/semantic.py` — Qdrant 벡터 DB 의미 메모리
- `packages/memory/user_model.py` — Honcho dialectic 사용자 취향 모델링
- `packages/memory/skill_memory.py` — 스킬 사용 패턴 학습

**참조:**
- `NousResearch/hermes-agent` — FTS5 세션 검색, Honcho 통합
- `plastic-labs/honcho`
- `ahonore42/logios-brain`

---

## Phase 7 — 스킬 시스템 (OpenClaw/Hermes/agentskills.io)

**목표:** agentskills.io 표준 포맷으로 스킬을 작성하고 자기개선 루프를 붙인다.

### 7-1. `packages/skills/` 구조 생성

```bash
mkdir -p packages/skills/builtin packages/skills/user packages/skills/self_improve
```

### 7-2. 내장 스킬 작성 (Markdown 포맷)

| 스킬 파일 | 내용 |
|---|---|
| `builtin/ableton_osc.md` | Ableton Live OSC 제어 (트랙/클립/파라미터) |
| `builtin/m4l_generate.md` | Max for Live 디바이스 생성 프로시저 |
| `builtin/vst3_create.md` | JUCE/Cmajor VST3 플러그인 생성 |
| `builtin/audio_analyze.md` | 레퍼런스 트랙 스펙트럼·구조 분석 |
| `builtin/mix_master.md` | 믹싱·마스터링 체인 구성 |

### 7-3. 자기개선 루프 (Hermes 패턴)

- `packages/skills/self_improve/skill_creator.py` — 태스크 완료 후 스킬 자동 생성
- `packages/skills/self_improve/skill_improver.py` — 스킬 사용 중 피드백 기반 개선

**참조:** `NousResearch/hermes-agent` — Skills System

---

## Phase 8 — DSP 엔진 연동 (Ableton OSC + M4L + VST3)

**목표:** Ableton Live와 실제 연동하는 DSP 도구 레이어를 구현한다.

### 8-1. `packages/dsp/` 구조 생성

```bash
mkdir -p packages/dsp
```

### 8-2. 구현 항목

- `packages/dsp/ableton_osc.py` — python-osc 기반 Ableton Live 양방향 제어
- `packages/dsp/m4l_generator.py` — Max Patch (`.amxd`) 자동 생성
- `packages/dsp/vst3_builder.py` — JUCE CMake 기반 VST3 빌드 자동화
- `packages/dsp/cmajor_engine.py` — Cmajor JIT 컴파일러 연동

### 8-3. 도구 등록

DSP 함수들을 LangChain Tool / LangGraph ToolNode로 래핑하여 에이전트가 호출 가능하게 등록.

---

## Phase 9 — 멀티플랫폼 게이트웨이 (OpenClaw/Hermes 패턴)

**목표:** Telegram, Discord, CLI 인터페이스를 동일한 에이전트 코어에 연결한다.

### 9-1. `packages/gateway/` 구조 생성

```bash
mkdir -p packages/gateway
```

### 9-2. 구현 항목

- `packages/gateway/cli.py` — Typer + Rich 기반 터미널 인터페이스 (pi-coding-agent 패턴)
- `packages/gateway/telegram.py` — python-telegram-bot 기반 봇
- `packages/gateway/discord.py` — discord.py 기반 봇

### 9-3. 공통 게이트웨이 인터페이스

```python
# packages/gateway/base.py
class GatewayBase:
    async def send_message(self, text: str, ...) -> None: ...
    async def receive_message(self) -> Message: ...
```

**참조:**
- `NousResearch/hermes-agent` — Messaging Gateway
- `nullclaw/nullclaw` — SIGNAL.md

---

## Phase 10 — TUI 터미널 UI (pi-tui 패턴)

**목표:** Ableton 상태를 실시간으로 터미널에 표시하는 UI를 구현한다.

### 10-1. `packages/tui/` 구조 생성

```bash
mkdir -p packages/tui
```

### 10-2. 구현 항목

- `packages/tui/renderer.py` — Rich Live 기반 차분 렌더링
- `packages/tui/ableton_status.py` — 현재 트랙/BPM/클립 상태 패널
- `packages/tui/skill_panel.py` — 활성 스킬 표시 패널

**참조:** `badlogic/pi-mono` `packages/tui/`

---

## Phase 11 — 컴포넌트 관리 허브 (NullHub 패턴)

**목표:** 모든 서비스 컴포넌트를 매니페스트로 등록하고 감독한다.

### 11-1. `packages/hub/` 구조 생성

```bash
mkdir -p packages/hub
```

### 11-2. 구현 항목

- `ab-manifest.json` — 컴포넌트 등록 파일 (NullHub manifest 패턴)
- `packages/hub/process_supervisor.py` — 서브프로세스 감독 + 크래시 복구
- `packages/hub/health_check.py` — 주기적 HTTP 헬스체크
- `packages/hub/log_stream.py` — 컴포넌트 로그 스트리밍

**참조:** `nullclaw/nullhub` — Architecture, CLI Usage

---

## Phase 12 — cron 스케줄러 (Hermes 패턴)

**목표:** 자연어로 정의된 반복 자동화 태스크를 실행한다.

### 12-1. `cron/` 구조 생성

```bash
mkdir -p cron
```

### 12-2. 구현 항목

- `cron/schedules.json` — 스케줄 정의 (예: 매일 오전 9시 레퍼런스 분석)
- `cron/scheduler.py` — APScheduler 기반 실행 엔진

**예시 스케줄:**
```json
{
  "schedules": [
    {
      "id": "daily-reference-analysis",
      "cron": "0 9 * * *",
      "task": "레퍼런스 폴더에서 새 트랙 분석하고 MEMORY.md 업데이트",
      "deliver_to": "telegram"
    }
  ]
}
```

**참조:** `NousResearch/hermes-agent` — Cron Scheduling

---

## Phase 13 — RL 학습 환경 (Hermes/Atropos 패턴) [선택]

**목표:** 음악 완성도를 reward로 삼는 RL 환경을 구성한다.

### 13-1. `rl/` 구조 생성

```bash
mkdir -p rl
```

### 13-2. 구현 항목

- `rl/trajectory.py` — 에이전트 실행 trajectory 기록
- `rl/reward.py` — 음악 완성도 reward 모델 (사용자 피드백 + 음향 지표)
- `rl/atropos_env.py` — Atropos 호환 RL 환경

**참조:** `NousResearch/hermes-agent` — Research, Atropos RL

---

## Phase 14 — 통합 테스트 + 첫 실행

### 14-1. 컴포넌트 헬스체크

```bash
python -m packages.hub.health_check
```

### 14-2. CLI로 첫 대화

```bash
python -m packages.gateway.cli
```

### 14-3. Ableton 연동 테스트

```bash
# Ableton Live를 열고 OSC 수신 설정 후:
python -m packages.dsp.ableton_osc --test
```

### 14-4. 텔레그램 게이트웨이 시작

```bash
python -m packages.gateway.telegram
```

---

## 구축 단계 요약 (체크리스트)

```
Phase 0  — 환경 준비                         [ ]
Phase 1  — SOUL.md + 정체성 파일              [ ]
Phase 2  — 멀티프로바이더 LLM 추상화           [ ]
Phase 3  — 에이전트 코어 + 상태 머신           [ ]
Phase 4  — 워크플로우 오케스트레이터           [ ]
Phase 5  — 태스크 트래커 + KV 스토어          [ ]
Phase 6  — 메모리 시스템                      [ ]
Phase 7  — 스킬 시스템 + 자기개선 루프         [ ]
Phase 8  — DSP 엔진 연동 (Ableton/M4L/VST3)  [ ]
Phase 9  — 멀티플랫폼 게이트웨이              [ ]
Phase 10 — TUI 터미널 UI                     [ ]
Phase 11 — 컴포넌트 관리 허브                 [ ]
Phase 12 — cron 스케줄러                     [ ]
Phase 13 — RL 학습 환경 (선택)               [ ]
Phase 14 — 통합 테스트 + 첫 실행              [ ]
```
