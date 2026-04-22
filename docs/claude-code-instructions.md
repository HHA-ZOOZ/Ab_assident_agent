# Claude Code CLI — 단계별 구축 지시문

> 이 문서는 **Claude Code CLI**를 사용하여  
> Ab_assident_agent를 순서대로 구축할 때  
> 각 Phase에서 Claude에게 입력할 **정확한 지시 프롬프트**를 정리한다.
>
> - 실행 환경: `ANTHROPIC_API_KEY` 또는 `MOONSHOT_API_KEY` 설정 후 `claude` 실행
> - 실행 위치: 저장소 루트 (`/path/to/Ab_assident_agent/`)
> - 모든 지시는 한 Phase씩 순서대로 수행한다

---

## 사전 준비

```bash
# 저장소 루트에서 Claude Code 실행
cd /path/to/Ab_assident_agent
set -a && source .env && set +a
claude
```

---

## Phase 0 — 환경 준비

### Claude Code 지시문

```
저장소 루트에 다음 파일들을 생성해줘:

1. `.env.example` 파일 생성
   - docs/env-setup.md 의 "4. .env.example 파일" 섹션 내용을 그대로 사용

2. `.gitignore` 파일 생성
   - .env, __pycache__/, venv/, .venv/, *.pyc, *.egg-info/, dist/, build/, .ab_assident/ 를 포함

3. `pyproject.toml` 파일 생성
   - 프로젝트명: ab-assident-agent
   - Python 3.11+
   - 의존성:
     langgraph>=0.2, langchain-openai>=0.2, langchain-anthropic>=0.2,
     python-dotenv>=1.0, python-osc>=1.8, sqlalchemy>=2.0,
     qdrant-client>=1.9, rich>=13.0, typer>=0.12, pydantic-settings>=2.0,
     httpx>=0.27, apscheduler>=3.10, python-telegram-bot>=21.0
   - [project.optional-dependencies] dev = pytest, ruff, mypy

4. `packages/__init__.py` 빈 파일 생성
5. `packages/ai/__init__.py` 빈 파일 생성
6. `packages/agent-core/__init__.py` 빈 파일 생성
7. `packages/orchestrator/__init__.py` 빈 파일 생성
8. `packages/tracker/__init__.py` 빈 파일 생성
9. `packages/memory/__init__.py` 빈 파일 생성
10. `packages/skills/__init__.py` 빈 파일 생성
11. `packages/gateway/__init__.py` 빈 파일 생성
12. `packages/tui/__init__.py` 빈 파일 생성
13. `packages/hub/__init__.py` 빈 파일 생성
14. `packages/dsp/__init__.py` 빈 파일 생성

완료 후 uv로 설치 명령도 알려줘.
```

---

## Phase 1 — 에이전트 정체성 파일

### Claude Code 지시문

```
다음 파일들을 생성해줘. docs/architecture.md 의 OpenClaw 패턴을 따른다.

1. `SOUL.md` 생성:
   Ab_assident의 정체성 파일.
   - 이름: Ab_assident
   - 역할: Ableton Live 전문 음악 프로듀서 AI 에이전트
   - 핵심 능력: Max for Live 디바이스 생성, VST3 플러그인 생성,
     레퍼런스 트랙 분석, 믹싱/마스터링, OSC 기반 Ableton 제어
   - 작업 원칙: 사용자 음악 스타일 학습, 반복 패턴 스킬화,
     모든 생성물 메모리에 기록
   - 포맷: OpenClaw SOUL.md 스타일, 마크다운

2. `MEMORY.md` 생성:
   - 초기 빈 메모리 템플릿
   - 섹션: ## 음악 스타일, ## 자주 쓰는 패턴, ## 프로젝트 히스토리, ## 기술 메모

3. `USER.md` 생성:
   - 사용자 취향 모델 템플릿
   - 섹션: ## 선호 장르, ## 자주 쓰는 키/스케일, ## BPM 범위, ## 레퍼런스 아티스트

4. `AGENTS.md` 생성:
   - 에이전트가 따라야 할 작업 규칙 (pi-mono/NullClaw 패턴)
   - 포함: 도구 호출 전 항상 확인, Ableton 명령은 단계적으로 실행,
     생성 결과는 반드시 MEMORY.md에 기록, 사용자 승인 없이 파일 삭제 금지

5. `ab-manifest.json` 생성:
   - NullHub manifest 패턴
   - 컴포넌트: ableton-osc-bridge, orchestrator, gateway-telegram, gateway-cli
   - 각 컴포넌트에 launch 명령, health_check URL, description 포함
```

