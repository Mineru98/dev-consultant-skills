# Chrome Extension Consultant Workflow

Detailed workflow guide for Chrome extension development consulting.

## Overview

This workflow transforms browser extension ideas into comprehensive Manifest V3 specifications through 8 specialized agents.

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

**Focus Areas for Extensions**:
- Extension type (popup, content, background)
- Target websites/domains
- Required browser APIs
- Permission requirements
- User data handling

**Output**: `.shared/01-requirements.md`

### Step 2: UI Wireframing

**Agent**: `ui-sketcher`

**Extension-Specific Considerations**:
- Popup size constraints (800x600 max)
- Options page layout
- Content script overlays
- Badge icons and text
- Context menu structure

**Output**: `.shared/02-wireframes.md`

### Step 3: UX Specification

**Agent**: `ux-spec-writer`

**Extension UX Focus**:
- Browser integration patterns
- Permission request flow
- First-run experience
- Settings organization
- Keyboard shortcuts

**Output**: `.shared/03-ux-specification.md`

## Phase 2: Specification (Parallel)

Run these three agents simultaneously:

### Step 4: Extension Architecture

**Agent**: `extension-architect`

**Key Deliverables**:
- Manifest V3 configuration
- Permission strategy
- Message passing flow
- Storage schema
- Security considerations

**Output**: `.shared/04-tech-architecture.md`

### Step 5: Flow Diagrams

**Agent**: `mermaid-designer`

**Diagram Types**:
- Message flow between contexts
- User interaction flow
- Data flow (storage, API)
- Permission request flow
- Extension lifecycle

**Output**: `.shared/05-flow-diagrams.md`

### Step 6: Interaction Design

**Agent**: `interactive-designer`

**Extension Animations**:
- Popup transitions
- Content script overlays
- Loading indicators
- Badge animations
- Notification toasts

**Output**: `.shared/06-animations.md`

## Phase 3: Final (Sequential)

### Step 7: Project Planning

**Agent**: `planner`

**Extension Project Phases**:
1. Manifest setup
2. Background service worker
3. Content scripts
4. Popup UI
5. Options page
6. Chrome Web Store preparation

**Output**: `.shared/07-roadmap.md`

### Step 8: Quality Assurance

**Agent**: `browser-qa`

**Extension Testing**:
- Install/uninstall flow
- Permission prompts
- Message passing
- Storage operations
- Cross-browser (Edge)

**Output**: `.shared/08-qa-report.md`

## Agent Coordination

### Input/Output Dependencies

```
01-requirements.md
        ↓
02-wireframes.md ← reads 01 (popup/options)
        ↓
03-ux-specification.md ← reads 01, 02
        ↓
        ├── 04-tech-architecture.md ← (MV3 design)
        ├── 05-flow-diagrams.md ← (message flow)
        └── 06-animations.md ← (UI transitions)
                ↓
        07-roadmap.md ← reads ALL above
                ↓
        08-qa-report.md ← reads ALL
```

## Quality Gates

### After Phase 1
- [ ] Extension type clearly defined
- [ ] Permission needs identified
- [ ] Target sites documented

### After Phase 2
- [ ] Manifest V3 compliant
- [ ] Message flow documented
- [ ] Storage schema defined

### After Phase 3
- [ ] Chrome Web Store ready
- [ ] Testing plan complete

## Extension-Specific Checklist

- [ ] Manifest V3 compliant
- [ ] Minimal permissions
- [ ] No remote code
- [ ] Content Security Policy set
- [ ] Service Worker patterns used
- [ ] Storage strategy clear
- [ ] Message passing defined
- [ ] Error handling complete
- [ ] Icons all sizes provided
- [ ] Privacy policy drafted
