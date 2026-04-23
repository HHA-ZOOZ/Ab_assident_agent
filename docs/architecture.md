# Ab_assident_agent — 융합 아키텍처 설계

> 4개 오픈소스 에이전트 프레임워크(NullClaw, OpenClaw, pi-mono, Hermes)의  
> 우수 패턴을 분석·이식하여 Ab_assident_agent를 구축하기 위한 아키텍처 문서.

---

## 1. 참조 프레임워크 개요

### 1-1. NullClaw 에코시스템 (Zig)

```
nullclaw    → 에이전트 실행 런타임 (Executor)
nullboiler  → 오케스트레이션 정책 엔진 (Policy Engine)
nulltickets → 태스크 트래커 + 영속 KV 스토어 (Source of Truth)
nullhub     → 관리 콘솔 + 프로세스 감독 (Management UI)
```

**핵심 원칙:**
- `tracker = source of truth`
- `orchestrator = policy engine`
- `agent = executor`

**이식 포인트:**
| 컴포넌트 | 원본 패턴 | Ab_assident 적용 |
|---|---|---|
| NullBoiler | `task/agent/route/interrupt/send/transform/subgraph` 노드 | LangGraph 노드 타입 설계 기반 |
| NullBoiler | 체크포인트 포킹 (checkpoint forking) | 음악 생성 중간 상태 저장·분기 |
| NullBoiler | `store_updates` — 워크플로우 결과 → 영속 메모리 | 생성 결과 자동 메모리화 |
| NullTickets | 태스크 큐 + KV 스토어 + FTS | 세션 간 태스크·기억 영속화 |
| NullHub | `nullhub-manifest.json` 매니페스트 기반 컴포넌트 | `ab-manifest.json`으로 서비스 등록/감독 |
| NullHub | 프로세스 감독 + 헬스체크 + SSE 로그 스트리밍 | 컴포넌트 상태 모니터링 |

---

### 1-2. OpenClaw (Python, Claw 패밀리 원조)

**핵심 파일 구조:**
```
~/.openclaw/
├── SOUL.md      ← 에이전트 페르소나/정체성
├── MEMORY.md    ← 영속 메모리 (학습된 사실)
├── USER.md      ← 사용자 모델
└── skills/      ← 스킬 라이브러리 (agentskills.io 표준)
```

**이식 포인트:**
| 컴포넌트 | 원본 패턴 | Ab_assident 적용 |
|---|---|---|
| SOUL.md | 에이전트 정체성 파일 | `SOUL.md` — 음악 프로듀서 + Ableton 전문가 페르소나 |
| agentskills.io | 범용 스킬 포맷 표준 | `skills/` 호환 포맷 — 커뮤니티 스킬 재사용 |
| Command allowlist | 위험 명령 승인 패턴 | Ableton/OS 명령 안전 승인 |
| 멀티플랫폼 게이트웨이 | Telegram/Discord/Slack/WhatsApp/Signal | `gateway/` 레이어 |

---

### 1-3. pi-mono (TypeScript, 모노레포)

**패키지 구성:**
```
packages/
├── ai/           ← 멀티프로바이더 LLM API (OpenAI, Anthropic, Google 통합)
├── agent/        ← 에이전트 코어 (tool calling + state management)
├── coding-agent/ ← 코딩 에이전트 CLI
├── tui/          ← 터미널 UI (차분 렌더링)
├── web-ui/       ← 웹 컴포넌트
├── pods/         ← vLLM GPU 배포 관리 CLI
└── mom/          ← Slack 봇
```

**이식 포인트:**
| 컴포넌트 | 원본 패턴 | Ab_assident 적용 |
|---|---|---|
| pi-ai | 교체 가능한 LLM 프로바이더 추상화 | `ai/providers/` (Kimi-K2/Claude/GPT 런타임 교체) |
| pi-agent-core | tool calling 상태 머신 | `agent-core/tool_loop.py` + `state.py` |
| pi-tui | 차분(differential) 렌더링 TUI | Ableton 상태 실시간 표시 |
| pi-pods | vLLM GPU 배포 관리 | DSP 전용 로컬 모델 배포 |
| 모노레포 | npm workspaces 기반 패키지 분리 | `packages/` 전체 구조 참조 |

---

### 1-4. Hermes (NousResearch, Python, 자기개선 에이전트)

**핵심 차별점 — 자기개선 루프:**
```
복잡한 태스크 완료 → 스킬 자동 생성
스킬 사용 중       → 스킬 자동 개선
세션 간            → FTS5 검색 + LLM 요약 기반 기억 recall
사용자 패턴        → Honcho dialectic 모델링 (취향 학습)
```

