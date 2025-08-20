# Analog Circuit LLM Benchmark

**Evaluating LLM Reasoning Capabilities in Analog Circuit Design Through In-Context Learning**

## Overview

This repository contains a systematic benchmark for evaluating the fundamental limitations of transformer-based Large Language Models (LLMs) in analog circuit design tasks using only in-context learning.

## Research Question

*"What are the fundamental limitations of transformer-based reasoning for analog circuit design tasks using only in-context learning?"*

## Methodology

### Circuit Progression (10 Circuits)
1. Single transistor switch
2. Common source amplifier  
3. Source follower buffer
4. Basic current mirror
5. Differential pair
6. Differential pair with current mirror load
7. Cascode current mirror
8. Single-stage OTA
9. Two-stage OTA
10. CMOS comparator

### Experimental Protocol
- **Fixed prompt template** (no iteration allowed)
- **GF 180nm CMOS process**
- **NGSPICE validation**
- **3-5 attempts per circuit per model**
- **±20% tolerance for success**

## Repository Structure

```
├── prompts/                    # Fixed prompt template and circuit specs
├── model_outputs/             # Raw LLM outputs and netlists
│   ├── claude_opus_4_1/
│   └── gpt4/
├── simulation_results/        # NGSPICE simulation outputs
├── analysis/                  # Failure taxonomy and scoring
└── reference/                 # SPICE models and evaluation rubric
```

## Usage

1. **Setup**: Ensure NGSPICE and GF 180nm models are available
2. **Testing**: Use fixed prompt template with each circuit specification  
3. **Validation**: Run extracted netlists through NGSPICE
4. **Analysis**: Record results using provided taxonomy

## Models Tested
- Claude Opus 4.1
- GPT-4 (latest available)

## Success Criteria
Circuit must meet all specifications within ±20% tolerance with functionally correct NGSPICE netlist.

## Conference Target
ETECOM Conference - Track 1: Artificial Intelligence (AI), Machine Learning (ML), and Business Analytics

## Citation
[To be added upon publication]

## License
[To be specified]
