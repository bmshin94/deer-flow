# 🦌 DeerFlow 2.0 전수조사 분석 & 수익화 전략 (한국어 정리)

> 작성일: 2026-10-03
> 작성: Claude Code (카리나 페르소나) × @bmshin94
> 이 문서는 DeerFlow 2.0 저장소를 전수조사한 결과와, 설치/활용/수익화 전략을 한국어로 정리한 것입니다.

## 🔗 관련 깃허브 주소

| 구분 | 주소 |
| --- | --- |
| **이 저장소 (포크)** | https://github.com/bmshin94/deer-flow |
| **원본 (upstream)** | https://github.com/bytedance/deer-flow |
| v1 (구 Deep Research) | https://github.com/bytedance/deer-flow/tree/main-1.x |
| 자매 프로젝트 LLM Space | https://github.com/deer-flow/llm-space |
| 공식 웹사이트 | https://deerflow.tech |
| Claude Code 연동 스킬 설치 | `npx skills add https://github.com/bytedance/deer-flow --skill claude-to-deerflow` |

---

## 목차

1. [프로젝트 정체 파악](#1-프로젝트-정체-파악)
2. [쉽게 이해하기 (비유 설명)](#2-쉽게-이해하기-비유-설명)
3. [Q&A 7문 7답](#3-qa-7문-7답)
4. [수익화 전략 9가지](#4-수익화-전략-9가지)
5. [실행 체크리스트](#5-실행-체크리스트)

---

## 1. 프로젝트 정체 파악

### 1.1 한 줄 정의

**DeerFlow 2.0** = ByteDance(TikTok 모회사)가 공개한 오픈소스 **슈퍼 에이전트 하네스(Super Agent Harness)**.

- 이름 뜻: **D**eep **E**xploration and **E**fficient **R**esearch **Flow**
- 라이선스: **MIT** (상업적 이용·수정·재배포 자유, 로열티 0원)
- 저작권: Copyright (c) 2025 Bytedance Ltd. / (c) 2025-2026 DeerFlow Authors
- 2026년 2월 28일 **GitHub Trending 1위** 달성
- 기반 기술: **LangGraph + LangChain** (Python 3.12+), **Next.js** (Node 22+)
- 2.0은 1.x와 코드를 공유하지 않는 **완전 재작성(ground-up rewrite)** 버전

### 1.2 저장소 규모

| 항목 | 수치 |
| --- | --- |
| 백엔드 Python 파일 | 약 1,512개 |
| 프론트엔드 TS/TSX 파일 | 약 489개 |
| 내장 공개 스킬 | 23개 |
| 에이전트 미들웨어 | 40개 이상 |
| 지원 메신저 채널 | 10종 |
| README | 약 167KB (영/중/일/불/러 — **한국어 없음**) |
| CHANGELOG | 약 275KB |

### 1.3 서비스 토폴로지 (`make dev` / Docker 1세트)

| 서비스 | 포트 | 역할 |
| --- | --- | --- |
| **Nginx** | `2026` | 유일한 외부 진입점 (브라우저 접속 지점) |
| **Gateway API** | `8001` | FastAPI REST + LangGraph 호환 에이전트 런타임 |
| **Frontend** | `3000` | Next.js 웹 채팅 UI |
| **Provisioner** | `8002` | 선택 — 샌드박스가 provisioner/K8s 모드일 때만 |

보안 기본값: 두 compose 파일 모두 진입 포트를 `"${BIND_HOST:-127.0.0.1}:${PORT:-2026}:2026"`,
즉 **루프백 전용**으로 퍼블리시한다. Nginx가 `default_server`로 IPv4/IPv6를 듣고 Gateway가
`0.0.0.0:8001`에 바인딩하는 것은 모두 **컨테이너 내부**로 의도된 것이며, 외부 노출 면은
퍼블리시된 nginx 포트 하나뿐이다. `backend/tests/test_compose_default_bind_host.py`가 이를 고정한다.

### 1.4 저장소 구조

```
deer-flow/
├── Makefile                        # 루트 오케스트레이션 (dev/start/stop, docker, setup)
├── config.example.yaml             # → config.yaml 로 복사 (gitignored)
├── extensions_config.example.json  # → extensions_config.json (MCP 서버 + 스킬)
├── backend/
│   ├── packages/harness/deerflow/  # 에이전트 프레임워크 본체 (import: deerflow.*)
│   │   ├── agents/                 # lead_agent, memory, middlewares(40+), task_continuity
│   │   ├── tools/builtins/         # task, tool_search, clarification, present_file 등
│   │   ├── sandbox/                # local / docker / E2B / provisioner(K8s)
│   │   ├── skills/                 # 스킬 로더 + SkillScan 보안 스캐너 + review
│   │   ├── mcp/                    # MCP 서버 연동
│   │   ├── subagents/              # 서브에이전트 위임
│   │   ├── scheduler/              # 예약 작업
│   │   ├── persistence/            # DB + 마이그레이션
│   │   ├── authz/                  # RBAC 권한
│   │   ├── projects/ · reflection/ · uploads/ · tracing/
│   │   └── tui/                    # 터미널 워크벤치
│   ├── packages/extension-api/     # deerflow-extension-api (공개 확장 계약)
│   └── app/
│       ├── gateway/                # FastAPI 라우터, 인증, CSRF, health
│       └── channels/               # slack, telegram, discord, feishu, wechat,
│                                   # wecom, dingtalk, github, buzz(nostr)
├── frontend/                       # Next.js App Router (pnpm)
├── skills/public/                  # 23개 내장 스킬 (custom/ 은 gitignored)
├── docker/                         # docker-compose + nginx + provisioner + lark-cli-init
├── deploy/helm/deer-flow/          # 쿠버네티스 Helm 차트
├── contracts/                      # 크로스컴포넌트 JSON 계약 (subagent status, skill review)
├── examples/deerflow-extension-example/  # 5가지 확장 기여 전부 시연하는 예제 패키지
├── scripts/                        # check, configure, doctor, support_bundle, deploy 등
└── docs/                           # 설계 문서 / 플랜 / 아키텍처
```

### 1.5 핵심 기능 6가지

#### ① Skills (스킬) — 가장 중요한 차별점

`SKILL.md` 마크다운 1개 = 능력 모듈 1개. 워크플로우, 베스트 프랙티스, 보조 리소스 참조를 정의한다.

**내장 공개 스킬 23개**

| 스킬 | 용도 |
| --- | --- |
| `deep-research` | 다각도·다단계 심층 웹 리서치 (WebSearch 대체) |
| `github-deep-research` | 깃허브 저장소 심층 분석 |
| `systematic-literature-review` | 체계적 문헌 고찰 |
| `academic-paper-review` | 논문 리뷰 |
| `consulting-analysis` | 컨설팅식 구조화 분석 |
| `data-analysis` | 데이터/CSV 분석 |
| `chart-visualization` | 차트 시각화 |
| `ppt-generation` | 슬라이드별 AI 이미지 생성 → PPTX 조립 |
| `image-generation` | 이미지 생성 |
| `video-generation` | 영상 생성 |
| `music-generation` | 음악 생성 |
| `podcast-generation` | 팟캐스트 제작 |
| `newsletter-generation` | 뉴스레터 작성 |
| `frontend-design` | 프론트엔드/UI 디자인 |
| `web-design-guidelines` | 웹 디자인 가이드라인 |
| `code-documentation` | 코드 문서화 |
| `skill-creator` | 스킬을 만드는 스킬 |
| `skill-reviewer` | 읽기 전용 스킬 품질 검수 (`review_skill_package` 툴) |
| `find-skills` | 스킬 탐색 |
| `claude-to-deerflow` | Claude Code ↔ DeerFlow 연동 |
| `vercel-deploy-claimable` | Vercel 배포 |
| `bootstrap` | 초기 부트스트랩 |
| `surprise-me` | 랜덤 제안 |

**핵심 설계 포인트**
- **점진적 로딩(progressive loading)**: 필요할 때만 스킬을 읽어 컨텍스트 창을 절약 → 토큰 민감 모델에서도 잘 동작하고, **운영 원가가 낮아진다**(수익화 핵심).
- **슬래시 활성화**: `/data-analysis analyze uploads/foo.csv` 처럼 턴 단위 명시 활성화 가능.
- **allowed-tools 정책**: 슬래시 활성화되거나 `read_file`로 활성 컨텍스트에 포착된 뒤에만 적용. 모델 가시 스키마와 실제 실행 양쪽을 필터링. 단, "최선 노력 행위 스코핑"이며 하드 보안 경계는 아님.
- **스킬 디렉터리 = 패키지 경계**: `SKILL.md`를 찾으면 그 하위의 중첩 `SKILL.md`는 보조 데이터로 취급.
- **UTF-8 강제**: 스킬 텍스트는 플랫폼 로케일에 의존하지 않고 명시적 UTF-8로 읽고 쓴다.
- **SkillScan**: 설치/편집 시 결정적(deterministic) 안전 스캐너가 LLM 스캐너보다 먼저 동작. Phase 1은 오프라인(Semgrep 불필요)으로 개인키·셸 실행 등 고신뢰 `CRITICAL`을 차단. `skill_scan.enabled: false`로 결정적 분석만 비활성화 가능(아카이브 안전 추출과 LLM 스캐너는 유지).
- **활성 상태 ↔ 샌드박스 동기화**: 스킬을 비활성화하면 샌드박스 `/mnt/skills` 투영에서도 사라진다.
- **CI 웨이버**: `.github/skill-review-waivers.v1.json`. 신뢰된 base 매니페스트만 findings를 억제할 수 있어, 웨이버 활용은 "매니페스트 머지 → 스킬 변경 머지" 2단계가 필요하다. blocker는 절대 웨이버 불가.

#### ② Sandbox (샌드박스)

에이전트가 실제 코드를 실행하는 격리 환경. 4가지 공급자:

| 모드 | 특징 |
| --- | --- |
| `Local` | 호스트 직접 실행. **관리형 툴 경로 경계이며 호스트 파일시스템 격리는 아님.** 명시적 per-Agent 스킬 정책은 호스트 bash가 비활성(기본)일 때만 허용 |
| `Docker / AIO` | 권장. 실제 파일시스템 경계 확보 |
| `E2B` | 클라우드 샌드박스 (`E2B_API_KEY`). 기존 샌드박스는 생성 시점 스냅샷 유지 |
| `Provisioner (K8s)` | 엔터프라이즈. hostPath/PVC/initContainer 지원 |

#### ③ Sub-Agents (서브에이전트)

`task` 툴로 작업을 위임. 리드 에이전트와 동일한 점진적 발견/활성화 정책을 사용하며,
`delegation_ledger`, `subagent_limit_middleware`가 위임 폭주를 제어한다.
서브에이전트 상태 계약은 `contracts/`에 JSON으로 고정돼 있다.

#### ④ Long-Term Memory (장기 기억)

스레드 경계를 넘는 기억. `agents/memory/`에 manager, backends, summarization_hook, tools.
최근 커밋에 **opt-in relevance-aware retrieval ranking**(관련도 기반 검색 순위) 추가(#5251).

#### ⑤ Context Engineering (미들웨어 체인) — 기술적 백본

`agents/middlewares/`의 40개 이상 미들웨어가 하네스의 본질이다.

| 미들웨어 | 역할 |
| --- | --- |
| `token_budget_middleware` | 토큰 예산 관리 |
| `summarization_middleware` | 컨텍스트 압축 요약 |
| `dynamic_context_middleware` / `durable_context_middleware` | 동적·영속 컨텍스트 |
| `loop_detection_middleware` | 무한 루프 감지 |
| `skill_activation_middleware` / `skill_tool_policy_middleware` | 스킬 활성화·툴 권한 |
| `deferred_tool_filter_middleware` | 지연 툴 카탈로그 필터링 |
| `mcp_routing_middleware` | MCP 라우팅 |
| `memory_middleware` | 기억 주입 |
| `read_before_write_middleware` | 쓰기 전 읽기 강제 |
| `sandbox_audit_middleware` / `audit_context` | 샌드박스 감사 |
| `safety_finish_reason_middleware` / `terminal_response_middleware` | 안전·종료 처리 |
| `clarification_middleware` | 사용자 재질의 |
| `todo_middleware` | 할 일 추적 |
| `subagent_limit_middleware` / `delegation_ledger` | 위임 제어 |
| `llm_error_handling_middleware` / `tool_error_handling_middleware` | 오류 복원 |
| `title_middleware` | 스레드 제목 자동 생성 |
| `input_sanitization_middleware` | 입력 정화 |
| `receipt_verification` | 영수증 검증 |
| `configured_extensions` | 설정된 외부 미들웨어 로딩 |

#### ⑥ IM Channels (메신저 연동)

`app/channels/` — **Slack, Telegram, Discord, Feishu(Lark), WeChat, WeCom, DingTalk, GitHub, Buzz(Nostr)**.
`run_policy`, `dedupe_store`, `message_bus`, `commands`로 중복 처리·실행 정책·명령을 공통화.

### 1.6 그 외 주요 기능

- **Projects** (`/workspace/projects`) — 작업 묶음 관리
- **Scheduled Tasks** (`/workspace/scheduled-tasks`) — `config.yaml -> scheduler.enabled` 게이트.
  예약 실행은 **의도적으로 비대화형**: `context.non_interactive=true`일 때 리드 에이전트 툴셋에서
  `ask_clarification`이 제외된다. `non_interactive`·`disable_clarification`·`github_token`은
  **내부 인증 호출자만** 반영되고, 클라이언트가 보낸 복사본은 `body.context`/`body.config` 양쪽에서 폐기된다.
  바쁜 시점의 occurrence는 `queued`로 **영속 저장**되고, `launching`은 리스 펜싱된 짧은 클레임,
  `running`이 정상 실행 수명주기. `scheduler.queue_timeout_seconds`가 대기 상한.
  (skip-on-overlap을 되살리거나 대기 행을 `max_concurrent_runs`에 포함시키지 말 것)
- **Session Goals** — `client.set_goal()` / TUI `/goal`
- **Manual Context Compaction** — 수동 컨텍스트 압축
- **Capability Center** (`/workspace/capabilities`) — Plugins(MCP + Lark) & Skills 관리 UI.
  Community 탭에서 `.skill` 아카이브 임포트.
- **Custom Agents** (`/workspace/agents`) — 에이전트 직접 정의
- **Chat Archive / Trash** — 아카이브·휴지통(복구 가능)
- **Showcase** (`/showcase/[thread_id]`) — 로그인 없이 보는 읽기 전용 쇼케이스
- **Terminal Workbench (TUI)** — Gateway/프론트/nginx/Docker 없이 임베디드 실행
- **Embedded Python Client** — `DeerFlowClient`로 in-process 사용
- **Tracing** — LangSmith / Langfuse / Monocle
- **Authorization (RBAC)** — `authorization.enabled`. role별 `tools`/`routes`/`models`/`skills`/`sandbox` allow·deny.
  모든 실행 시작 라우트에 `runs:create` 요구(`POST /api/runs/stream`, `/api/runs/wait`, 예약 작업 변경 포함)
- **Extensions (Python 플러그인)** — 아래 1.7 참조

### 1.7 확장 방법 3가지

| 구분 | Skills | MCP Servers | Python Extensions |
| --- | --- | --- | --- |
| 형태 | `SKILL.md` 마크다운 | 외부 툴 서버 (stdio/HTTP) | Python 패키지 |
| 코드 실행 | 아니오 (지시문/워크플로우) | 외부 프로세스 | **예 — Gateway 권한** |
| 설정 위치 | `skills/` | `extensions_config.json` | `config.yaml` → 최상위 `plugins:` |
| Gateway 재시작 | 불필요 | 불필요 | **필수** |
| API로 수정 | 가능 | 가능 | **불가(의도적)** |
| 난이도 | 낮음 | 보통 | 높음 |
| 위험도 | 낮음 (SkillScan) | 보통 | **높음** |

**Extension이 기여할 수 있는 5가지**
1. 리드/서브에이전트의 **모델·툴 위치에 격리된 미들웨어**
2. 리드·서브에이전트 **태스크 라이프사이클 훅**
3. 미들웨어 모델 훅으로 감싸지지 않는 **DeerFlow 소유 모델 호출 옵저버**(goal, memory, title, summarization)
4. **Gateway 수명 서비스**(`ExtensionService`)
5. **eager FastAPI HTTP 라우터**

```toml
[project.entry-points."deerflow.extensions"]
acme = "acme_deerflow_extension:install"
```

```bash
make extension-install SOURCE="deerflow-extension-acme==1.2.3"
make extension-install SOURCE="git+https://github.com/acme/ext.git@<40자 커밋해시>"
make extension-install SOURCE="$PWD/examples/deerflow-extension-example"
make extension-list / extension-upgrade / extension-enable NAME=.. / extension-disable / extension-remove
```

제약·주의:
- 설치는 **대화형**(빌드 훅 실행 + 확장이 Gateway 권한으로 동작). 자동화는 `--yes`로 명시 승인.
- uv **0.8.0+** 필요(제공 Docker 이미지는 uv 0.11.1 고정).
- 원격 Git은 **공개 HTTPS만** (SSH 거부 — Docker 빌더가 호스트 SSH를 전달하지 않음).
- 소스 URL에 **자격증명 금지** (userinfo/크리덴셜형 쿼리는 uv 실행 전에 거부).
- 확장 라우터는 호스트 라우트 **이후** 마운트. 확정적 shadow나 인증/CSRF 면제 경로 진입은 거부.
  → **모든 기여 엔드포인트는 인증된 세션 필요**. 즉 **공개 웹훅/공개 상태 엔드포인트는 이번 릴리스 범위 밖.**
- `deerflow_extension_api.auth`: `resolve_principal(request)`, `require_admin(request)`(실패 시 close).
  확장은 user id / admin flag / internal flag / roles 의 **projection**만 받는다.
- 라우터 startup/shutdown 훅, 커스텀 lifespan, Mount, WebSocket은 미지원.
- `ExtensionRuntimeDeps.run_evidence_reader` — 감사·평가·동기화·관측용 **읽기 전용 안정 인터페이스**.
  재개 가능 커서로 변경된 run 발견, `after_seq`로 이벤트 페이징, 상태는 이벤트와 분리 조회.
  이벤트 **메타데이터는 비밀 리댁션**되지만 **이벤트 내용은 원본 그대로**. 변경 페이지에는 삭제 툼스톤이 없다.
- `required: true`면 로드 실패 시 startup 중단. 관리형 설치는 기본 `required: false`로 기록.

### 1.8 활용 시나리오

- **딥 리서치 → 보고서 자동 생성**
- **주제 → PPT 자동 제작**(슬라이드 이미지 생성 + 스타일 일관성 유지)
- **데이터 분석 파이프라인**(CSV 업로드 → 분석 → 차트)
- **콘텐츠 자동화**(뉴스레터·팟캐스트·영상·음악)
- **코드 작업**(샌드박스 실제 실행)
- **예약 작업**(매일 아침 뉴스 요약 등)
- **사내 봇**(슬랙/텔레그램에 붙여 팀 전체 사용)
- **학술 워크플로우**(논문 리뷰·체계적 문헌고찰)

### 1.9 보안 주의사항 (반드시 숙지)

| 항목 | 내용 |
| --- | --- |
| **에이전트가 명령을 실행한다** | 기본값이 루프백 바인딩인 근본 이유 |
| `BIND_HOST=0.0.0.0` | 자체 TLS/인증 프론트도어나 방화벽이 있을 때만. **첫 실행 설정을 먼저 완료**한 뒤 노출 |
| **Local 샌드박스** | 호스트 파일시스템 격리가 아님. 셸 접근과 함께 경계가 필요하면 Docker/AIO·K8s·E2B 사용 |
| **Lark/Feishu 통합** | per-user 자격증명 디렉터리가 샌드박스에 마운트됨 (`config`는 RO + `config/locks`만 RW, `data`는 RW). 에이전트가 실행하는 프로세스가 **읽을 수 있으므로** 프롬프트 인젝션으로 유출 가능. 사이드카 크리덴셜 브로커 후속 작업 전까지 **샌드박스는 Lark 자격증명 신뢰 경계 내부**로 간주 |
| **`config.yaml` / `extensions_config.json`** | 신뢰된 **운영자 전용** 파일. 미들웨어·커스텀 툴·모델·샌드박스·가드레일·MCP 서버/인터셉터 경로는 모두 **코드 실행**과 동일 |
| **외부 마운트 직접 쓰기** | `POST /api/skills/reload`로 리로드 가능하지만, 검증·SkillScan·이력을 **우회**. 운영자 통제 시스템만 쓰기 권한 |
| **스킬 allowed-tools** | 최선 노력 행위 스코핑이며 하드 보안 경계가 아님 |
| **민감 파일** | `config.yaml`, `extensions_config.json`, `.env`는 gitignored — **절대 커밋 금지** |
| **지원 번들** | `make support-bundle`은 리댁션된 진단만 포함(`.env`·원문 대화·사용자 파일 내용 제외) |

---

## 2. 쉽게 이해하기 (비유 설명)

### 2.1 "AI 직원을 위한 사무실"

LLM은 아주 똑똑한 신입사원이지만 혼자서는 아무것도 못 한다. 컴퓨터도, 메모장도, 매뉴얼도, 동료도 없기 때문.
**DeerFlow가 하는 일은 이 신입사원에게 사무실을 차려주는 것.**

| AI에게 필요한 것 | DeerFlow 구성요소 | 비유 |
| --- | --- | --- |
| 작업용 컴퓨터 | **Sandbox** | 격리된 책상 + PC |
| 기억 | **Memory** | 업무 노트 |
| 업무 매뉴얼 | **Skills** | 사내 매뉴얼 23권 |
| 도구 | **Tools / MCP** | 전화기·프린터·검색엔진 |
| 동료 | **Sub-Agents** | 인턴 여러 명 |
| 출퇴근/반복업무 | **Scheduler** | 매일 9시 자동 업무 |
| 소통 창구 | **IM Channels** | 슬랙·텔레그램 |
| 비서실 | **Gateway** | 요청 접수·배분 |
| 작업 규율 | **Middlewares** | 사내 규정 40개 |

> **Harness(하네스)** 는 원래 "마구(馬具)". 말(LLM)에 씌워 원하는 방향으로 일하게 만드는 장비라는 뜻.

### 2.2 실제 동작 흐름 예시

요청: "2026년 전기차 시장 조사해서 PPT로 만들어줘"

```
1. 리드 에이전트   : 작업 분해 — 리서치 + PPT 두 단계
2. describe_skill : deep-research, ppt-generation 발견 → 점진적 로딩
3. 서브에이전트 3  : 북미 / 유럽 / 아시아 시장 병렬 조사
4. Sandbox        : 데이터 수집 → 분석 → 차트 생성
5. image-generation: 슬라이드 배경 이미지 생성 (앞 슬라이드를 참조해 스타일 일관성 유지)
6. python-pptx    : PPTX 조립
7. Memory         : 사용자 관심사 기록
8. present_file   : 결과물 전달
```

### 2.3 레고 비유

| 종류 | 예시 | 난이도 |
| --- | --- | --- |
| **Skills** = 조립 설명서 | "PPT 만드는 방법" 마크다운 | 낮음 |
| **MCP** = 외부 부품 상자 | Notion, DB, Figma 연결 | 보통 |
| **Extensions** = 레고판 개조 | 미들웨어·라우터 추가 | 높음 |

---

## 3. Q&A 7문 7답

### Q1. 설치 및 사용법?

#### 방법 A — Docker (권장)

```bash
git clone https://github.com/bmshin94/deer-flow.git
cd deer-flow

make setup          # 대화형 마법사(약 2분): LLM 제공자 / 웹검색 / 샌드박스·bash·파일쓰기 권한
                    #   → config.yaml 생성 + .env 에 키 기록
make doctor         # 설정·시스템 요구사항 점검 (실행 가능한 수정 힌트 제공)
make docker-init    # Docker 사전 준비 (서비스는 아직 시작 안 됨)
make docker-start   # 개발 환경 기동
# 브라우저: http://localhost:2026
```

#### 방법 B — 로컬 개발

```bash
make config    # config.yaml + extensions_config.json 생성 (make dev 는 만들어주지 않음!)
make install   # 프론트 + 백엔드 의존성 + pre-commit 훅
make check     # node / pnpm / uv / nginx 설치 확인
make dev       # 전체 스택 핫리로드
```

> 순서 주의: `config.yaml`이 없으면 서비스가 부팅에 실패한다.

#### 방법 C — 터미널 전용 TUI (Gateway·프론트·nginx·Docker 전부 불필요)

```bash
uv pip install 'deerflow-harness[tui]'

deerflow                                # 터미널 UI (TTY 필요)
deerflow --tui-transparent              # 터미널 기본 배경 사용
deerflow --continue                     # 최근 스레드 이어서
deerflow --resume THREAD                # 스레드 ID로 재개
deerflow --print "이 레포 요약해줘"        # 헤드리스 1회 응답
deerflow --json "hello"                 # 개행 구분 StreamEvents (자동화용)
deerflow --recursion-limit 250 --print "task"
```
동일한 `config.yaml`·체크포인터·스킬·메모리·MCP·샌드박스 설정을 공유하고,
**Gateway 없이도** 공유 스레드 저장소에 기록하므로 TUI 세션이 웹 UI 사이드바에도 나타난다.
슬래시 팔레트, `/clear`(표시만 삭제), `/goal`, `/model`, `/threads`, PageUp/Down, Esc·Ctrl+C 인터럽트 지원.

#### 방법 D — 임베디드 Python 클라이언트

```python
from deerflow.client import DeerFlowClient

client = DeerFlowClient()
response = client.chat("이 논문 분석해줘", thread_id="my-thread")

for event in client.stream("hello"):
    if event.type == "messages-tuple" and event.data.get("type") == "ai":
        print(event.data["content"])

client.list_models()                                # {"models": [...]}
client.list_skills()                                # {"skills": [...]}
client.update_skill("web-search", enabled=True)
client.upload_files("thread-1", ["./report.pdf"])
client.set_goal("thread-1", "구현을 끝내고 테스트를 전부 통과시킬 것")
client.get_goal("thread-1"); client.clear_goal("thread-1")
```
`stream()`의 각 `values` 이벤트에는 현재 압축 컨텍스트 요약인 `summary_text`가 포함된다(없으면 `None`).
스레드 ID는 `^[A-Za-z0-9_-]{1,64}$` 규칙을 따른다(생략/None이면 UUID 자동 생성, 빈 문자열은 무효).

#### 주요 웹 UI 경로

| 경로 | 내용 |
| --- | --- |
| `/workspace` | 메인 작업공간 |
| `/workspace/chats` · `/workspace/chats/[id]` | 대화 목록·상세 |
| `/workspace/agents` · `/agents/new` | 커스텀 에이전트 |
| `/workspace/capabilities` | Capability Center (Plugins + Skills) |
| `/workspace/projects/[id]` | 프로젝트 |
| `/workspace/scheduled-tasks` | 예약 작업 |
| `/workspace/trash` | 휴지통 |
| `/showcase/[thread_id]` | 공개 읽기 전용 쇼케이스 |
| `/login` · `/setup` · `/auth/callback` | 인증·초기 설정 |
| `/[lang]/docs/...` · `/blog/...` | 문서·블로그 (MDX) |

#### 그 외 명령

```bash
make help                 # 전체 목록
make support-bundle       # 문제 신고용 리댁션 리포트 + AI 이슈 드래프트 + 선택적 zip
make up / make down       # 프로덕션 Docker 스택 (헬스 프로브 통과까지 대기)
make start                # 로컬 프로덕션 모드 (SKIP_FRONTEND_BUILD=1 로 빌드 재사용)
make stop                 # 전체 종료
make docker-logs[-gateway|-frontend|-redis]
make config-upgrade / setup-sandbox / detect-blocking-io
```

단일 테스트 실행:
```bash
cd backend && python -m pytest tests/test_compose_default_bind_host.py -q
cd frontend && pnpm rstest run <pattern>
```
커밋 전 필수: `cd backend && make format` (CI가 `ruff format --check`), `cd frontend && pnpm check`.

### Q2. 플러그인? 스킬? MCP?

**정답: 셋 다 아니다. DeerFlow는 이 세 가지를 모두 받아들이는 "플랫폼(호스트)"이다.**

```
                  DeerFlow = 플랫폼(슈퍼 에이전트 하네스)
                                 ↑ 여기에 꽂는 것들 ↑
        ┌──────────────┬─────────────────┬──────────────────┐
   Skills(마크다운)  MCP(외부 툴 서버)  Extensions(Python)  Tools(내장)
```

- **플러그인이 아니다** → 플러그인을 **받는 쪽**. `config.yaml`의 `plugins:`가 그 입구.
- **스킬이 아니다** → 스킬 23개를 **내장·실행**하는 쪽.
- **MCP가 아니다** → MCP 서버를 **연결해 쓰는 클라이언트**.

**단 하나의 예외**: `skills/public/claude-to-deerflow/` 는 진짜 "스킬"이다.

```bash
npx skills add https://github.com/bytedance/deer-flow --skill claude-to-deerflow
# Claude Code 에서 /claude-to-deerflow
```
가능한 작업: 메시지 전송 + 스트리밍 수신, 실행 모드 선택(flash / standard / pro / ultra),
헬스 체크, 모델·스킬·에이전트 목록, 스레드·대화 이력 관리, 파일 업로드.

| 환경변수 | 기본값 |
| --- | --- |
| `DEERFLOW_URL` | `http://localhost:2026` |
| `DEERFLOW_GATEWAY_URL` | `http://localhost:2026` |
| `DEERFLOW_LANGGRAPH_URL` | `http://localhost:2026/api/langgraph` |

### Q3. API 토큰을 사용해야 돼?

**DeerFlow 자체는 토큰이 필요 없다**(MIT 무료 오픈소스). 비용은 LLM 제공사에 지불한다.

#### 필수 — LLM 키 최소 1개 (`config.yaml` → `models:`)

| 제공자 | 환경변수 | 비고 |
| --- | --- | --- |
| Volcengine (Doubao) | `VOLCENGINE_API_KEY` | 공식 추천. **Coding Plan**(`/api/coding/v3`)이면 키 1개로 Doubao·GLM·DeepSeek·Kimi·MiniMax |
| OpenAI | `OPENAI_API_KEY` | |
| DeepSeek | `DEEPSEEK_API_KEY` | 가성비 |
| Gemini | `GEMINI_API_KEY` | |
| Novita / MiniMax / StepFun / OpenViking | 각 키 | OpenAI 호환 |
| **vLLM / 로컬 모델** | `VLLM_API_KEY` | 로컬 운영 시 **비용 0** |

모델 엔트리 예시 필드: `name`, `display_name`, `use`, `model`, `api_base`, `api_key`,
`timeout`, `max_retries`, `context_window`, `supports_thinking`, `supports_vision`,
`supports_reasoning_effort`, `when_thinking_enabled/disabled`.
선택적 per-model `request_admission`으로 제공자 RPM 한도를 맞출 수 있다(기본 비활성).

#### 선택 — 기능별

| 용도 | 키 | 없을 때 |
| --- | --- | --- |
| 웹 검색 | `TAVILY_API_KEY`, `SERPER_API_KEY`, `SERPLY_API_KEY`, `JINA_API_KEY`, `INFOQUEST_API_KEY`, `SOFYA_API_KEY`, `FIRECRAWL_API_KEY` | **DDG·SearXNG는 키 없이 무료** |
| 이미지 생성 | `IMAGE_GENERATION_API_KEY` (+ `_PROVIDER`, `_BASE_URL`, `_MODEL`, `_SIZE`) | image/ppt/video 스킬 불가 |
| 클라우드 샌드박스 | `E2B_API_KEY` | Local/Docker 모드면 불필요 |
| 메신저 | `SLACK_BOT_TOKEN`·`SLACK_APP_TOKEN`, `TELEGRAM_BOT_TOKEN`, `DISCORD_BOT_TOKEN`, `FEISHU_APP_ID/SECRET`, `WECOM_*`, `DINGTALK_*` | 해당 채널만 불가 |
| 깃허브 | `GITHUB_TOKEN` | 레이트리밋 |
| 관측 | `LANGSMITH_*`, Langfuse, Monocle | 디버깅 편의만 |
| DB | `DATABASE_URL` | SQLite 기본값 사용 |
| CORS | `GATEWAY_CORS_ORIGINS` | 통합 nginx(2026) 쓰면 불필요 |

웹 검색 공통 옵션: `time_range` = `day|week|month|year` (생략 시 기존 동작).
Tavily는 툴 엔트리별 `include_domains`/`exclude_domains` 지원(비어있지 않은 include는 `filter` 모드).
**Tavily `web_search`와 `web_fetch`는 각각 자기 엔트리의 `api_key`를 읽는다**(공유하지 않음) — 둘 다 쓰려면 양쪽에 설정하거나 `TAVILY_API_KEY` 사용.

> 비용 최소화 조합: **Ollama/vLLM(로컬 모델) + DuckDuckGo 검색 = 사실상 무료 운영.**
> 점진적 스킬 로딩이 토큰을 아껴주므로 작은 모델에서도 실용적이다.

### Q4. AI 에이전트 구축에 도움이 될까?

**매우 도움이 된다.** 활용 수준에 따라 3단계로 나뉜다.

| 레벨 | 기간 | 내용 |
| --- | --- | --- |
| **L1 그냥 사용** | 1일 | 설치 + `SKILL.md` 작성으로 업무 자동화. 코딩 거의 0 |
| **L2 커스터마이즈** | 1~2주 | 스킬 자작 + MCP 연결 + 커스텀 에이전트 + 프론트 리브랜딩 → 서비스 출시 가능 |
| **L3 학습 교재** | 1~3개월 | 직접 프레임워크를 만들 때의 최고 레퍼런스 |

**L3에서 배울 것과 위치**

| 주제 | 경로 |
| --- | --- |
| 미들웨어 체인 설계 | `backend/packages/harness/deerflow/agents/middlewares/` |
| 토큰 예산 / 컨텍스트 압축 | `token_budget_middleware.py`, `summarization_middleware.py` |
| 무한루프 방지 | `loop_detection_middleware.py` |
| 스킬 권한 스코핑 | `skill_tool_policy_middleware.py`, `skill_activation_middleware.py` |
| 서브에이전트 위임 | `subagents/`, `agents/middlewares/delegation_ledger.py` |
| 샌드박스 격리 4종 | `sandbox/` |
| 툴 지연 로딩 | `tools/builtins/tool_search.py` |
| RBAC 인증 | `authz/`, `gateway/authz.py` |
| 멀티채널 봇 | `app/channels/` |
| 확장 계약 | `backend/packages/extension-api/`, `examples/deerflow-extension-example/` |

**직접 만들 때 대비 절약 추정**

| 직접 구현 | DeerFlow |
| --- | --- |
| 샌드박스 격리 (약 2개월) | 4가지 모드 완성 |
| 장기 메모리 + 검색 랭킹 (약 1개월) | 완성 |
| 멀티채널 봇 10종 (약 2개월) | 완성 |
| 토큰/컨텍스트 관리 (약 1개월) | 미들웨어 40+ |
| 예약작업 + 큐 + 리스 펜싱 (약 3주) | 완성 |
| K8s Helm 배포 (약 2주) | `deploy/helm/` |

→ **대략 6~8개월 절약.**

**안 맞는 경우**: 단순 챗봇 1개만 필요 / 100% PHP·Java 전용 환경 / 초경량 서버리스.

### Q5. 수익화 아이디어 있어?

→ [4장 수익화 전략 9가지](#4-수익화-전략-9가지) 참조.

### Q6. React나 PHP로 만들 수 있어?

#### React — **이미 React다**

```
frontend/src/   (TS/TSX 약 489개)
├── app/        Next.js App Router
├── components/ · core/(스토어·스트리밍) · hooks/ · lib/ · styles/ · content/(MDX)
```
- Next.js App Router + TypeScript + Tailwind, 테스트는 rstest, 패키지 매니저는 pnpm
- 번들러: Webpack 기본, `DEER_FLOW_DEV_BUNDLER=turbo`로 Turbopack
- 호스트 pnpm 호출은 `scripts/pnpm.py` 경유(Windows는 `pnpm.cmd` 우선, POSIX는 역순. Corepack 동일 규칙)

```bash
cd frontend
pnpm dev     # 개발 서버
pnpm check   # 린트 + 타입체크 (커밋 전 필수)
pnpm test    # 유닛 테스트
```
→ **UI는 바로 손댈 수 있고, 프론트를 완전히 새로 만들어도 된다**(Gateway REST + SSE만 호출).

#### PHP — 2가지 길

**길 A (권장): PHP를 클라이언트/프론트로 사용.**

```php
<?php
// DeerFlow Gateway에 작업 요청 (SSE 스트리밍 프록시)
$ch = curl_init('http://localhost:2026/api/runs/stream');
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_HTTPHEADER => ['Content-Type: application/json'],
    CURLOPT_POSTFIELDS => json_encode([
        'input' => ['messages' => [['role' => 'user', 'content' => '전기차 시장 조사해줘']]],
    ]),
    CURLOPT_WRITEFUNCTION => function ($ch, $chunk) {
        echo $chunk;            // SSE 조각을 그대로 흘려보냄
        return strlen($chunk);
    },
]);
curl_exec($ch);
```
> 주의: `POST /api/runs/stream`·`/api/runs/wait`는 `runs:create` 권한을 요구하고,
> 요청 본문에 thread ID를 주면 소유권도 별도 검증된다.

**길 B (비권장): PHP로 전체 재작성.**
LangGraph/LangChain은 Python 전용이고 PHP 대체품이 사실상 없으며, Python 1,512개 파일 이식과
긴 스트리밍 실행에 대한 PHP의 request-response 모델 한계, upstream 추적 불가가 겹친다.

→ **결론: 프론트는 React(이미 구현됨), PHP는 API 소비자, 백엔드는 Python 유지.**

### Q7. 유튜브 강의 영상 제작 가능할까?

**가능하며, 타이밍이 좋다.**

| 유리한 조건 | 상태 |
| --- | --- |
| 라이선스 | MIT — 영상·강의·수익화 자유 |
| 화제성 | GitHub Trending 1위 경력 |
| 한국어 자료 | **거의 없음** (README는 영·중·일·불·러만, 한국어 없음) |
| 시각적 임팩트 | PPT·이미지·영상·음악 생성 → 썸네일 소재 풍부 |
| 분량 | 기능 다수 → 20편 이상 시리즈 가능 |
| 검색 수요 | "AI 에이전트" 키워드 상승세 |

**추천 20편 커리큘럼**

입문(1~5): ①10분 설치 ②`make setup` 완전정복 + 무료 운영(Ollama+DDG) ③주제→PPT 라이브 시연
④웹 UI 투어(Capability Center / Projects / Scheduled Tasks) ⑤TUI 사용법
중급(6~12): ⑥첫 스킬 만들기 ⑦`skill-creator`로 스킬 자동 생성 ⑧MCP 연결(Notion·DB·Figma)
⑨슬랙봇 사내 비서 ⑩서브에이전트 병렬 리서치 ⑪매일 아침 자동 리포트(예약 작업) ⑫샌드박스 4종 비교
고급(13~20): ⑬미들웨어 40개 해부 ⑭Python Extension 만들기 ⑮커스텀 에이전트 + RBAC
⑯프론트 리브랜딩 ⑰Docker/K8s 프로덕션 배포(Helm) ⑱**보안 편** ⑲PHP/워드프레스 통합 ⑳SaaS 수익화

**제작 팁**
- "무료로 돌리기" 편이 최고 유입 (Ollama + DuckDuckGo)
- 결과물 먼저 보여주고 과정 설명(역순 편집)
- **보안 경고 필수**: 루프백 기본값 이유, `BIND_HOST=0.0.0.0` 위험성, Local 샌드박스 한계
- **한국어 README PR** → 영상 신뢰도 + 컨트리뷰터 타이틀
- 재미 소재: `surprise-me` 스킬
- 비교 영상: DeerFlow vs Manus vs AutoGPT vs CrewAI
- 주의: 화면에 **API 키 노출 금지**, 버전 변화가 빠르므로 커밋 해시/날짜 표기, 출처는 "ByteDance 오픈소스 DeerFlow (MIT)"

---

## 4. 수익화 전략 9가지

> **대전제**: MIT 라이선스 — 상업적 이용·수정·재배포 자유. 조건은 **라이선스 고지 + 저작권 표시 유지**뿐. 로열티 0원.

### TIER 1 — 즉시 시작 (초기비용 ~0원)

#### ① 콘텐츠 크리에이터 (유튜브 / 블로그 / 강의)

| 항목 | 내용 |
| --- | --- |
| 투자 | 시간 |
| 수익원 | 애드센스, 유데미·인클래스 강의, 멤버십, 기업 협찬 |
| 예상 | 월 50~500만원 |
| 난이도 | 낮음 |
| 근거 | 한국어 자료 공백, 트렌딩 1위 화제성, AI 키워드 수요 |

실행: 유튜브 20편 시리즈 → 블로그 텍스트 버전(SEO) → 유료 강의(₩99,000) →
한국어 README PR(컨트리뷰터 권위) → 전자책(₩19,900)

#### ② 스킬 마켓플레이스 / 프리미엄 스킬 판매

| 항목 | 내용 |
| --- | --- |
| 투자 | 거의 0원 |
| 수익원 | 스킬 팩 판매, 구독 |
| 예상 | 월 30~300만원 |
| 난이도 | 낮음 (마크다운만 작성) |

한국 특화 스킬 팩 후보:

| 스킬 팩 | 내용 | 가격(예) |
| --- | --- | --- |
| `korean-biz-docs` | 기안서·품의서·사업계획서·주간보고 | ₩49,000 |
| `dart-analysis` | DART 전자공시 → 재무분석 리포트 | ₩99,000 |
| `smartstore-seo` | 스마트스토어 상품명·상세페이지 최적화 | ₩79,000 |
| `k-marketing` | 인스타·블로그·틱톡 한국형 콘텐츠 | ₩59,000 |
| `gov-bid` | 나라장터 공고 분석 + 제안서 초안 | ₩149,000 |
| `k-legal` | 표준계약서 검토 (법률조언 아님 면책 필수) | ₩129,000 |
| `k-academic` | KCI 논문 형식 + 한국어 학술 글쓰기 | ₩69,000 |
| `tax-prep` | 세무 자료 정리 보조 | ₩89,000 |

> 스킬은 `SKILL.md` 마크다운이라 Python 없이 제작 가능. `skill-creator`로 생산, `skill-reviewer`로 검수.

#### ③ 설치·구축 대행 (SI)

| 패키지 | 내용 | 가격(예) |
| --- | --- | --- |
| Basic | Docker 설치 + 기본 설정 + 2시간 교육 | ₩3,000,000 |
| Standard | + 커스텀 스킬 3개 + 슬랙 연동 | ₩8,000,000 |
| Pro | + K8s 배포 + RBAC + 사내DB MCP 연동 | ₩20,000,000 |
| 유지보수 | 월 구독 | ₩500,000/월 |

타겟: 중견기업 기획·마케팅팀, 컨설팅펌, 연구소, 대학 연구실, 로펌.

### TIER 2 — 본격 사업 (1~6개월)

#### ④ 버티컬 SaaS (잠재력 최대)

| 서비스(예) | 타겟 | 핵심 기능 | 가격(예) |
| --- | --- | --- | --- |
| 리포트봇 | 마케팅 대행사 | 광고 성과 → 주간 리포트 자동화 | ₩99,000/월 |
| 공시읽어줌 | 개인투자자 | DART 공시 → 요약 + 시사점 | ₩29,000/월 |
| IR덱 메이커 | 스타트업 | 사업 정보 → 투자용 PPT | ₩149,000/월 |
| 논문요약왕 | 대학원생 | 논문 업로드 → 리뷰 + 레퍼런스 | ₩19,000/월 |
| 셀러AI | 이커머스 셀러 | 상품 등록 + SEO + 경쟁사 모니터링 | ₩79,000/월 |
| 뉴스레터팩토리 | 1인 미디어 | 매일 자동 발행 | ₩49,000/월 |
| 입찰헌터 | 중소기업 | 나라장터 모니터링 + 제안서 초안 | ₩199,000/월 |

**원가 차익 구조**
```
고객 과금     월 ₩99,000
실제 LLM 원가 월 ₩15,000  (저가 모델 + 점진적 스킬 로딩으로 토큰 절약)
──────────────────────────
마진율        약 85%
```

**반드시 활용할 기능**
- `authorization.enabled` + **RBAC** → 멀티테넌시(고객사별 tools/routes/models/skills/sandbox 분리)
- `scheduler` → 자동 실행 = 구독 유지의 핵심
- `sandbox` **E2B / K8s 모드** → 고객 간 격리 (Local 모드는 격리가 아님)
- `extensions` 미들웨어 → 토큰 사용량 미터링 → 과금 산정
- `run_evidence_reader` → 감사 로그 (엔터프라이즈 영업 포인트).
  단, 이벤트 **메타데이터만 리댁션**되고 내용은 원본이며, 삭제 툼스톤이 없으므로 삭제 동기화는 상태 폴링으로 처리
- `DeerFlowClient` → Gateway 없이 임베디드 경량 운영

#### ⑤ 워드프레스 플러그인 / PHP 통합

전세계 웹사이트의 약 40%가 워드프레스인데 AI 에이전트 플러그인은 희소하다.

| 플러그인 | 기능 | 가격(예) |
| --- | --- | --- |
| AI Blog Writer | 딥리서치 → 자동 포스팅 | $49/년 |
| AI 상담 챗봇 | 방문자 상담(제품 DB MCP 연동) | $99/년 |
| SEO 자동 최적화 | 경쟁사 분석 → 메타·키워드 | $79/년 |
| 리포트 위젯 | 관리자 대시보드 AI 리포트 | $59/년 |

```
WordPress(PHP)  ──HTTP/SSE──▶  DeerFlow Gateway(Python)  ──▶  LLM
```
판매처: CodeCanyon, 자체 사이트, WordPress.org(Freemium).
한국형: 카페24 / 아임웹 / 그룹웨어 연동 앱도 동일 구조.

#### ⑥ 교육 · 부트캠프

| 상품 | 내용 | 가격(예) |
| --- | --- | --- |
| 기업 출강 | 1일 8시간 실습 워크숍 | ₩3,000,000/일 |
| 부트캠프 | 4주 "AI 에이전트 엔지니어"(20명) | ₩1,500,000/인 |
| 온라인 강의 | 유데미·인클래스 | ₩99,000 |
| 1:1 멘토링 | 주 1회 | ₩300,000/월 |

K-디지털 트레이닝 등 국비지원 과정 등록 시 모집이 쉬워진다.

### TIER 3 — 장기 플레이

#### ⑦ 한국 특화 포크 배포 ("K-DeerFlow")
한국어 UI 완전 지원, 카카오톡·네이버웍스 채널 추가, 네이버 검색·클로바 모델 연동,
DART·나라장터·공공데이터 MCP 내장. 오픈소스 무료 배포 + 구축·지원 수익(Red Hat 모델).
커뮤니티가 최강의 영업 자산.

#### ⑧ 전문직 AI 어시스턴트 (고마진)

| 분야 | 서비스 | 가격(예) |
| --- | --- | --- |
| 법률 | 판례 리서치 + 서면 초안 | ₩500,000/월 |
| 의료 | 논문 모니터링 + 요약 | ₩300,000/월 |
| 세무 | 세법 개정 모니터링 | ₩200,000/월 |
| 건설 | 입찰 공고 + 견적 보조 | ₩400,000/월 |

면책조항 + 전문가 최종 검토 + 규제 검토 필수.

#### ⑨ 에이전트 성능 컨설팅
토큰 비용 절감 컨설팅(점진적 스킬 로딩 적용으로 30~70% 절감 가능),
프롬프트·컨텍스트 엔지니어링 감사, 자매 프로젝트 **LLM Space**로 각 단계 검사·실패 리플레이·벤치마크.
진단 ₩5,000,000 / 개선 ₩20,000,000+.

### 최종 비교표

| # | 아이디어 | 초기투자 | 난이도 | 수익 잠재력 | 회수기간 | 추천도 |
| --- | --- | --- | --- | --- | --- | --- |
| ① | 콘텐츠/유튜브 | 낮음 | ★★ | ★★ | 1~3개월 | **즉시 시작** |
| ② | 스킬 판매 | 낮음 | ★★ | ★★ | 1~2개월 | **즉시 시작** |
| ③ | 구축 대행 | 낮음 | ★★★ | ★★★ | 1개월 | 높음 |
| ④ | **SaaS** | 높음 | ★★★★ | ★★★★★ | 6~12개월 | **최고 잠재력** |
| ⑤ | WP 플러그인 | 보통 | ★★★ | ★★★ | 3~6개월 | 높음 |
| ⑥ | 교육 | 보통 | ★★★ | ★★★ | 2~4개월 | 높음 |
| ⑦ | 한국형 포크 | 높음 | ★★★★★ | ★★★★ | 12개월+ | 보통 |
| ⑧ | 전문직 AI | 높음 | ★★★★ | ★★★★ | 6개월 | 높음 |
| ⑨ | 컨설팅 | 낮음 | ★★★★★ | ★★★★ | 3개월 | 보통 |

### 추천 3단 전략

```
[1단계 0~3개월] 인지도 + 실력
  ① 유튜브 시리즈 시작 (한국어 1호)
  ② 스킬 팩 2~3개 제작·판매
  + 한국어 README PR → 컨트리뷰터 타이틀
  목표: 월 100만원 + 포트폴리오

[2단계 3~9개월] 수익 본격화
  ③ 구축 대행 (유튜브 구독자가 리드로 전환)
  ⑥ 기업 교육 출강
  ④ SaaS 1종 선택 → MVP 개발 착수
  목표: 월 500만원

[3단계 9개월~] 스케일업
  ④ SaaS 정식 출시 → MRR 축적
  ⑤ 워드프레스 플러그인으로 글로벌 진출
  목표: 월 2,000만원+
```

핵심 인사이트: **①②가 ③④⑥의 영업 채널이 된다.**
콘텐츠로 신뢰 축적 → 구축·교육 문의 → 그 과정에서 발견한 실제 니즈로 SaaS 설계.

### 법적 체크리스트

- MIT 라이선스 **고지 + 저작권 표시 유지** (필수)
- "ByteDance 공식/제휴" 사칭 금지 → "DeerFlow 기반"으로 표기
- 사용 LLM 제공사의 **상업적 이용 약관** 확인
- 개인정보 처리 시 **개인정보보호법** 준수 (B2B는 더 엄격)
- 전문 분야(법률·의료·세무)는 **면책조항 필수**
- 재판매 시 고객에게 LLM API 비용 구조 투명 공개

---

## 5. 실행 체크리스트

### 설치 (처음 1회)
- [ ] `git clone https://github.com/bmshin94/deer-flow.git && cd deer-flow`
- [ ] `make setup` (또는 `make config` 수동)
- [ ] LLM 키 1개 이상 `.env` + `config.yaml -> models:` 설정
- [ ] `make doctor` 통과 확인
- [ ] Docker: `make docker-init` → `make docker-start` / 로컬: `make check` → `make install` → `make dev`
- [ ] `http://localhost:2026` 접속 확인
- [ ] `config.yaml` / `extensions_config.json` / `.env` 가 커밋되지 않았는지 확인 (gitignored)

### 탐색 (1주)
- [ ] 내장 스킬 23개 하나씩 실행해보기 (`/deep-research`, `/ppt-generation` 등)
- [ ] Capability Center에서 스킬 on/off 체감
- [ ] TUI(`deerflow`)로 터미널 워크플로우 체험
- [ ] 예약 작업 1개 등록해 자동 실행 확인
- [ ] 슬랙 또는 텔레그램 봇 1개 연동

### 커스터마이즈 (2~4주)
- [ ] 내 업무용 `SKILL.md` 1개 작성 → `skill-reviewer`로 검수
- [ ] MCP 서버 1개 연결
- [ ] 커스텀 에이전트 1개 정의
- [ ] `examples/deerflow-extension-example` 읽고 Extension 구조 파악
- [ ] 프론트 브랜딩 변경 (`frontend/`)

### 수익화 (1~3개월)
- [ ] 유튜브 1편 업로드 (설치 + 무료 운영 편)
- [ ] 한국어 README 번역 PR 제출
- [ ] 스킬 팩 1종 완성 → 판매 채널 오픈
- [ ] 구축 대행 패키지 가격표 작성
- [ ] SaaS 아이디어 1종 선정 → 타겟 인터뷰 5건

### 보안 (운영 전 필수)
- [ ] `BIND_HOST`는 기본 `127.0.0.1` 유지 (외부 노출 시 TLS/인증 프론트도어 필수)
- [ ] 샌드박스는 Docker/AIO·K8s·E2B 중 선택 (Local은 격리 아님)
- [ ] `config.yaml` / `extensions_config.json` 쓰기 권한은 운영자만
- [ ] Lark 통합 사용 시 샌드박스가 자격증명 신뢰 경계 내부임을 인지
- [ ] 다중 테넌트 운영 시 `authorization.enabled` + RBAC 구성
- [ ] 이슈 제출 시 `make support-bundle` 사용 (리댁션된 진단만 포함)

---

## 부록 A. 자주 쓰는 명령 치트시트

```bash
# 루트 (전체 스택)
make help / setup / doctor / support-bundle / config / config-upgrade / check / install
make dev / start / stop / clean
make docker-init / docker-start / docker-stop / docker-logs
make up / down                     # 프로덕션 Docker (헬스 프로브 대기)
make extension-install SOURCE=... / extension-list / extension-enable NAME=... / extension-remove NAME=...

# 백엔드
cd backend && make dev             # Gateway (8001) 리로드
cd backend && make test            # 기본 스위트 (live·blocking-IO 제외)
cd backend && make test-blocking-io
cd backend && make lint / format   # ruff (CI가 format --check 강제)
cd backend && python -m pytest tests/path/to/test.py::test_func -q

# 프론트엔드
cd frontend && pnpm dev            # Webpack 기본 (DEER_FLOW_DEV_BUNDLER=turbo)
cd frontend && pnpm check          # 린트 + 타입체크 (커밋 전 필수)
cd frontend && pnpm test
cd frontend && pnpm rstest run <pattern>

# TUI
deerflow / --continue / --resume THREAD / --print "..." / --json "..."

# 스킬 리뷰 CLI
cd backend && uv run python -m deerflow.skills.review.cli ../skills/public/data-analysis \
  --format text --fail-on error --fail-on-incomplete
```

## 부록 B. 문서 지도

| 문서 | 내용 |
| --- | --- |
| `README.md` (+ `_zh/_ja/_fr/_ru`) | 프로젝트 전체 개요·사용법 |
| `AGENTS.md` | 모노레포 오리엔테이션 (AI 코딩 에이전트용 가이드, 진실의 원천) |
| `CLAUDE.md` | `@AGENTS.md` 임포트 shim (직접 수정 금지) |
| `backend/AGENTS.md` | 백엔드 심화 (하네스/앱 분리, 미들웨어, 샌드박스, MCP, 영속화, 설정) |
| `frontend/AGENTS.md` | 프론트 심화 (App Router, 스트리밍 데이터 흐름, 코드 스타일) |
| `Install.md` | 코딩 에이전트용 부트스트랩 지시서 |
| `CONTRIBUTING.md` | 기여 가이드 |
| `SECURITY.md` | 보안 정책 |
| `RELEASING.md` | 릴리스 절차 (버전 4곳 lockstep) |
| `CHANGELOG.md` | 변경 이력 |
| `docs/ARCHITECTURE.md` | 아키텍처 |
| `backend/docs/CONFIGURATION.md` | 설정 스키마 상세 |
| `backend/docs/TUI.md` | TUI 전체 가이드 |
| `backend/docs/MCP_SERVER.md` | MCP 세션 노트 |
| `backend/packages/harness/deerflow/extensions/AGENTS.md` | 확장 매니저·기여 계약 |
| `docs/plans/2026-07-10-pluggable-authorization-rfc.md` | 인가 RFC |

> 버전 동기화 주의: 릴리스 버전은 `backend/pyproject.toml`, `frontend/package.json`,
> `deploy/helm/deer-flow/Chart.yaml`(`version` + `appVersion`)에서 **완전히 일치**해야 하며,
> `v*` 태그 푸시 시 `scripts/verify_versions.sh`가 불일치를 발견하면 퍼블리싱을 전면 차단한다.
> 버전 변경은 `scripts/bump_version.sh <ver>` → `scripts/verify_versions.sh <ver>` 순서로.

---

*이 문서는 DeerFlow 2.0 저장소(커밋 `ade55e7` 기준)를 전수조사해 작성되었습니다.*
*DeerFlow는 MIT 라이선스 오픈소스이며, 본 문서는 비공식 한국어 분석 자료입니다.*