---

## Phase 2 — 멀티프로바이더 LLM 추상화

### Claude Code 지시문

```
docs/architecture.md 의 pi-ai 패턴과 docs/env-setup.md 를 참고하여
`packages/ai/` 레이어를 구현해줘.

1. `packages/ai/base.py`
   - 추상 기본 클래스 `BaseLLMProvider` 정의
   - 메서드: async chat(messages, **kwargs) -> str
   - 메서드: async stream_chat(messages, **kwargs) -> AsyncIterator[str]
   - 속성: model_name, provider_name

2. `packages/ai/providers/kimi.py` — 핵심 (Kimi-K2.6)
   - OpenAI SDK를 사용하여 Kimi API 연결
   - 환경변수: MOONSHOT_API_KEY, KIMI_BASE_URL, KIMI_MODEL
   - LangChain ChatOpenAI 래퍼도 함께 제공 (langchain_kimi() 함수)
   - 기본값: model="kimi-k2-0528", base_url="https://api.moonshot.cn/v1"

3. `packages/ai/providers/anthropic.py`
   - langchain_anthropic.ChatAnthropic 래퍼
   - 환경변수: ANTHROPIC_API_KEY
   - 기본값: model="claude-opus-4-5"

4. `packages/ai/providers/openai.py`
   - langchain_openai.ChatOpenAI 래퍼
   - 환경변수: OPENAI_API_KEY

5. `packages/ai/factory.py`
   - get_llm(provider: str = None) -> BaseChatModel 함수
   - provider=None 이면 환경변수 AB_DEFAULT_PROVIDER 사용 (기본값 "kimi")
   - provider 값: "kimi", "anthropic", "openai", "openrouter"

6. `packages/ai/__init__.py`
   - get_llm을 공개 API로 export

7. `tests/test_ai_providers.py`
   - 각 프로바이더가 올바른 환경변수를 읽는지 단위 테스트
   - API 호출은 mock으로 처리
```

---

## Phase 3 — 에이전트 코어 + 상태 머신

### Claude Code 지시문

```
docs/architecture.md 의 pi-agent-core + NullClaw 패턴을 참고하여
`packages/agent-core/` 를 구현해줘.

1. `packages/agent-core/state.py`
   - LangGraph TypedDict 기반 AgentState 정의
   - 포함 필드:
     messages: list (대화 이력)
     current_project: str (현재 Ableton 프로젝트명)
     current_track: str (현재 작업 중인 트랙)
     bpm: float
     key: str
     active_skills: list[str] (현재 로드된 스킬 목록)
     memory_context: str (현재 세션 메모리 요약)
     user_intent: str (현재 사용자 의도)
     tool_results: list (도구 실행 결과)
     error: str | None

2. `packages/agent-core/tool_loop.py`
   - LangGraph 기반 ReAct 스타일 도구 호출 루프
   - should_continue() 조건 함수 (도구 결과 있으면 계속, 없으면 종료)
   - call_model() 노드 함수 (Kimi-K2.6 호출)
   - call_tools() 노드 함수 (등록된 도구 실행)
   - build_graph(tools: list) -> CompiledGraph 함수

3. `packages/agent-core/interrupt.py`
   - NullBoiler interrupt 패턴
   - HumanApprovalRequired 예외 클래스
   - interrupt_if_dangerous(action: str) 함수
     위험 키워드 목록: ["delete", "rm", "format", "overwrite", "shutdown"]
   - Ableton 명령 승인 요청 함수 request_approval(action, context)

4. `packages/agent-core/checkpoint.py`
   - LangGraph MemorySaver 기반 체크포인트
   - save_checkpoint(state, checkpoint_id) 함수
   - load_checkpoint(checkpoint_id) -> AgentState 함수
   - fork_checkpoint(checkpoint_id, new_id) 함수 (NullBoiler 포킹 패턴)
   - list_checkpoints() -> list[CheckpointInfo] 함수

5. `tests/test_agent_core.py`
   - AgentState 생성 테스트
   - interrupt_if_dangerous 단위 테스트
   - checkpoint save/load 단위 테스트
```