**이식 포인트:**
| 컴포넌트 | 원본 패턴 | Ab_assident 적용 |
|---|---|---|
| 자기개선 루프 | 태스크 완료 → 스킬 자동 생성·개선 | `skills/self_improve/` |
| Honcho | 사용자 취향·패턴 누적 학습 | `memory/user_model.py` |
| FTS5 세션 검색 | 과거 대화 full-text search + LLM 요약 | `memory/episodic.py` |
| 서브에이전트 위임 | 병렬 워크스트림 스폰 | orchestrator 병렬 에이전트 |
| cron 스케줄러 | 자연어 기반 예약 자동화 | `cron/schedules.json` |
| Atropos RL | trajectory 생성 + RL 미세조정 | `rl/` (음악 완성도 reward) |
| 멀티플랫폼 게이트웨이 | Telegram, Discord, CLI 통합 | `gateway/` 레이어 |

---

## 2. Ab_assident_agent 융합 파일 구조

```
ab_assident_agent/
│
├── SOUL.md                        ← [OpenClaw] 에이전트 페르소나
│                                     (음악 프로듀서 + Ableton 전문가 정체성)
├── MEMORY.md                      ← [OpenClaw/Hermes] 영속 메모리
├── USER.md                        ← [Hermes/Honcho] 사용자 취향 모델
├── AGENTS.md                      ← [pi-mono/NullClaw] 에이전트 작업 규칙
├── ab-manifest.json               ← [NullHub] 컴포넌트 등록 매니페스트
│
├── packages/                      ← [pi-mono] 모노레포 구조
│   │
│   ├── ai/                        ← [pi-ai] 멀티프로바이더 LLM 추상화
│   │   └── providers/
│   │       ├── kimi.py            ← Kimi-K2.6 (음악 생성 특화)
│   │       ├── anthropic.py       ← Claude Opus (복잡한 분석)
│   │       └── openai.py          ← GPT-4o (보조)
│   │
│   ├── agent-core/                ← [pi-agent-core + NullClaw] 에이전트 루프
│   │   ├── state.py               ← 상태 관리 (세션, 트랙, 컨텍스트)
│   │   ├── tool_loop.py           ← 도구 호출 루프
│   │   └── interrupt.py           ← [NullBoiler] 인터럽트/체크포인트
│   │
│   ├── orchestrator/              ← [NullBoiler] 워크플로우 오케스트레이터
│   │   ├── workflow_graph.py      ← LangGraph 기반 그래프 실행
│   │   ├── nodes/                 ← task/agent/route/send/transform 노드
│   │   │   ├── task_node.py
│   │   │   ├── route_node.py
│   │   │   ├── interrupt_node.py
│   │   │   ├── send_node.py       ← fan-out 병렬 실행
│   │   │   └── transform_node.py
│   │   ├── checkpoint.py          ← 실행 상태 저장/포킹
│   │   └── store_updates.py       ← 워크플로우 결과 → 영속 메모리
│   │
│   ├── tracker/                   ← [NullTickets] 태스크 + KV 스토어
│   │   ├── task_queue.py          ← 태스크 관리 (생성/완료/우선순위)
│   │   ├── kv_store.py            ← 세션 간 영속 스토어
│   │   └── fts.py                 ← [Hermes] FTS5 세션 검색
│   │
│   ├── memory/                    ← [Hermes + OpenClaw] 메모리 시스템
│   │   ├── episodic.py            ← 세션 기억 (Hermes FTS5)
│   │   ├── semantic.py            ← 의미 메모리 (벡터 DB)
│   │   ├── user_model.py          ← [Honcho] 사용자 취향 모델링
│   │   └── skill_memory.py        ← 스킬 사용 패턴 학습
│   │
│   ├── skills/                    ← [OpenClaw/Hermes/agentskills.io] 스킬
│   │   ├── builtin/               ← 내장 스킬 (agentskills.io 포맷)
│   │   │   ├── ableton_osc.md     ← Ableton OSC 제어
│   │   │   ├── m4l_generate.md    ← M4L 디바이스 생성
│   │   │   ├── vst3_create.md     ← VST3 플러그인 생성
│   │   │   ├── audio_analyze.md   ← 레퍼런스 트랙 분석
│   │   │   └── mix_master.md      ← 믹싱/마스터링
│   │   ├── user/                  ← 사용자 생성 스킬
│   │   └── self_improve/          ← [Hermes] 자동 생성·개선 스킬
│   │
│   ├── gateway/                   ← [OpenClaw/Hermes] 멀티플랫폼 게이트웨이
│   │   ├── telegram.py
│   │   ├── discord.py
│   │   └── cli.py                 ← [pi-coding-agent] 터미널 인터페이스
│   │
│   ├── tui/                       ← [pi-tui] 터미널 UI
│   │   ├── renderer.py            ← 차분 렌더링
│   │   └── ableton_status.py      ← Ableton 상태 실시간 패널
│   │
│   ├── hub/                       ← [NullHub] 관리 콘솔
│   │   ├── process_supervisor.py  ← 프로세스 감독 + 크래시 복구
│   │   ├── health_check.py        ← 주기적 헬스체크
│   │   └── log_stream.py          ← SSE 로그 스트리밍
│   │
│   └── dsp/                       ← [VST3 SDK + Cmajor + M4L] DSP 엔진
│       ├── m4l_generator.py       ← Max Patch 자동 생성
│       ├── vst3_builder.py        ← VST3 플러그인 빌드
│       └── cmajor_engine.py       ← Cmajor JIT 컴파일러
│
├── cron/                          ← [Hermes] 스케줄 자동화
│   └── schedules.json
│
└── rl/                            ← [Hermes/Atropos] RL 학습
    ├── trajectory.py              ← trajectory 생성
    └── reward.py                  ← 음악 완성도 reward 모델
```

