---
name: chrome-extension-consultant
description: Chrome 확장 프로그램 개발 컨설턴트
---

# Chrome Extension Development Consultant

Transform browser extension ideas into comprehensive specifications through orchestrated multi-agent collaboration. Specialized for Manifest V3 Chrome extensions.

## Tech Stack

- **Manifest**: Version 3 (MV3)
- **Background**: Service Worker
- **UI**: HTML + Tailwind CSS + Vanilla JS
- **Storage**: chrome.storage API
- **Messaging**: chrome.runtime messaging

## When to Use

**Use when:**
- Building Chrome/Edge browser extensions
- Need to enhance web browsing experience
- Want to integrate with browser APIs
- Content script injection required
- Cross-site functionality needed

**Don't use when:**
- Full web application needed
- Server-side processing required
- Non-browser platform targeted
- Manifest V2 features required (deprecated)

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
│   UI Sketcher     │ → Popup/Options wireframes
│                   │ → .shared/02-wireframes.md
└───────────────────┘
            ↓
┌───────────────────┐
│  UX Spec Writer   │ → Extension UX specifications
│                   │ → .shared/03-ux-specification.md
└───────────────────┘
            ↓
┌─────────────────────────────────────────────────────────────┐
│                   Phase 2: Specification                     │
│                      (Parallel)                              │
└─────────────────────────────────────────────────────────────┘
            ↓
┌───────────────────┬───────────────────┬───────────────────┐
│Extension Architect│ Mermaid Designer  │Interactive Design │
│ (MV3 + Messaging) │ (Flow diagrams)   │ (Animations)      │
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
│   Browser QA      │ → Extension testing
│                   │ → .shared/08-qa-report.md
└───────────────────┘
```

## 8 Agent Summary

| # | Agent | Output | Purpose |
|---|-------|--------|---------|
| 1 | Interviewer (or Hell) | 01-requirements.md | Extract requirements |
| 2 | UI Sketcher | 02-wireframes.md | Popup/Options UI |
| 3 | UX Spec Writer | 03-ux-specification.md | Extension UX docs |
| 4 | **Extension Architect** | 04-tech-architecture.md | MV3 architecture |
| 5 | Mermaid Designer | 05-flow-diagrams.md | Message flow diagrams |
| 6 | Interactive Designer | 06-animations.md | UI animations |
| 7 | Planner | 07-roadmap.md | Development roadmap |
| 8 | Browser QA | 08-qa-report.md | Extension testing |

## .shared Folder Structure

```
.shared/
├── 01-requirements.md         # What to build
├── 02-wireframes.md           # Popup/Options layouts
├── 03-ux-specification.md     # Extension UX patterns
├── 04-tech-architecture.md    # MV3 + Messaging architecture
├── 05-flow-diagrams.md        # Message flow diagrams
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
- Follow Manifest V3 requirements

MUST NOT DO:
- Skip reading previous phase outputs
- Use deprecated MV2 patterns
- Request unnecessary permissions
- Violate Chrome Web Store policies
```

## Extension-Specific Considerations

### Manifest V3 Requirements
- Service Worker (not background pages)
- Declarative Net Request (not webRequest blocking)
- Promise-based APIs
- No remote code execution

### Permission Strategy
- Minimal permissions principle
- Optional permissions for advanced features
- Host permissions scoped narrowly
- User consent for sensitive actions

### Component Architecture
- Background: Service Worker (event-driven)
- Content Scripts: Isolated world
- Popup: Quick actions (closes on blur)
- Options: Settings management
- Side Panel: Persistent UI (Chrome 114+)

### Message Passing
- chrome.runtime.sendMessage (one-time)
- chrome.runtime.connect (long-lived)
- chrome.tabs.sendMessage (to content scripts)

### Storage Types
| Type | Limit | Sync | Use Case |
|------|-------|------|----------|
| sync | 100KB | Yes | User settings |
| local | 5MB | No | Large data |
| session | - | No | Temporary |

## Extension UI Patterns

### Popup (400x600 max)
```
┌────────────────────────────┐
│ [Icon] Extension Name   [×]│
├────────────────────────────┤
│                            │
│   Main functionality       │
│   Quick actions            │
│                            │
├────────────────────────────┤
│ [Settings]    [Help]       │
└────────────────────────────┘
```

### Content Script Injection
- Floating UI elements
- Page modification overlays
- Context-aware sidebars

## Final Outputs

8 comprehensive markdown documents in `.shared/`:
1. Requirements with extension use cases
2. ASCII wireframes for popup/options
3. UX specification for browser integration
4. Manifest V3 technical architecture
5. Mermaid message flow diagrams
6. Animation and interaction specs
7. Prioritized development roadmap
8. Extension QA test report

## Reference Files

- `references/workflow.md` - Detailed workflow guide
- `references/manifest-v3-guide.md` - MV3 patterns
- `references/extension-architecture.md` - Architecture patterns
- `references/chrome-storage-patterns.md` - Storage best practices
- `references/shared-folder-spec.md` - Output standards
- `references/common-agent-tools.md` - Tool usage
