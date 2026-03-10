# Chemist-Guided Human-AI Workflow for Covalent Organic Framework Synthesis
![Framework](https://github.com/user-attachments/assets/91b57efc-c213-4939-9844-89c57c172689)

This repository contains the code accompanying the JACS paper **“Chemist-Guided Human-AI Workflow for Covalent Organic Framework Synthesis.”**

**Paper**  
- ACS article page: https://pubs.acs.org/doi/10.1021/jacs.5c20068
- DOI: [10.1021/jacs.5c20068](https://doi.org/10.1021/jacs.5c20068)

**Citation**  
Lihan Chen, Zhen Lu, Lin Chen, Linxi Hou, and Dong Zhang. *Chemist-Guided Human-AI Workflow for Covalent Organic Framework Synthesis*. **Journal of the American Chemical Society** 2026, **148** (7), 7440-7449. DOI: [10.1021/jacs.5c20068](https://doi.org/10.1021/jacs.5c20068)

## Overview

Covalent organic framework (COF) synthesis is often guided by literature-informed intuition and iterative trial-and-error. This repository provides a notebook-based implementation of a **chemist-guided, closed-loop human-AI workflow** for COF synthesis planning and optimization.

Given an unseen linker combination, the workflow:

1. extracts ligand names from a natural-language query;
2. retrieves analogous COF synthesis records from a structured knowledge base;
3. builds a **range-type synthesis prior** over solvent system, catalyst, temperature, time, and stoichiometry;
4. proposes a **10-experiment exploration plan** for Round 1;
5. diagnoses outcomes from experimental observations and PXRD images;
6. updates the synthesis prior and recommends the **next round of experiments**.

The current public release is centered on a research notebook implementation and is intended to document the workflow used in the associated study.

## Main Components

### 1. Agent 1: Query Parsing and Retrieval
- Extracts 2- or 3-ligand queries from user input.
- Retrieves relevant COF synthesis cases using embedding-based nearest-neighbor search.
- Supports adaptive Top-K expansion to improve evidence diversity.

### 2. Agent 2: Prior Generation
- Samples literature evidence using multiple context-assembly strategies.
- Generates multiple prior cards through repeated LLM calls.
- Selects and merges the most stable synthesis prior over:
  - solvent candidates,
  - catalyst candidates,
  - temperature range,
  - reaction time range,
  - stoichiometric ratio.

### 3. Agent 3: Experiment Planning
- Converts the prior into a structured **10-experiment initial exploration matrix**.
- Emphasizes solvent/catalyst screening in early rounds.
- Produces standardized experiment records for laboratory execution.

### 4. Feedback and Iterative Optimization
- Accepts execution feedback as structured JSON.
- Interprets observations together with PXRD images.
- Classifies results using a failure taxonomy:
  - Class A: Reaction stalled
  - Class B: Solubility trap
  - Class C: Amorphous kinetic trap
  - Class D: Near-crystalline / poorly crystalline
- Updates the prior and proposes the next experimental round.


## Requirements

Recommended environment:

- Python 3.10+
- NumPy
- Pydantic
- OpenAI Python SDK
- LlamaIndex core
- Google GenAI integration for LlamaIndex
- FAISS 


## API Keys

The workflow uses external model services. Before running the notebook, configure the following environment variables:

```bash
export DASHSCOPE_API_KEY="your_dashscope_key"
export GOOGLE_API_KEY="your_google_key"
```

Do **not** commit real API keys to GitHub.


### 1. Initialize the workflow

```python
from llama_index.core.workflow import Context

settings = Settings(
    rag_db_path="rag_database_v2",
    records_file="rag_database/cof_data.json",
)

wf = COFIntegratedWorkflow(settings=settings, timeout=3600, verbose=False)
ctx = Context(wf)
```

### 2. Reset the session

```python
res = await wf.run(ctx=ctx, user_msg="reset")
pretty_print(res)
```

### 3. Generate a Round-1 plan for a new ligand pair

```python
query = (
    "1,3,6,8-Tetrakis(4-aminophenyl)pyrene (TAPPy) + "
    "2',3',5',6'-Tetrafluorobiphenyl-4,4'-dicarbaldehyde"
)

res = await wf.run(ctx=ctx, user_msg=query)
pretty_print(res)
```

### 4. Submit experimental feedback

```python
import json

feedback = {
    "executed_experiments": [
        {
            "exp_id": "R1-E01",
            "executed": True,
            "observations": "Powdery solid",
            "pxrd_image_path": "pxrd/pxrd1.png",
            "pxrd_image_mime": "image/png",
        }
    ]
}

res2 = await wf.run(ctx=ctx, user_msg=json.dumps(feedback, ensure_ascii=False))
pretty_print(res2)
```

## Expected Input Format

### Natural-language synthesis query

```text
ligand_A + ligand_B
```

Example:

```text
1,3,6,8-Tetrakis(4-aminophenyl)pyrene (TAPPy) + 2',3',5',6'-Tetrafluorobiphenyl-4,4'-dicarbaldehyde
```

### Feedback JSON

```json
{
  "executed_experiments": [
    {
      "exp_id": "R1-E01",
      "executed": true,
      "observations": "Powdery solid",
      "pxrd_image_path": "pxrd/pxrd1.png",
      "pxrd_image_mime": "image/png"
    }
  ]
}
```

## Expected Output

The workflow returns structured JSON objects that may include:

- the detected ligands,
- retrieval metadata,
- the selected synthesis prior,
- a 10-experiment plan for the current round,
- classifications for executed experiments,
- an updated prior and a next-round plan.


## Citation

If you use this repository in academic work, please cite the associated paper:

```bibtex
@article{Chen2026ChemistGuidedCOF,
  author  = {Chen, Lihan and Lu, Zhen and Chen, Lin and Hou, Linxi and Zhang, Dong},
  title   = {Chemist-Guided Human-AI Workflow for Covalent Organic Framework Synthesis},
  journal = {Journal of the American Chemical Society},
  year    = {2026},
  volume  = {148},
  number  = {7},
  pages   = {7440--7449},
  doi     = {10.1021/jacs.5c20068}
}
```


## Contact

For questions regarding the workflow, reproducibility, or data availability, please open an issue or contact the corresponding authors of the associated paper.