---

## Phase 4 — 워크플로우 오케스트레이터

### Claude Code 지시문

```
docs/architecture.md 의 NullBoiler → LangGraph 매핑을 참고하여
`packages/orchestrator/` 를 구현해줘.

1. `packages/orchestrator/nodes/task_node.py`
   - 분석·생성 작업을 수행하는 기본 노드
   - execute(state: AgentState, task_description: str) -> AgentState
   - Kimi-K2.6을 호출하여 태스크 수행

2. `packages/orchestrator/nodes/route_node.py`
   - 조건 분기 노드
   - route(state: AgentState) -> str (다음 노드 이름 반환)
   - 조건: 오류 있으면 "error_handler", 인터럽트 필요하면 "human_review", 그 외 "continue"

3. `packages/orchestrator/nodes/interrupt_node.py`
   - human-in-the-loop 노드
   - 사용자에게 확인 요청 후 대기
   - LangGraph interrupt() 사용

4. `packages/orchestrator/nodes/send_node.py`
   - fan-out 병렬 실행 노드 (NullBoiler send 패턴)
   - LangGraph Send API 사용
   - 여러 서브태스크를 병렬로 분배

5. `packages/orchestrator/nodes/transform_node.py`
   - 출력 변환 노드
   - 도구 결과를 AgentState에 맞게 가공

6. `packages/orchestrator/workflow_graph.py`
   - 전체 워크플로우 StateGraph 조립
   - 노드 등록: task → route → (interrupt | send | transform) → task
   - build_music_workflow(tools: list) -> CompiledGraph 함수
   - 엔트리포인트: task_node

7. `packages/orchestrator/store_updates.py`
   - 워크플로우 완료 후 MEMORY.md 자동 업데이트 (NullBoiler store_updates 패턴)
   - extract_memory_updates(state: AgentState) -> list[str] 함수
   - append_to_memory(updates: list[str], memory_path="MEMORY.md") 함수

8. `tests/test_orchestrator.py`
   - route_node 조건 분기 단위 테스트
   - store_updates 메모리 추출 단위 테스트
```

---

## Phase 5 — 태스크 트래커 + KV 스토어

### Claude Code 지시문

```
docs/architecture.md 의 NullTickets 패턴을 참고하여
`packages/tracker/` 를 구현해줘. SQLite를 백엔드로 사용.

1. `packages/tracker/models.py`
   - SQLAlchemy ORM 모델
   - Task 모델: id, title, description, status(pending/running/done/failed),
     priority, created_at, completed_at, session_id, result
   - KVEntry 모델: namespace, key, value(JSON), updated_at
   - SessionLog 모델: session_id, role, content, timestamp (FTS 검색용)

2. `packages/tracker/task_queue.py`
   - create_task(title, description, priority=5) -> Task
   - get_next_task() -> Task | None (우선순위 순)
   - complete_task(task_id, result) -> None
   - fail_task(task_id, error) -> None
   - list_tasks(status=None) -> list[Task]

3. `packages/tracker/kv_store.py`
   - set(namespace, key, value) -> None
   - get(namespace, key) -> Any | None
   - delete(namespace, key) -> None
   - list_keys(namespace) -> list[str]
   - namespace 예시: "skills", "user_prefs", "project_state"

4. `packages/tracker/fts.py`
   - Hermes FTS5 패턴: 세션 로그 전문 검색
   - log_message(session_id, role, content) -> None
   - search(query: str, limit=10) -> list[SearchResult]
   - summarize_session(session_id) -> str (Kimi-K2.6으로 요약)

5. `packages/tracker/db.py`
   - get_engine() 함수 (AB_HOME/.ab_assident/tracker.db)
   - init_db() 함수 (테이블 생성)

6. `tests/test_tracker.py`
   - 인메모리 SQLite로 task_queue, kv_store, fts 단위 테스트
```

