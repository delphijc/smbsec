# Design Thinking Workflow
**Type:** Interactive Document Workflow
**Module:** Creative Intelligence System (CIS)
## Purpose
Guides human-centered design processes through the complete design thinking methodology: Empathize, Define, Ideate, Prototype, and Test. Creates solutions deeply rooted in user needs by combining empathy-driven research with systematic creative problem-solving.
## Distinctive Features
- **Phase-Based Structure**: Full five-phase design thinking journey from empathy to testing
- **Method Library**: Curated collection of design methods in `design-methods.csv` organized by phase
- **Context Integration**: Accepts design briefs or user research via data attribute
- **Facilitation Principles**: Guides divergent thinking before convergent action, emphasizes rapid prototyping over discussion
## Usage
```bash
# Basic invocation
workflow design-thinking
# With project context
workflow design-thinking --data {prompt input}
## Inputs
- **design_challenge**: Problem or opportunity being explored
- **users_stakeholders**: Primary users and affected parties
- **constraints**: Time, budget, technology limitations
- **recommended_inputs**: Existing research or context documents
## Outputs
**File:** `design-thinking-{date}.md`
**Structure:**
- Design challenge statement and point-of-view
- User insights and empathy mapping
- "How Might We" questions and problem framing
- Generated solution concepts
- Prototype designs and test plans
- Validated learning and iteration roadmap
## Workflow Components
- `design-thinking-workflow.md` - Configuration with design_methods CSV reference
- `design-thinking-instructions.md` - 7-step facilitation guide through design thinking phases
- `design-thinking-template.md` - Structured output format
- `design-methods.csv` - Phase-specific design techniques library
