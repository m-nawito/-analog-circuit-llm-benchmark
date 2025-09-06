# Two-Tier Evaluation Framework for LLM Analog Circuit Design

## Overview

This framework systematically evaluates LLM capabilities across two independent tiers:
- **Tier 1**: Design reasoning quality (independent of syntax)
- **Tier 2**: Tool syntax competency (NGSPICE compatibility)

## Tier 1: Design Reasoning Assessment

### 1. Topology Correctness
**Scale**: Correct / Partially Correct / Incorrect

- **Correct**: Appropriate circuit structure that can meet specifications
- **Partially Correct**: Right circuit family but with critical error (e.g., wrong biasing, missing component)
- **Incorrect**: Wrong topology that cannot achieve specifications

**Example**: For common source amplifier:
- Correct: NMOS with drain load and proper biasing
- Partially Correct: NMOS amplifier but inadequate biasing  
- Incorrect: Current mirror instead of amplifier

### 2. Topology Reasoning
**Scale**: Reasonable / Unreasonable

- **Reasonable**: Sound engineering justification for topology choice
- **Unreasonable**: Poor, missing, or incorrect reasoning

**Note**: Highlight exceptional insights or fundamental misunderstandings

### 3. Analysis Method
**Scale**: Complete / Incomplete / Wrong

- **Complete**: Testbench sufficient to verify all stated specifications
- **Incomplete**: Missing key measurements or bias conditions
- **Wrong**: Wrong analysis type (e.g., AC when DC needed, missing critical simulations)

**Example**: For amplifier gain specification:
- Complete: DC bias + AC analysis with appropriate frequency range
- Incomplete: Only DC analysis, missing AC gain measurement
- Wrong: Transient analysis only

### 4. Analysis Reasoning  
**Scale**: Reasonable / Unreasonable

- **Reasonable**: Sound justification for chosen analysis methodology
- **Unreasonable**: Poor understanding of what analysis verifies which specs

### 5. Specification Targeting
**Scale**: All specs / Some specs / Misses specs

- **All specs**: Design and analysis attempt to address every specification
- **Some specs**: Addresses major specs but ignores some requirements
- **Misses specs**: Fails to target key specifications in design/analysis

## Tier 2: Syntax Competency Assessment

### 1. First Run Success
**Scale**: Yes / No

- **Yes**: Raw LLM output compiles and runs in NGSPICE without modification
- **No**: Compilation errors prevent execution

### 2. Post-Cleanup Success
**Scale**: Yes / No

Apply **basic cleanup only**:
- Remove code fences (```spice, ```ngspice, etc.)
- Fix Unicode characters (µ → u, Ω → ohm)
- Add missing `.end` if needed
- Fix obvious whitespace issues

- **Yes**: Compiles after basic cleanup
- **No**: Still has compilation errors after cleanup

### 3. Error Classification
**Categories**: Formatting / Syntax / Hallucination

- **Formatting**: Code fences, Unicode issues, whitespace, missing `.end`
- **Syntax**: Typos in directive names, wrong unit suffixes, parameter formatting
- **Hallucination**: Non-existent devices/models, invented directives, impossible connections

## Documentation Template

```
Circuit X: [Circuit Name]
Model: [Claude Opus 4.1 / ChatGPT-5]

=== TIER 1: DESIGN REASONING ===
Topology Correctness: [Correct/Partially Correct/Incorrect]
Topology Reasoning: [Reasonable/Unreasonable]  
Analysis Method: [Complete/Incomplete/Wrong]
Analysis Reasoning: [Reasonable/Unreasonable]
Spec Targeting: [All specs/Some specs/Misses specs]

Notable Insights: [1-2 line highlight if exceptional/interesting]

=== TIER 2: SYNTAX COMPETENCY ===
First Run Success: [Yes/No]
Post-Cleanup Success: [Yes/No]  
Error Type: [Formatting/Syntax/Hallucination]

Error Details: [Brief description if notable]

=== SUMMARY ===
Tier 1 Overall: [Brief assessment of design reasoning quality]
Tier 2 Overall: [Brief assessment of syntax competency]
```

## Cross-Circuit Analysis Guidelines

### Pattern Recognition
- **Design reasoning degradation**: At what circuit complexity does reasoning break down?
- **Consistent error types**: Do models make similar mistakes across circuits?
- **Model differences**: How do Claude vs ChatGPT-5 compare on each tier?

### Notable Cases Documentation
- **Exceptional insights**: Unexpectedly sophisticated reasoning  
- **Fundamental errors**: Basic misunderstandings worth highlighting
- **Interesting failures**: Instructive failure modes

## Analysis File Structure

### tier1_design_reasoning_results.csv
```
Circuit_ID,Model,Topology_Correctness,Topology_Reasoning,Analysis_Method,Analysis_Reasoning,Spec_Targeting,Notable_Insights
1,Claude,Correct,Reasonable,Complete,Reasonable,All specs,"Good Vgs calculation"
1,ChatGPT5,Partially Correct,Reasonable,Incomplete,Unreasonable,Some specs,"Missing bias analysis"
...
```

### tier2_syntax_results.csv  
```
Circuit_ID,Model,First_Run,Post_Cleanup,Error_Type,Error_Details
1,Claude,No,Yes,Formatting,"Missing .end statement"
1,ChatGPT5,No,No,Hallucination,"Unknown device model"
...
```

### Cross-Model Comparison
- Success rates by tier and circuit complexity
- Error pattern analysis  
- Qualitative differences in reasoning approaches

## Quality Assurance

### Evaluation Consistency
- Document evaluation rationale for borderline cases
- Maintain consistent standards across circuits
- Note any evaluation difficulties or ambiguities

### Time Management
- Keep notes brief (1-2 lines maximum)
- Focus on patterns rather than exhaustive details
- Prioritize clear documentation over perfect analysis

This framework balances systematic evaluation with practical time constraints while maintaining research rigor.