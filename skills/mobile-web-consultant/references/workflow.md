# Mobile Web Consultant Workflow

Detailed workflow guide for mobile-first PWA development consulting.

## Overview

This workflow transforms mobile app ideas into comprehensive PWA specifications through 8 specialized agents with mobile-first focus.

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

**Focus Areas for Mobile**:
- Primary device usage (phone vs tablet)
- Offline requirements
- Touch interaction needs
- Performance expectations (3G/4G)
- PWA installation requirements

**Output**: `.shared/01-requirements.md`

### Step 2: Mobile UI Wireframing

**Agent**: `mobile-ui-sketcher`

**Mobile-First Considerations**:
- Thumb zone optimization
- Bottom navigation pattern
- Touch target sizes (44px+)
- Safe area handling
- Responsive breakpoints

**Output**: `.shared/02-wireframes.md`

### Step 3: UX Specification

**Agent**: `ux-spec-writer`

**Mobile UX Focus**:
- Touch gesture patterns
- Pull-to-refresh
- Swipe actions
- Loading states
- Offline UX

**Output**: `.shared/03-ux-specification.md`

## Phase 2: Specification (Parallel)

Run these three agents simultaneously:

### Step 4: Technical Architecture

**Agent**: `client-tech-architect`

**PWA Requirements**:
- Service Worker strategy
- Cache strategies
- IndexedDB schema
- Background sync
- Push notifications (if needed)

**Output**: `.shared/04-tech-architecture.md`

### Step 5: Flow Diagrams

**Agent**: `mermaid-designer`

**Diagram Types**:
- User journey (mobile flow)
- Data sync flow
- Offline/online states
- Service Worker lifecycle
- Cache strategies

**Output**: `.shared/05-flow-diagrams.md`

### Step 6: Mobile Interactions

**Agent**: `mobile-interaction-designer`

**Touch Interactions**:
- Tap feedback (ripple)
- Swipe gestures
- Pull-to-refresh
- Long press actions
- Haptic patterns

**Output**: `.shared/06-animations.md`

## Phase 3: Final (Sequential)

### Step 7: Project Planning

**Agent**: `planner`

**Mobile Project Phases**:
1. Core mobile UI
2. Touch interactions
3. Offline capability
4. PWA manifest & icons
5. Performance optimization
6. Cross-device testing

**Output**: `.shared/07-roadmap.md`

### Step 8: Quality Assurance

**Agent**: `browser-qa`

**Mobile Testing**:
- Device emulation (Chrome DevTools)
- Touch interaction testing
- Offline mode testing
- Performance audit (Lighthouse)
- Responsive breakpoints

**Output**: `.shared/08-qa-report.md`

## Agent Coordination

### Input/Output Dependencies

```
01-requirements.md
        ↓
02-wireframes.md ← reads 01 (mobile-first)
        ↓
03-ux-specification.md ← reads 01, 02 (touch patterns)
        ↓
        ├── 04-tech-architecture.md ← (PWA + Storage)
        ├── 05-flow-diagrams.md ← (mobile flows)
        └── 06-animations.md ← (touch interactions)
                ↓
        07-roadmap.md ← reads ALL above
                ↓
        08-qa-report.md ← reads ALL (mobile testing)
```

## Quality Gates

### After Phase 1
- [ ] Mobile use cases clearly defined
- [ ] Wireframes show mobile layout
- [ ] Touch gestures documented

### After Phase 2
- [ ] PWA strategy defined
- [ ] Service Worker planned
- [ ] Touch interactions specified

### After Phase 3
- [ ] Mobile-first phases prioritized
- [ ] Cross-device testing planned

## Mobile-Specific Checklist

- [ ] Touch targets >= 44px
- [ ] Thumb zone optimized
- [ ] Offline mode supported
- [ ] Service Worker registered
- [ ] Web App Manifest complete
- [ ] Icons all sizes provided
- [ ] Splash screen configured
- [ ] Performance metrics met
- [ ] Responsive to all breakpoints
