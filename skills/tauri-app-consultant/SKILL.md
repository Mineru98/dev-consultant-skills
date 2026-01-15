---
name: tauri-app-consultant
description: Tauri 기반 크로스플랫폼 데스크톱/모바일 앱 개발 컨설턴트
---

# Tauri App Development Consultant

Transform vague desktop application ideas into comprehensive project specifications through orchestrated multi-agent collaboration. Specialized for Tauri v2 cross-platform applications.

## Tech Stack

- **Frontend**: React + TypeScript
- **Backend**: Rust
- **Framework**: Tauri v2
- **Build**: Vite + Tauri CLI
- **Storage**: SQLite (via sqlx) + Tauri State
- **Styling**: Tailwind CSS

## When to Use

**Use when:**
- Building cross-platform desktop applications
- Need native performance with web technologies
- Require system-level integrations (filesystem, notifications)
- Want smaller bundle size than Electron
- Building desktop + mobile from same codebase

**Don't use when:**
- Simple static website needed
- Server-side rendering required
- Web-only application is sufficient
- No need for native capabilities

## Agent Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                    Phase 1: Discovery                        │
│                      (Sequential)                            │
└─────────────────────────────────────────────────────────────┘
            ↓
┌───────────────────┐
│   Interviewer     │ → Ask: Standard or Hell Interviewer?
│ (or Hell Int.)    │ → .shared/01-requirements.md
└───────────────────┘
            ↓
┌───────────────────┐
│   UI Sketcher     │ → Desktop-first wireframes
│                   │ → .shared/02-wireframes.md
└───────────────────┘
            ↓
┌───────────────────┐
│  UX Spec Writer   │ → UX specifications
│                   │ → .shared/03-ux-specification.md
└───────────────────┘
            ↓
┌─────────────────────────────────────────────────────────────┐
│                   Phase 2: Specification                     │
│                      (Parallel)                              │
└─────────────────────────────────────────────────────────────┘
            ↓
┌───────────────────┬───────────────────┬───────────────────┐
│ Tauri Architect   │ Mermaid Designer  │Interactive Design │
│ (Rust backend)    │ (Flow diagrams)   │ (Animations)      │
│ 04-tech-arch.md   │ 05-flow-diag.md   │ 06-animations.md  │
└───────────────────┴───────────────────┴───────────────────┘
            ↓
┌─────────────────────────────────────────────────────────────┐
│                     Phase 3: Final                           │
│                    (Sequential)                              │
└─────────────────────────────────────────────────────────────┘
            ↓
┌───────────────────┐
│     Planner       │ → Project roadmap
│                   │ → .shared/07-roadmap.md
└───────────────────┘
            ↓
┌───────────────────┐
│   Browser QA      │ → QA testing (Tauri dev window)
│                   │ → .shared/08-qa-report.md
└───────────────────┘
```

## 8 Agent Summary

| # | Agent | Output | Purpose |
|---|-------|--------|---------|
| 1 | Interviewer (or Hell) | 01-requirements.md | Extract requirements |
| 2 | UI Sketcher | 02-wireframes.md | Desktop wireframes |
| 3 | UX Spec Writer | 03-ux-specification.md | UX documentation |
| 4 | **Tauri Architect** | 04-tech-architecture.md | Rust + Tauri design |
| 5 | Mermaid Designer | 05-flow-diagrams.md | Flow visualizations |
| 6 | Interactive Designer | 06-animations.md | Animation specs |
| 7 | Planner | 07-roadmap.md | Development roadmap |
| 8 | Browser QA | 08-qa-report.md | Quality testing |

## .shared Folder Structure

```
.shared/
├── 01-requirements.md         # What to build
├── 02-wireframes.md           # Desktop-first layouts
├── 03-ux-specification.md     # UX patterns
├── 04-tech-architecture.md    # Tauri + Rust architecture
├── 05-flow-diagrams.md        # Mermaid diagrams
├── 06-animations.md           # Animation specifications
├── 07-roadmap.md              # Development plan
└── 08-qa-report.md            # Test results
```

## Starting the Workflow

When user invokes this skill:

1. **Ask Interviewer Mode**:
```
Select interviewer mode:
- Standard (Quick, 2-3 questions, ~5 min)
- Hell Interviewer (Thorough, detailed exploration, 20-45 min)
```

2. **Launch Selected Interviewer**
3. **Follow Sequential → Parallel → Sequential flow**

## Agent Delegation Format

```markdown
TASK: [Specific goal]
EXPECTED OUTCOME: [Output file]
REQUIRED AGENT: [Agent name from table above]
CONTEXT: [Required input files]

MUST DO:
- Read previous outputs from .shared/
- Follow agent's AGENT.md guidelines
- Write output to specified file

MUST NOT DO:
- Skip reading previous phase outputs
- Make assumptions without documenting
- Deviate from Tauri/React/Rust tech stack
```

## Tauri-Specific Considerations

### Desktop UI Patterns
- Window management (resize, minimize, close)
- Menu bar and system tray
- Native dialogs (file picker, alerts)
- Keyboard shortcuts (Ctrl/Cmd+key)
- Multi-window support

### Rust Backend Focus
- Tauri commands for all backend logic
- SQLite for persistent data
- Tauri State for runtime state
- Error handling with Result types

### Cross-Platform
- Windows, macOS, Linux support
- Platform-specific behaviors documented
- Conditional compilation (#[cfg])

## Final Outputs

8 comprehensive markdown documents in `.shared/`:
1. Requirements with user stories
2. ASCII wireframes for desktop layout
3. UX specification with Norman principles
4. Tauri + Rust technical architecture
5. Mermaid flow diagrams
6. Animation and interaction specs
7. Prioritized development roadmap
8. QA test report

## Reference Files

- `references/workflow.md` - Detailed workflow guide
- `references/tauri-architecture.md` - Tauri patterns
- `references/rust-patterns.md` - Rust best practices
- `references/tauri-commands.md` - Command patterns
- `references/shared-folder-spec.md` - Output standards
- `references/common-agent-tools.md` - Tool usage