---

## Phase 6 — 메모리 시스템

### Claude Code 지시문

```
docs/architecture.md 의 Hermes + OpenClaw 메모리 패턴을 참고하여
`packages/memory/` 를 구현해줘.

1. `packages/memory/episodic.py`
   - Hermes FTS5 패턴: 과거 세션 검색
   - packages/tracker/fts.py 를 활용
   - recall(query: str) -> list[str] 함수 (관련 과거 대화 반환)
   - inject_context(state: AgentState) -> AgentState (메모리를 state에 주입)

2. `packages/memory/semantic.py`
   - Qdrant 벡터 DB 기반 의미 검색
   - store(text: str, metadata: dict) -> None
   - search(query: str, limit=5) -> list[Document]
   - QDRANT_URL 환경변수 없으면 인메모리 Qdrant 사용

3. `packages/memory/user_model.py`
   - Honcho dialectic 패턴 (단순화 버전)
   - update_preference(key: str, value: Any) -> None
   - get_preferences() -> dict
   - infer_style_from_history() -> str (Kimi-K2.6으로 사용자 스타일 요약)
   - packages/tracker/kv_store.py 의 "user_prefs" namespace 사용

4. `packages/memory/skill_memory.py`
   - 스킬 사용 패턴 추적
   - record_skill_use(skill_name: str, context: str, success: bool) -> None
   - get_relevant_skills(task: str) -> list[str] (태스크에 맞는 스킬 추천)
   - packages/tracker/kv_store.py 의 "skill_usage" namespace 사용

5. `packages/memory/manager.py`
   - MemoryManager 클래스 — 위 4개를 통합
   - get_context_for_task(task: str) -> str
   - save_session_result(state: AgentState) -> None

6. `tests/test_memory.py`
   - user_model preference 단위 테스트
   - skill_memory record/retrieve 단위 테스트
```

---

## Phase 7 — 스킬 시스템

### Claude Code 지시문