---

## 3. 핵심 융합 포인트 상세

### 3-1. NullBoiler 노드 → LangGraph 노드 매핑

```
NullBoiler node type  →  LangGraph 구현
─────────────────────────────────────────
task                  →  분석/생성 StateNode
agent                 →  서브에이전트 ToolNode
route                 →  조건 분기 ConditionalEdge
interrupt             →  human-in-the-loop BreakPoint
send (fan-out)        →  병렬 Send API
transform             →  출력 변환 StateTransformNode
subgraph              →  재사용 가능한 SubGraph
```

### 3-2. Hermes 자기개선 루프 → 음악 특화 적응

```
Hermes 원본:
  복잡한 태스크 완료 → 스킬 자동 생성 → 사용 중 자동 개선

Ab_assident 적용:
  사용자: "808 킥 이런 방식으로 만들어줘"
       ↓
  에이전트 실행 완료 → M4L 생성 과정이 스킬로 저장
       ↓
  다음 유사 요청 → 저장 스킬 적용 + 개선 제안
       ↓
  Honcho 사용자 취향 누적 → 스타일 자동 예측·제안
```

### 3-3. NullHub 매니페스트 → Ab_assident 서비스 관리

```json
// ab-manifest.json
{
  "components": {
    "ableton-osc-bridge": {
      "launch": ["python", "-m", "packages.dsp.m4l_generator"],
      "health_check": "http://localhost:11000/health",
      "wizard_steps": ["Ableton Live 실행 확인", "OSC 포트 11000 설정"]
    },
    "orchestrator": {
      "launch": ["python", "-m", "packages.orchestrator.workflow_graph"],
      "health_check": "http://localhost:11002/health"
    },
    "gateway": {
      "launch": ["python", "-m", "packages.gateway.telegram"],
      "health_check": "http://localhost:11003/health"
    }
  }
}
```

---

## 4. 이식 우선순위 로드맵

| 우선순위 | 컴포넌트 | 출처 | 이유 |
|---|---|---|---|
| 🔴 P0 | `SOUL.md` + agentskills.io 스킬 포맷 | OpenClaw | 에이전트 정체성 + 스킬 재사용성 |
| 🔴 P0 | pi-ai 멀티프로바이더 추상화 | pi-mono | LLM 교체 가능성 — 기본 인프라 |
| 🔴 P0 | NullBoiler 노드 타입 → LangGraph | NullClaw | 워크플로우 설계 기반 |
| 🟡 P1 | Hermes 자기개선 스킬 루프 | Hermes | 장기적 품질 향상 핵심 |
| 🟡 P1 | NullTickets KV + FTS 검색 | NullClaw | 영속 메모리 + 세션 검색 |
| 🟡 P1 | pi-tui 차분 렌더링 TUI | pi-mono | Ableton 상태 실시간 시각화 |
| 🟡 P1 | Hermes 멀티플랫폼 게이트웨이 | Hermes | 텔레그램/디스코드/CLI 통합 |
| 🟢 P2 | Hermes cron 스케줄러 | Hermes | 레퍼런스 분석 자동화 등 |
| 🟢 P2 | NullHub 매니페스트 + 감독 | NullClaw | 서비스 관리 편의성 |
| 🟢 P2 | Hermes Atropos RL 환경 | Hermes | 장기 에이전트 미세조정 |
| 🔵 P3 | pi-pods vLLM GPU 관리 | pi-mono | 로컬 DSP 모델 배포 |
