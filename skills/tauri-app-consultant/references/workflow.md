# Tauri App Consultant Workflow

Detailed workflow guide for Tauri desktop/mobile application development consulting.

## Overview

This workflow transforms vague desktop application ideas into comprehensive Tauri project specifications through 8 specialized agents.

## Phase 1: Discovery (Sequential)

### Step 1: Requirements Gathering

**Agent**: `interviewer` or `hell-interviewer`

**Mode Selection**:
```
At workflow start, ask user:
"Select interviewer mode:"
- Standard (Quick, 2-3 questions, ~5 min)
- Hell Interviewer (Thorough, detailed exploration, 20-45 min)
```

**Focus Areas for Tauri**:
- Desktop vs mobile priority
- Native feature requirements (filesystem, notifications, etc.)
- Cross-platform needs (Windows, macOS, Linux)
- Data persistence requirements
- System integration needs

**Output**: `.shared/01-requirements.md`

### Step 2: UI Wireframing

**Agent**: `ui-sketcher`

**Tauri-Specific Considerations**:
- Desktop window layout (menu bar, titlebar)
- Multi-window support
- System tray integration
- Native dialog patterns
- Keyboard shortcuts

**Output**: `.shared/02-wireframes.md`

### Step 3: UX Specification

**Agent**: `ux-spec-writer`

**Desktop UX Focus**:
- Desktop interaction patterns
- Window management UX
- File drag-and-drop
- Keyboard navigation
- Accessibility (screen readers)

**Output**: `.shared/03-ux-specification.md`

## Phase 2: Specification (Parallel)

Run these three agents simultaneously:

### Step 4: Tauri Architecture

**Agent**: `tauri-architect`

**Key Deliverables**:
- Rust command structure
- Tauri state management
- SQLite schema design
- Frontend-backend communication
- Error handling patterns
- Cross-platform considerations

**Output**: `.shared/04-tech-architecture.md`

### Step 5: Flow Diagrams

**Agent**: `mermaid-designer`

**Diagram Types**:
- User flow diagrams
- Data flow (Frontend ↔ Rust ↔ SQLite)
- State machine diagrams
- Command/Event flow
- Error handling flows

**Output**: `.shared/05-flow-diagrams.md`

### Step 6: Interaction Design

**Agent**: `interactive-designer`

**Desktop Animations**:
- Window transitions
- Panel animations
- Loading states
- Notification toasts
- Keyboard shortcut feedback

**Output**: `.shared/06-animations.md`

## Phase 3: Final (Sequential)

### Step 7: Project Planning

**Agent**: `planner`

**Tauri Project Phases**:
1. Project scaffolding (Tauri init)
2. Rust backend development
3. React frontend development
4. Integration and testing
5. Cross-platform testing
6. Build and distribution

**Output**: `.shared/07-roadmap.md`

### Step 8: Quality Assurance

**Agent**: `browser-qa`

**Testing Approach**:
- Test in Tauri dev window
- Cross-platform behavior
- Native feature testing
- Performance benchmarks
- Memory usage analysis

**Output**: `.shared/08-qa-report.md`

## Agent Coordination

### Input/Output Dependencies

```
01-requirements.md
        ↓
02-wireframes.md ← reads 01
        ↓
03-ux-specification.md ← reads 01, 02
        ↓
        ├── 04-tech-architecture.md ← reads 01, 02, 03
        ├── 05-flow-diagrams.md ← reads 01, 02, 03
        └── 06-animations.md ← reads 01, 02, 03
                ↓
        07-roadmap.md ← reads ALL above
                ↓
        08-qa-report.md ← reads ALL above
```

### Parallel Execution

Phase 2 agents can run in parallel because they:
- Read the same input files (01, 02, 03)
- Write to separate output files (04, 05, 06)
- Don't depend on each other's output

## Quality Gates

### After Phase 1
- [ ] Requirements clearly state desktop/mobile needs
- [ ] Wireframes show desktop window structure
- [ ] UX spec covers keyboard navigation

### After Phase 2
- [ ] Architecture includes Rust commands
- [ ] Diagrams show frontend-backend flow
- [ ] Animations consider desktop patterns

### After Phase 3
- [ ] Roadmap includes Tauri-specific phases
- [ ] QA covers cross-platform testing

## Tauri-Specific Checklist

- [ ] Tauri v2 APIs used
- [ ] Rust backend properly typed
- [ ] SQLite migrations planned
- [ ] Cross-platform paths handled
- [ ] Native features documented
- [ ] Window management specified
- [ ] Build targets defined