```
docs/architecture.md 의 OpenClaw/Hermes agentskills.io 패턴을 참고하여
`packages/skills/` 를 구현해줘.

1. `packages/skills/loader.py`
   - Markdown 스킬 파일 파싱 및 로딩
   - load_skill(path: str) -> Skill 함수
   - load_all_skills(directory: str) -> list[Skill] 함수
   - Skill 데이터클래스: name, description, instructions, examples, tags

2. 내장 스킬 파일 5개 생성 (agentskills.io 포맷 — Markdown):
   
   `packages/skills/builtin/ableton_osc.md`
   - 설명: Ableton Live OSC 제어
   - 지시: 트랙 선택, 클립 실행, 파라미터 조절 방법
   - 도구: ableton_send_osc, ableton_get_state
   
   `packages/skills/builtin/m4l_generate.md`
   - 설명: Max for Live 디바이스 (.amxd) 자동 생성
   - 지시: Max Patch 구조, 오브젝트 연결 패턴
   - 도구: write_m4l_patch, install_m4l_device
   
   `packages/skills/builtin/audio_analyze.md`
   - 설명: 레퍼런스 트랙 분석 (BPM, 키, 스펙트럼, 구조)
   - 지시: librosa를 이용한 분석 절차
   - 도구: analyze_audio_file, detect_bpm, detect_key
   
   `packages/skills/builtin/vst3_create.md`
   - 설명: JUCE 기반 VST3 플러그인 생성
   - 지시: CMakeLists.txt 생성, 기본 플러그인 코드 템플릿
   - 도구: write_cpp_file, cmake_build
   
   `packages/skills/builtin/mix_master.md`
   - 설명: 믹싱·마스터링 체인 구성
   - 지시: EQ, 컴프레서, 리미터 설정 패턴
   - 도구: ableton_add_device, set_device_parameter

3. `packages/skills/self_improve/skill_creator.py`
   - Hermes 자기개선 패턴
   - should_create_skill(state: AgentState) -> bool
     (태스크가 복잡하고 반복 가능한 패턴이면 True)
   - create_skill_from_session(state: AgentState) -> str (새 스킬 파일 경로)
   - Kimi-K2.6을 호출하여 스킬 내용 자동 생성

4. `packages/skills/self_improve/skill_improver.py`
   - improve_skill(skill_name: str, feedback: str) -> None
   - 기존 스킬 파일을 피드백 기반으로 업데이트

5. `tests/test_skills.py`
   - 스킬 로딩 파싱 단위 테스트
   - should_create_skill 조건 단위 테스트
```

---

## Phase 8 — DSP 엔진 연동

### Claude Code 지시문

```
`packages/dsp/` 를 구현해줘. Ableton Live와 OSC로 통신한다.

1. `packages/dsp/ableton_osc.py`
   - python-osc 라이브러리 사용
   - AbletonOSCClient 클래스:
     send(address: str, *args) 메서드
     async get_state() -> dict 메서드 (현재 Ableton 상태)
     track_select(track_index: int) 메서드
     clip_launch(track_index: int, clip_index: int) 메서드
     set_parameter(track, device, param, value) 메서드
     set_bpm(bpm: float) 메서드
   - 환경변수: ABLETON_OSC_HOST, ABLETON_OSC_SEND_PORT, ABLETON_OSC_RECEIVE_PORT

2. `packages/dsp/m4l_generator.py`
   - M4LGenerator 클래스
   - generate_patch(description: str) -> str (Max Patch JSON 생성)
     → Kimi-K2.6을 호출하여 Max Patch 구조 생성
   - write_amxd(patch_json: str, output_path: str) -> str
   - AMXD_TEMPLATE: 기본 Max Patch 템플릿 문자열

3. `packages/dsp/audio_analyzer.py`
   - AudioAnalyzer 클래스 (librosa 사용)
   - analyze(file_path: str) -> AudioInfo
   - AudioInfo 데이터클래스: bpm, key, scale, duration, loudness, spectrum_summary

4. `packages/dsp/tool_registry.py`
   - 위 DSP 함수들을 LangChain Tool로 래핑하여 반환
   - get_dsp_tools() -> list[Tool]
   - 포함 도구: ableton_send_osc, ableton_get_state, generate_m4l,
     analyze_audio, set_bpm, track_select

5. `tests/test_dsp.py`
   - AbletonOSCClient mock 테스트 (실제 Ableton 없이)
   - AudioAnalyzer 단위 테스트 (테스트용 오디오 파일 사용)
```

---

## Phase 9 — 멀티플랫폼 게이트웨이

### Claude Code 지시문

