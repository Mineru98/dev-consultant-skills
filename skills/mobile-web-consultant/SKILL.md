---
name: mobile-web-consultant
description: 모바일 친화적 PWA/반응형 웹앱 개발 컨설턴트
---

# Mobile Web Development Consultant

Transform mobile app ideas into comprehensive PWA/responsive web specifications through orchestrated multi-agent collaboration. Specialized for mobile-first progressive web applications.

## Tech Stack

- **Structure**: HTML5
- **Styling**: Tailwind CSS
- **Logic**: Vanilla JavaScript (ES6+)
- **PWA**: Service Worker + Web App Manifest
- **Storage**: IndexedDB (via localbase)
- **Offline**: Cache API + Background Sync

## When to Use

**Use when:**
- Building mobile-first responsive web applications
- Need offline capability (PWA)
- Want app-like experience without app store
- Touch-optimized interface required
- Single codebase for all devices

**Don't use when:**
- Desktop-first application needed
- Native device APIs required (camera, sensors)
- Complex desktop interactions needed
- Offline not a priority

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
│ Mobile UI Sketcher│ → Mobile-first wireframes
│                   │ → .shared/02-wireframes.md
└───────────────────┘
            ↓
┌───────────────────┐
│  UX Spec Writer   │ → Mobile UX specifications
│                   │ → .shared/03-ux-specification.md
└───────────────────┘
            ↓
┌─────────────────────────────────────────────────────────────┐
│                   Phase 2: Specification                     │
│                      (Parallel)                              │
└─────────────────────────────────────────────────────────────┘
            ↓
┌───────────────────┬───────────────────┬─────────────────────┐
│ Client Tech Arch  │ Mermaid Designer  │ Mobile Interaction  │
│ (PWA + Storage)   │ (Flow diagrams)   │     Designer        │
│ 04-tech-arch.md   │ 05-flow-diag.md   │ 06-animations.md    │
└───────────────────┴───────────────────┴─────────────────────┘
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
│   Browser QA      │ → Mobile device testing
│                   │ → .shared/08-qa-report.md
└───────────────────┘
```

## 8 Agent Summary

| # | Agent | Output | Purpose |
|---|-------|--------|---------|
| 1 | Interviewer (or Hell) | 01-requirements.md | Extract requirements |
| 2 | **Mobile UI Sketcher** | 02-wireframes.md | Mobile-first wireframes |
| 3 | UX Spec Writer | 03-ux-specification.md | Mobile UX documentation |
| 4 | Client Tech Architect | 04-tech-architecture.md | PWA + Storage design |
| 5 | Mermaid Designer | 05-flow-diagrams.md | Flow visualizations |
| 6 | **Mobile Interaction Designer** | 06-animations.md | Touch interactions |
| 7 | Planner | 07-roadmap.md | Development roadmap |
| 8 | Browser QA | 08-qa-report.md | Mobile testing |

## .shared Folder Structure

```
.shared/
├── 01-requirements.md         # What to build
├── 02-wireframes.md           # Mobile-first layouts
├── 03-ux-specification.md     # Mobile UX patterns
├── 04-tech-architecture.md    # PWA + Storage architecture
├── 05-flow-diagrams.md        # Mermaid diagrams
├── 06-animations.md           # Touch interaction specs
├── 07-roadmap.md              # Development plan
└── 08-qa-report.md            # Mobile test results
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
- Prioritize mobile-first design

MUST NOT DO:
- Skip reading previous phase outputs
- Design desktop-first layouts
- Ignore touch interaction requirements
- Use non-standard mobile patterns
```

## Mobile-Specific Considerations

### Touch Patterns
- Thumb zone optimization
- 44px minimum touch targets
- Swipe gestures (delete, navigate)
- Pull-to-refresh
- Bottom navigation

### PWA Requirements
- Service Worker registration
- Web App Manifest
- Offline fallback pages
- Add to Home Screen prompt
- Push notifications (optional)

### Performance
- First Contentful Paint < 1.8s
- Largest Contentful Paint < 2.5s
- Cumulative Layout Shift < 0.1
- Time to Interactive < 3.8s

### Responsive Breakpoints
- Mobile: 320px - 480px
- Tablet: 481px - 768px
- Desktop: 769px+

## Final Outputs

8 comprehensive markdown documents in `.shared/`:
1. Requirements with mobile use cases
2. ASCII wireframes for mobile-first layout
3. UX specification with touch patterns
4. PWA + IndexedDB technical architecture
5. Mermaid flow diagrams
6. Touch interaction and gesture specs
7. Prioritized development roadmap
8. Mobile QA test report

## Reference Files

- `references/workflow.md` - Detailed workflow guide
- `references/mobile-ux-patterns.md` - Mobile patterns
- `references/pwa-guide.md` - PWA implementation
- `references/touch-interactions.md` - Touch guidelines
- `references/shared-folder-spec.md` - Output standards
- `references/common-agent-tools.md` - Tool usage
