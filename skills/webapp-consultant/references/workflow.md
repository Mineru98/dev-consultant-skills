# Workflow Guide

Complete workflow for transforming vague requirements into detailed specifications.

## Phase 1: Discovery (Interviewer)

**Goal**: Extract clear requirements from vague input
**Output**: `.shared/01-requirements.md`

**Process**:
1. Start with open-ended questions
2. Dig deeper with follow-ups
3. Confirm understanding
4. Document all requirements

## Phase 2: Visualization (UI Sketcher)

**Goal**: Create ASCII wireframes from requirements
**Output**: `.shared/02-wireframes.md`

**Process**:
1. Identify main screens
2. Sketch layout structure
3. Add UI elements
4. Annotate with Tailwind hints

## Phase 3: Documentation (UX Spec Writer)

**Goal**: Create comprehensive spec with UX philosophy
**Output**: `.shared/03-ux-specification.md`

**Process**:
1. Transform requirements into user stories
2. Connect wireframes to UX principles (Norman/Nielsen)
3. Document acceptance criteria

## Phase 4: Technical Research (Client Tech Architect)

**Goal**: Define data architecture and storage patterns
**Output**: `.shared/04-tech-architecture.md`

**Process**:
1. Design localbase collections
2. Define repository pattern classes
3. Document CRUD operations

## Phase 5: Flow Design (Mermaid Designer)

**Goal**: Visualize user journeys
**Output**: `.shared/05-flow-diagrams.md`

**Process**:
1. Map user journeys
2. Create flowcharts
3. Document decision points

## Phase 6: Interaction Design (Interactive Designer)

**Goal**: Define animations and micro-interactions
**Output**: `.shared/06-animations.md`

**Process**:
1. Identify interaction points
2. Design hover/focus/active states
3. Write Tailwind animation code

## Phase 7: Planning (Planner)

**Goal**: Create development roadmap
**Output**: `.shared/07-roadmap.md`

**Process**:
1. Apply MoSCoW to features
2. Create WBS breakdown
3. Define phases and milestones

## Phase 8: Quality Assurance (Browser QA)

**Goal**: Verify implementation quality
**Output**: `.shared/08-qa-report.md`

**Process**:
1. Test all functional flows
2. Verify responsive layouts
3. Check accessibility
4. Document issues

## Parallel Execution

```
Sequential:
Interviewer → UI Sketcher → UX Spec Writer

Parallel (after UX Spec Writer):
├── Client Tech Architect
├── Mermaid Designer
└── Interactive Designer

Final:
Planner → Browser QA (needs running app)
```

## .shared Folder Structure

All outputs go to `.shared/` in target repository:

```
.shared/
├── 01-requirements.md      # Interviewer
├── 02-wireframes.md        # UI Sketcher
├── 03-ux-specification.md  # UX Spec Writer
├── 04-tech-architecture.md # Client Tech Architect
├── 05-flow-diagrams.md     # Mermaid Designer
├── 06-animations.md        # Interactive Designer
├── 07-roadmap.md           # Planner
└── 08-qa-report.md         # Browser QA
```

## Handoff Checklist

Before moving to next phase:

- [ ] Current phase output complete
- [ ] Output saved to `.shared/` folder
- [ ] Output reviewed for completeness
- [ ] Context documented for handoff

## Dependency Graph

```
01-requirements
       ↓
02-wireframes
       ↓
03-ux-specification
       ↓
    ┌──┼──┐
    ↓  ↓  ↓
   04 05 06
    └──┼──┘
       ↓
07-roadmap
       ↓
08-qa-report (needs running app)
```