```
docs/architecture.md 의 OpenClaw/Hermes 게이트웨이 패턴을 참고하여
`packages/gateway/` 를 구현해줘.

1. `packages/gateway/base.py`
   - GatewayBase 추상 클래스
   - async send_message(text: str, chat_id=None) -> None
   - async on_message(handler: Callable) -> None
   - Message 데이터클래스: text, sender_id, platform, timestamp

2. `packages/gateway/cli.py`
   - Rich + Typer 기반 터미널 인터페이스
   - CLIGateway(GatewayBase) 구현
   - Rich Live 패널: 좌측 대화창 + 우측 Ableton 상태
   - 슬래시 명령: /new, /memory, /skills, /status, /help
   - 멀티라인 입력 지원

3. `packages/gateway/telegram.py`
   - python-telegram-bot 기반
   - TelegramGateway(GatewayBase) 구현
   - 환경변수: TELEGRAM_BOT_TOKEN
   - 명령: /start, /new, /skills, /status
   - 이미지·음성 파일 수신 지원

4. `packages/gateway/dispatcher.py`
   - 들어온 메시지를 워크플로우 그래프로 라우팅
   - dispatch(message: Message) -> str (응답)
   - 워크플로우 그래프와 메모리 관리자 연결

5. `packages/gateway/__main__.py`
   - `python -m packages.gateway [cli|telegram|discord]` 진입점
   - argparse로 플랫폼 선택

6. `tests/test_gateway.py`
   - CLI 게이트웨이 메시지 파싱 단위 테스트
   - dispatcher 라우팅 단위 테스트
```

---

## Phase 10 — TUI 터미널 UI

### Claude Code 지시문

```
`packages/tui/` 를 구현해줘. Rich 라이브러리를 사용한다.
pi-tui 의 차분 렌더링 개념을 Rich Live 로 구현한다.

1. `packages/tui/panels.py`
   - AbletonStatusPanel: 현재 BPM, 키, 트랙, 클립 상태 표시
   - SkillPanel: 현재 로드된 스킬 목록
   - MemoryPanel: 최근 기억 항목 5개
   - ConversationPanel: 대화 이력 스크롤

2. `packages/tui/renderer.py`
   - AbRenderer 클래스
   - Rich Layout 기반 3-패널 레이아웃:
     상단: AbletonStatusPanel
     좌측: ConversationPanel
     우측: SkillPanel + MemoryPanel
   - start() / stop() / update(state: AgentState) 메서드
   - Rich Live 사용

3. `packages/tui/__main__.py`
   - `python -m packages.tui` 로 TUI 미리보기 실행 (mock 데이터)
```

---

## Phase 11 — 컴포넌트 허브

### Claude Code 지시문

```
docs/architecture.md 의 NullHub 패턴을 참고하여
`packages/hub/` 를 구현해줘.

1. `packages/hub/manifest.py`
   - ab-manifest.json 파싱
   - ComponentManifest 데이터클래스: name, launch, health_check_url, description
   - load_manifest(path="ab-manifest.json") -> list[ComponentManifest]

2. `packages/hub/process_supervisor.py`
   - NullHub 프로세스 감독 패턴
   - ProcessSupervisor 클래스:
     start(component: ComponentManifest) -> None
     stop(name: str) -> None
     restart(name: str) -> None
     get_status(name: str) -> ProcessStatus
     start_all() / stop_all() 메서드
   - 크래시 감지 시 자동 재시작 (최대 3회)

3. `packages/hub/health_check.py`
   - check(url: str) -> bool (HTTP GET 200 확인)
   - check_all(components: list[ComponentManifest]) -> dict[str, bool]

4. `packages/hub/__main__.py`
   - `python -m packages.hub [start-all|stop-all|status]` 진입점
   - Rich Table로 컴포넌트 상태 표시

5. `tests/test_hub.py`
   - manifest 파싱 단위 테스트
   - health_check mock 테스트
```

---

## Phase 12 — cron 스케줄러

### Claude Code 지시문

