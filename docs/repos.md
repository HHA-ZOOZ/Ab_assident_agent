# 참조 저장소 목록 (Reference Repositories)

> Ab_assident_agent 구축에 참조·이식하는 오픈소스 저장소 전체 목록.  
> 각 항목에 역할, 이식 대상, 라이선스를 기재.

---

## 1. NullClaw 에코시스템

| 저장소 | URL | 언어 | 역할 | 이식 대상 |
|---|---|---|---|---|
| **nullclaw/nullclaw** | https://github.com/nullclaw/nullclaw | Zig | 에이전트 실행 런타임 | 에이전트 루프 설계, 도구 호출 패턴 |
| **nullclaw/nullboiler** | https://github.com/nullclaw/nullboiler | Zig | 오케스트레이션 정책 엔진 | 워크플로우 그래프 노드 타입, 체크포인트 포킹, store_updates |
| **nullclaw/nulltickets** | https://github.com/nullclaw/nulltickets | Zig | 태스크 트래커 + KV 스토어 | 태스크 큐 설계, 영속 KV 스토어, FTS |
| **nullclaw/nullhub** | https://github.com/nullclaw/nullhub | Zig + Svelte | 관리 콘솔 | 매니페스트 기반 컴포넌트 등록, 프로세스 감독, SSE 로그 |

### NullBoiler 워크플로우 노드 타입 (LangGraph 매핑 참조)

```
task       → 분석/생성 노드
agent      → 서브에이전트 노드
route      → 조건 분기 엣지
interrupt  → human-in-the-loop 브레이크포인트
send       → fan-out 병렬 실행
transform  → 출력 변환 노드
subgraph   → 재사용 서브그래프
```

---

## 2. OpenClaw (Claw 패밀리 원조)

| 저장소 | URL | 언어 | 역할 | 이식 대상 |
|---|---|---|---|---|
| **OpenClaw** (private/commercial) | https://openclaw.ai | Python | 원조 Claw 에이전트 | SOUL.md 페르소나, agentskills.io 스킬 표준, 멀티플랫폼 게이트웨이 |

### agentskills.io 관련

| 저장소 | URL | 설명 |
|---|---|---|
| **agentskills.io** (표준) | https://agentskills.io | OpenClaw/Hermes 공유 스킬 포맷 표준 |
| **kcosr/pi-extensions** | https://github.com/kcosr/pi-extensions | pi-mono 확장 스킬 예제 |
| **open-gitagent/gitagent** | https://github.com/open-gitagent/gitagent | git-native 에이전트 스킬 표준 |
| **mkhsu2002/openclaw-agent-feeds** | https://github.com/mkhsu2002/openclaw-agent-feeds | OpenClaw 표준화 데이터 피드 |

---

## 3. pi-mono (badlogic)

| 저장소 | URL | 언어 | 역할 | 이식 대상 |
|---|---|---|---|---|
| **badlogic/pi-mono** | https://github.com/badlogic/pi-mono | TypeScript | 모노레포 에이전트 프레임워크 | 전체 패키지 구조, 멀티프로바이더 AI, 에이전트 코어, TUI |

### pi-mono 서브패키지

| 패키지 | 경로 | 이식 대상 |
|---|---|---|
| `@mariozechner/pi-ai` | `packages/ai` | LLM 멀티프로바이더 추상화 레이어 |
| `@mariozechner/pi-agent-core` | `packages/agent` | 에이전트 상태 머신 + tool calling 루프 |
| `@mariozechner/pi-coding-agent` | `packages/coding-agent` | CLI 인터랙션 패턴 |
| `@mariozechner/pi-tui` | `packages/tui` | 차분 렌더링 터미널 UI |
| `@mariozechner/pi-web-ui` | `packages/web-ui` | 웹 컴포넌트 UI |
| `@mariozechner/pi-pods` | `packages/pods` | vLLM GPU 배포 관리 |

---

## 4. Hermes (NousResearch)

| 저장소 | URL | 언어 | 역할 | 이식 대상 |
|---|---|---|---|---|
| **NousResearch/hermes-agent** | https://github.com/NousResearch/hermes-agent | Python | 자기개선 AI 에이전트 | 자기개선 스킬 루프, Honcho 사용자 모델, FTS5 세션 검색, cron, Atropos RL, 멀티플랫폼 게이트웨이 |

### Hermes 관련 보조 저장소

| 저장소 | URL | 설명 |
|---|---|---|
| **plastic-labs/honcho** | https://github.com/plastic-labs/honcho | dialectic 사용자 모델링 (Hermes에서 사용) |
| **NousResearch/tinker-atropos** | (서브모듈) | RL trajectory 생성 + Atropos 환경 |
| **mudrii/hermes-agent-docs** | https://github.com/mudrii/hermes-agent-docs | Hermes 아키텍처 상세 문서 |
| **briancaffey/hermes-agent-curriculum** | https://github.com/briancaffey/hermes-agent-curriculum | Hermes 코드베이스 학습 커리큘럼 |

---

## 5. 아키텍처 참조 · 분석 저장소

| 저장소 | URL | 설명 |
|---|---|---|
| **breath57/how-ai-agents-remember** | https://github.com/breath57/how-ai-agents-remember | OpenClaw/NullClaw/Hermes 메모리 시스템 역공학 분석 |
| **subinium/awesome-agent-frameworks** | https://github.com/subinium/awesome-agent-frameworks | Claw 에코시스템 프레임워크 가이드 |
| **pigeonflow/brain-arch-v2** | https://github.com/pigeonflow/brain-arch-v2 | NullClaw 기반 뇌 구조 영감 아키텍처 |

---

## 6. 멀티에이전트 · 오케스트레이션 참조

| 저장소 | URL | 설명 |
|---|---|---|
| **zeynepyorulmaz/openclaw-orchestrator** | https://github.com/zeynepyorulmaz/openclaw-orchestrator | TypeScript 기반 멀티에이전트 오케스트레이션 |
| **open-gitagent/gitclaw** | https://github.com/open-gitagent/gitclaw | git-native 범용 에이전트 프레임워크 |
| **lanyasheng/openclaw-multiagent-framework** | https://github.com/lanyasheng/openclaw-multiagent-framework | OpenClaw 멀티에이전트 협업 프로토콜 |

---

## 7. 메모리 · 지식 그래프 참조

| 저장소 | URL | 설명 |
|---|---|---|
| **ahonore42/logios-brain** | https://github.com/ahonore42/logios-brain | 에피소드 메모리(Postgres+Qdrant) + 의미 지식(Neo4j) + MCP 통합 |
| **yantrikos/yantrikdb-hermes-plugin** | https://github.com/yantrikos/yantrikdb-hermes-plugin | 자기유지 메모리 — 정규화, 모순 추적, 최근성 랭킹 |
| **perlowja/mnemos** | https://github.com/perlowja/mnemos | 에이전트 메모리 OS — MCP + REST API |

---

## 8. 스킬 생태계 참조

| 저장소 | URL | 설명 |
|---|---|---|
| **wanshuiyin/Auto-claude-code-research-in-sleep** | https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep | Markdown-only 자율 연구 스킬 패턴 |
| **slhleosun/EvoClaw** | https://github.com/slhleosun/EvoClaw | SOUL 진화 프레임워크 (경험→반성→정체성 업데이트) |
| **farosud/traction-skills** | https://github.com/farosud/traction-skills | Hermes 호환 스킬 예제 19개 |
| **Danielhogben/hermes-skills** | https://github.com/Danielhogben/hermes-skills | Hermes 원본 스킬 6개 |
