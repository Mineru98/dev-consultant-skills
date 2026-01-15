---
name: webapp-consultant
description: Transform vague requirements into detailed specifications for HTML + Tailwind + vanilla JS web apps. Use when customers describe features vaguely, need project documentation, or want technical specifications without backend.
---

# Web App Development Consultant

Orchestrates 8 specialized agents to transform vague customer requests into complete project specifications.

## Tech Stack

- **Frontend**: HTML + Tailwind CSS + vanilla JavaScript (ES6+)
- **Storage**: LocalStorage (settings) + IndexedDB via localbase (data)
- **Backend**: None (client-side only)

## Agent Workflow

```
Customer Request (vague)
        ↓
  [1. Interviewer] ← Extract requirements
        ↓
  [2. UI Sketcher] → ASCII wireframes
        ↓
  [3. UX Spec Writer] → Technical spec with UX philosophy
        ↓
  ┌─────┼─────┐ (Parallel)
  ↓     ↓     ↓
[4]   [5]   [6]
Tech  Mermaid Interactive
Arch  Designer Designer
  └─────┼─────┘
        ↓
  [7. Planner] → Development roadmap
        ↓
  [8. Browser QA] → Quality verification
```

## 8 Agents

| # | Agent | Output | Purpose |
|---|-------|--------|---------|
| 1 | Interviewer | `01-requirements.md` | Extract clear requirements |
| 2 | UI Sketcher | `02-wireframes.md` | ASCII wireframes |
| 3 | UX Spec Writer | `03-ux-specification.md` | UX spec with Norman/Nielsen |
| 4 | Client Tech Architect | `04-tech-architecture.md` | Data architecture |
| 5 | Mermaid Designer | `05-flow-diagrams.md` | User flow diagrams |
| 6 | Interactive Designer | `06-animations.md` | Tailwind animations |
| 7 | Planner | `07-roadmap.md` | MoSCoW priorities, phases |
| 8 | Browser QA | `08-qa-report.md` | QA test report |

## .shared Folder

All agents write outputs to `.shared/` in the target repository:

```
[target-repo]/.shared/
├── 01-requirements.md
├── 02-wireframes.md
├── 03-ux-specification.md
├── 04-tech-architecture.md
├── 05-flow-diagrams.md
├── 06-animations.md
├── 07-roadmap.md
└── 08-qa-report.md
```

## Execution Order

### Sequential Phase
```
Interviewer → UI Sketcher → UX Spec Writer
```

### Parallel Phase
```
Client Tech Architect + Mermaid Designer + Interactive Designer
```

### Final Phase
```
Planner → Browser QA
```

## Agent Delegation

```markdown
TASK: [Goal]
EXPECTED OUTCOME: [Deliverable]
REQUIRED AGENT: [Agent name]
CONTEXT: [File paths, constraints]

MUST DO:
- [Requirement 1]
- [Requirement 2]

MUST NOT DO:
- [Forbidden action]
```

## When to Use

**Use when:**
- Customer request is vague or unclear
- Planning new web app from scratch
- Need comprehensive documentation before coding
- Building HTML + Tailwind + vanilla JS app
- Client-side only (no backend)

**Don't use when:**
- Requirements already crystal clear
- Building backend/server-side application
- Need framework-specific planning (React/Vue)
- Just need quick prototype

## Final Output

Comprehensive Markdown specification containing:
1. Requirements document
2. ASCII wireframes
3. UX specification with philosophy
4. Technical architecture
5. Flow diagrams (Mermaid)
6. Animation specifications
7. Development roadmap
8. QA report

## Reference Files

| Task | Reference |
|------|-----------|
| Overall workflow | `references/workflow.md` |
| Agent coordination | `references/shared-folder-spec.md` |
| Common tools | `references/common-agent-tools.md` |
| Detailed guide | `references/skill-detailed-guide.md` |
| Usage examples | `references/skill-usage-examples.md` |