```
`cron/` 을 구현해줘. Hermes cron 패턴을 따른다.

1. `cron/schedules.json` 업데이트:
   다음 기본 스케줄 포함:
   - id: "daily-reference-scan", cron: "0 9 * * *"
     task: "packages/skills/builtin/audio_analyze.md 스킬을 사용하여 ~/Music/References/ 폴더 스캔 후 MEMORY.md 업데이트"
   - id: "weekly-skill-review", cron: "0 10 * * 1"  
     task: "지난 주 생성된 스킬들을 검토하고 개선점을 skills/self_improve/ 에 적용"

2. `cron/scheduler.py`
   - APScheduler 기반
   - load_schedules(path="cron/schedules.json") 함수
   - CronScheduler 클래스:
     start() / stop() 메서드
     add_schedule(schedule: dict) 메서드
   - 스케줄 실행 시 dispatcher.dispatch()를 통해 에이전트 호출

3. `cron/__main__.py`
   - `python -m cron` 진입점
```

---

## Phase 13 — 통합 진입점

### Claude Code 지시문

```
전체를 하나로 연결하는 메인 진입점을 만들어줘.

1. `main.py`
   - 환경변수 로딩 (.env)
   - packages/tracker/db.py init_db() 호출
   - packages/hub/__main__.py 통해 컴포넌트 시작
   - 플랫폼 선택:
     `python main.py cli` → CLI 게이트웨이
     `python main.py telegram` → 텔레그램 게이트웨이
     `python main.py hub` → 허브 관리 콘솔만 실행

2. README.md 업데이트:
   - 빠른 시작 섹션 추가:
     1) .env 설정 (docs/env-setup.md 참조)
     2) uv pip install -e .
     3) python main.py cli
   - 문서 링크:
     - docs/architecture.md
     - docs/build-order.md
     - docs/claude-code-instructions.md
     - docs/env-setup.md
     - docs/repos.md
```

---

## Phase 14 — 테스트 + 검증

### Claude Code 지시문

```
전체 테스트를 실행하고 문제를 수정해줘.

1. 다음 명령 순서로 실행:
   uv pip install -e ".[dev]"
   python -m pytest tests/ -v --tb=short

2. 실패한 테스트가 있으면 원인을 분석하고 수정해줘.

3. Kimi-K2.6 연결 테스트 (MOONSHOT_API_KEY가 설정된 경우):
   python -c "
   from packages.ai.factory import get_llm
   llm = get_llm('kimi')
   result = llm.invoke('Ableton Live에서 킥 드럼을 만드는 방법을 한 줄로 설명해.')
   print(result.content)
   "

4. CLI 실행 테스트:
   python main.py cli

5. 각 컴포넌트 임포트 확인:
   python -c "from packages.agent_core.state import AgentState; print('OK')"
   python -c "from packages.orchestrator.workflow_graph import build_music_workflow; print('OK')"
   python -c "from packages.memory.manager import MemoryManager; print('OK')"
```

---

## 빠른 참조 — Phase별 핵심 파일

| Phase | 핵심 생성 파일 | 참조 프레임워크 |
|---|---|---|
| 0 | pyproject.toml, .gitignore, .env.example | — |
| 1 | SOUL.md, MEMORY.md, USER.md, AGENTS.md | OpenClaw |
| 2 | packages/ai/factory.py, providers/kimi.py | pi-mono (pi-ai) |
| 3 | packages/agent-core/state.py, tool_loop.py | pi-agent-core + NullClaw |
| 4 | packages/orchestrator/workflow_graph.py | NullBoiler → LangGraph |
| 5 | packages/tracker/task_queue.py, kv_store.py, fts.py | NullTickets |
| 6 | packages/memory/manager.py | Hermes + Honcho |
| 7 | packages/skills/builtin/*.md, skill_creator.py | OpenClaw/Hermes |
| 8 | packages/dsp/ableton_osc.py, tool_registry.py | 신규 |
| 9 | packages/gateway/cli.py, telegram.py | OpenClaw/Hermes |
| 10 | packages/tui/renderer.py | pi-tui |
| 11 | packages/hub/process_supervisor.py | NullHub |
| 12 | cron/scheduler.py | Hermes |
| 13 | main.py, README.md | — |
| 14 | 테스트 통과 확인 | — |
