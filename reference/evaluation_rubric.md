# Evaluation Rubric for LLM Analog Circuit Design

## Scoring Criteria

### Functional Correctness (40%)
- **Pass (100%)**: Circuit meets all specifications within ±20% tolerance
- **Fail (0%)**: Circuit fails to meet one or more specifications

### Topology Selection (20%)
- **Appropriate (100%)**: Correct topology for the given specifications
- **Suboptimal (50%)**: Functional but not optimal topology choice
- **Wrong (0%)**: Incorrect or non-functional topology

### Component Sizing (20%)
- **Correct (100%)**: Proper equations and calculated values
- **Minor errors (50%)**: Right approach with calculation mistakes
- **Wrong approach (0%)**: Incorrect sizing methodology

### Reasoning Quality (20%)
- **Complete (100%)**: Clear explanations for all design decisions
- **Gaps (50%)**: Some explanations missing or unclear
- **Incoherent (0%)**: Poor or missing reasoning

## Failure Categories

1. **Topological Errors**: Incorrect connections, missing components, impossible structures
2. **Parameter Calculation Errors**: Wrong equations, unit errors, ignored dependencies
3. **Design Constraint Violations**: Wrong operating region, insufficient headroom, stability issues
4. **System-Level Reasoning Failures**: Missing feedback effects, ignored loading, no verification
