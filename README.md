# Dev Consultant Plugin

모호한 고객 요구사항을 상세한 웹 애플리케이션 명세서로 변환하는 Claude Code 플러그인입니다.

## Overview

**8개의 전문화된 에이전트**가 구조화된 워크플로우를 통해 초기 컨셉부터 QA 리포트까지 포괄적인 문서를 생성합니다.

**타겟 기술 스택**: HTML + Tailwind CSS + 바닐라 JavaScript (클라이언트 전용)

## Project Structure

```
dev-consultant/
├── agents/                          # 8개 에이전트
│   ├── interviewer/AGENT.md         # 요구사항 추출
│   ├── ui-sketcher/AGENT.md         # ASCII 와이어프레임
│   ├── ux-spec-writer/AGENT.md      # UX 명세서
│   ├── client-tech-architect/AGENT.md # 기술 아키텍처
│   ├── mermaid-designer/AGENT.md    # 플로우 다이어그램
│   ├── interactive-designer/AGENT.md # 애니메이션 명세
│   ├── planner/AGENT.md             # 프로젝트 계획
│   └── browser-qa/AGENT.md          # 브라우저 QA
├── skills/
│   └── webapp-consultant/
│       ├── SKILL.md                 # 스킬 정의
│       └── references/              # 레퍼런스 (14개)
└── .claude-plugin/
    └── plugin.json
```

## 8 Specialized Agents

| # | Agent | Output | Purpose |
|---|-------|--------|---------|
| 1 | **Interviewer** | `01-requirements.md` | 모호한 요구사항에서 명확한 요구사항 추출 |
| 2 | **UI Sketcher** | `02-wireframes.md` | ASCII 와이어프레임 + Tailwind 힌트 |
| 3 | **UX Spec Writer** | `03-ux-specification.md` | UX 철학 (Norman/Nielsen) 통합 명세 |
| 4 | **Client Tech Architect** | `04-tech-architecture.md` | 데이터 모델 + Repository 패턴 |
| 5 | **Mermaid Designer** | `05-flow-diagrams.md` | 사용자 플로우 다이어그램 |
| 6 | **Interactive Designer** | `06-animations.md` | Tailwind 애니메이션 명세 |
| 7 | **Planner** | `07-roadmap.md` | MoSCoW/RICE 우선순위 + 로드맵 |
| 8 | **Browser QA** | `08-qa-report.md` | Chrome 자동화 QA 테스트 |

## Workflow

```
┌────────────────────────────────────────────────────────────────┐
│                      고객 요청 (모호)                            │
└───────────────────────────┬────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────────┐
│  Phase 1: Discovery (순차)                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Interviewer  │→│ UI Sketcher  │→│ UX Spec Writer│          │
│  │ (요구사항)    │  │ (와이어프레임) │  │ (UX 명세)     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└───────────────────────────┬────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────────┐
│  Phase 2: Specification (병렬)                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Client Tech  │  │   Mermaid    │  │ Interactive  │          │
│  │  Architect   │  │   Designer   │  │   Designer   │          │
│  │ (기술 설계)   │  │ (플로우)      │  │ (애니메이션)  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└───────────────────────────┬────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────────┐
│  Phase 3: Final (순차)                                          │
│  ┌──────────────┐  ┌──────────────┐                            │
│  │   Planner    │→│  Browser QA  │                            │
│  │ (로드맵)      │  │ (QA 테스트)   │                            │
│  └──────────────┘  └──────────────┘                            │
└───────────────────────────┬────────────────────────────────────┘
                            ↓
┌────────────────────────────────────────────────────────────────┐
│            최종 출력: .shared/ 폴더에 8개 문서                    │
└────────────────────────────────────────────────────────────────┘
```

## .shared Collaboration System

모든 에이전트는 `.shared/` 폴더에 결과물을 저장하여 다음 에이전트에게 전달합니다.

```
[target-repository]/.shared/
├── 01-requirements.md        ← Interviewer
├── 02-wireframes.md          ← UI Sketcher
├── 03-ux-specification.md    ← UX Spec Writer
├── 04-tech-architecture.md   ← Client Tech Architect
├── 05-flow-diagrams.md       ← Mermaid Designer
├── 06-animations.md          ← Interactive Designer
├── 07-roadmap.md             ← Planner
└── 08-qa-report.md           ← Browser QA
```

### Dependency Graph

```
01-requirements
       ↓
02-wireframes
       ↓
03-ux-specification
       ↓
    ┌──┼──┐
    ↓  ↓  ↓
   04 05 06  (병렬 실행)
    └──┼──┘
       ↓
07-roadmap
       ↓
08-qa-report (실행 중인 앱 필요)
```

## Installation

```bash
# Claude Code 마켓플레이스에서 설치
/plugin marketplace add Mineru98/dev-consultant
/plugin install dev-consultant
```

## Usage

### 1. webapp-consultant 스킬 사용

