# Analog Circuit LLM Benchmark

**Design Reasoning vs. Tool Competency: A Two-Tier In-Context-Only Evaluation of LLMs in Analog Circuit Design**

## Overview

This repository contains a systematic benchmark for evaluating Large Language Model (LLM) capabilities in analog circuit design through a novel two-tier framework that separates design reasoning from tool syntax competency.

## Research Innovation

**First study to systematically distinguish between:**
- **Tier 1**: Design reasoning capabilities (topology, analysis, component sizing)
- **Tier 2**: Tool syntax competency (EDA tool interface generation)

This separation addresses a fundamental question: *Can LLMs demonstrate sound analog circuit design reasoning independent of their ability to generate syntactically correct EDA tool input?*

## Research Questions

**Primary**: *"Can LLMs demonstrate sound analog circuit design reasoning independent of EDA tool syntax competency?"*

**Tier 1**: *"Do LLMs make appropriate topology choices, analysis decisions, and component sizing logic?"*

**Tier 2**: *"Can LLMs generate syntactically correct netlists for EDA tools without additional training?"*

## Methodology

### Two-Tier Evaluation Framework

**Tier 1: Design Reasoning Assessment**
- Topology correctness and reasoning quality
- Analysis methodology and reasoning
- Specification targeting accuracy
- *Evaluated independently of syntax correctness*

**Tier 2: Syntax Competency Assessment**  
- Raw netlist compilation success
- Post-cleanup compilation success
- Error type classification (Formatting/Syntax/Hallucination)

### Experimental Design
- **Approach**: In-context learning only (no fine-tuning, RAG, or external tools)
- **Models**: Claude Opus 4.1, ChatGPT-5
- **Circuits**: 10 circuits from simple switch to complex two-stage OTA
- **Evaluation**: Qualitative assessment using systematic criteria
- **Timeline**: 7-day execution (pilot study)

## Circuit Progression

1. **Single transistor switch** - Basic switching behavior
2. **Common source amplifier** - Single-stage amplification
3. **Source follower buffer** - Unity-gain buffering
4. **Current mirror** - Current replication
5. **Differential pair** - Differential amplification
6. **Differential pair with current mirror load** - Enhanced differential gain
7. **Cascode current mirror** - High output impedance
8. **Single-stage OTA** - Operational transconductance amplifier
9. **Two-stage OTA** - Complex multi-stage design
10. **CMOS comparator** - High-speed comparison

## Repository Structure

```
├── prompts/                    # Fixed prompt template and circuit specifications
├── model_outputs/             # Raw LLM outputs and extracted netlists
│   ├── claude_opus_4_1/       # Claude Opus 4.1 results
│   └── chatgpt5/              # ChatGPT-5 results
├── analysis/                  # Evaluation results and insights
├── reference/                 # SPICE models and evaluation framework
└── README.md                  # This file
```

## Usage

### Prerequisites
- NGSPICE simulator
- GF 180nm SPICE models (provided in `reference/`)

### Running Tests
1. Use the fixed prompt template in `prompts/fixed_template.txt`
2. Apply circuit specifications from `prompts/circuit_[1-10]_spec.txt`  
3. Record LLM outputs in `model_outputs/[model]/`
4. Evaluate using two-tier framework in `reference/evaluation_framework.md`

### Evaluation Criteria

**Tier 1 Scales:**
- Topology: Correct / Partially Correct / Incorrect
- Reasoning: Reasonable / Unreasonable  
- Analysis: Complete / Incomplete / Wrong
- Spec Targeting: All specs / Some specs / Misses specs

**Tier 2 Scales:**
- First Run: Yes / No
- Post-Cleanup: Yes / No
- Error Type: Formatting / Syntax / Hallucination

## Key Findings

*[Results will be added after study completion]*

## Conference Target

**ETECOM 2025 - Track 1: Artificial Intelligence (AI), Machine Learning (ML), and Business Analytics**

## Reproducibility

This benchmark is designed for full reproducibility:
- ✅ Open-source tools (NGSPICE, GF 180nm)
- ✅ Fixed methodology with no human iteration
- ✅ Complete documentation of evaluation criteria
- ✅ Public repository with all materials

## Future Work

- **Quantitative validation** with multiple attempts per circuit
- **Extended model coverage** (Gemini, Grok, specialized models)
- **Multi-process evaluation** across different PDKs
- **Syntax improvement studies** building on Tier 2 findings
- **Human-LLM collaboration** workflows

## Citation

*[Citation will be added upon publication]*

## License

*[License to be specified]*

---

**Contact**: *[Contact information to be added]*

**Last Updated**: *[Current date]*