```
User: "작업 관리 도구가 필요해요"

Claude (webapp-consultant 스킬 자동 실행):
1. Interviewer - 요구사항 명확화 질문
2. UI Sketcher - ASCII 와이어프레임 생성
3. UX Spec Writer - UX 명세서 작성
4. [병렬] Client Tech Architect + Mermaid Designer + Interactive Designer
5. Planner - 개발 로드맵 생성
6. Browser QA - QA 테스트 (구현 후)

Output: .shared/ 폴더에 8개 문서 생성
```

### 2. 개별 에이전트 사용

```javascript
Task({
  subagent_type: "interviewer",
  prompt: "개인 재무 추적기 요구사항 추출",
  description: "요구사항 수집"
})
```

## When to Use

### ✅ 사용해야 할 때
- 고객 요청이 모호하거나 불명확할 때
- 새 웹 앱을 처음부터 계획할 때
- 코딩 전 포괄적인 문서가 필요할 때
- HTML + Tailwind + 바닐라 JS 앱 구축 시
- 클라이언트 전용 (백엔드 없음) 프로젝트

### ❌ 사용하지 말아야 할 때
- 요구사항이 이미 명확할 때
- 백엔드/서버 사이드 애플리케이션 구축 시
- React/Vue 등 프레임워크 특화 계획 필요 시

## Tech Stack

| 계층 | 기술 |
|------|------|
| Structure | HTML5 |
| Styling | Tailwind CSS |
| Logic | Vanilla JavaScript (ES6+) |
| Settings | LocalStorage |
| Data | IndexedDB (via localbase) |
| Backend | None (client-only) |

### Constraints
- 브라우저 스토리지 제한: ~50MB
- 서버 동기화 없음 (로컬 전용)
- 클라이언트 사이드 필터링 (SQL 없음)

## Final Output

8개의 포괄적인 Markdown 문서:

1. **01-requirements.md** - 문제 정의, 사용자 페르소나, MoSCoW 기능 분류
2. **02-wireframes.md** - ASCII 와이어프레임 + Tailwind 힌트
3. **03-ux-specification.md** - 사용자 스토리, Norman/Nielsen UX 분석
4. **04-tech-architecture.md** - 데이터 모델, Repository 클래스
5. **05-flow-diagrams.md** - Mermaid.js 사용자 플로우
6. **06-animations.md** - Tailwind 애니메이션 코드
7. **07-roadmap.md** - MoSCoW 우선순위, WBS, 마일스톤
8. **08-qa-report.md** - 테스트 결과, 발견된 이슈

**총 분량**: 약 95-124 페이지의 포괄적인 프로젝트 문서

## Example

### Input
```
"작업 관리 도구가 필요해요"
```

### Process
```
[1] Interviewer
    Q: "개인용인가요, 팀용인가요?"
    Q: "카테고리나 마감일이 필요한가요?"
    → 01-requirements.md

[2] UI Sketcher
    - 작업 목록, 추가 양식, 필터 와이어프레임
    → 02-wireframes.md

[3] UX Spec Writer
    - 사용자 스토리, UX 분석
    → 03-ux-specification.md

[4-6] 병렬 실행
    - Tech: Task 컬렉션, TaskRepository
    - Mermaid: CRUD 플로우
    - Interactive: hover, spinner
    → 04, 05, 06.md

[7] Planner
    - Phase 1 (CRUD), Phase 2 (필터), Phase 3 (폴리시)
    → 07-roadmap.md

[8] Browser QA
    - 기능, UI, 접근성 테스트
    → 08-qa-report.md
```

### Output
`.shared/` 폴더에 8개의 문서 → 즉시 개발 시작 가능

## References

| Task | Reference File |
|------|----------------|
| 전체 프로세스 | `references/workflow.md` |
| 에이전트 협업 | `references/shared-folder-spec.md` |
| 공통 도구 | `references/common-agent-tools.md` |
| 상세 가이드 | `references/skill-detailed-guide.md` |
| 사용 예제 | `references/skill-usage-examples.md` |
| 인터뷰 패턴 | `references/interview-patterns.md` |
| ASCII 가이드 | `references/ascii-art-guide.md` |
| UX 철학 | `references/ux-philosophy.md` |
| localbase API | `references/localbase-guide.md` |
| Mermaid 패턴 | `references/mermaid-patterns.md` |
| 애니메이션 | `references/tailwind-animations.md` |
| 계획 방법론 | `references/planning-methods.md` |
| 기술 스택 | `references/spa-tech-stacks.md` |

## Credits

- Norman, Don. "The Design of Everyday Things"
- Nielsen, Jakob. "Usability Heuristics"
- ai-diagrams-toolkit (Mermaid patterns)
- localbase (IndexedDB wrapper)

## License

GPL-3.0 - See [LICENSE](LICENSE) file

## Version

Current version: 1.1.0
