<div align="center">
  <h1 align="center">NL2Logic: AST-Guided Translation of Natural Language into First-Order Logic with Large Language Models</h1>
</div>

<p align="center">
  <a href='https://arxiv.org/abs/2602.13237'><img src='https://img.shields.io/badge/Paper-Arxiv-crimson'></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-lavender.svg" alt="License: MIT"></a>
</p>



## Overview

NL2Logic is a framework for translating natural language into
*semantically faithful* and *syntactically executable* first-order logic (FOL)
using an AST-guided approach with large language models.

NL2Logic avoids single-step, free-form text generation.
Instead, it:
1. Iteratively parses natural language into a **First-Order Logic Abstract Syntax Tree (FOL AST)**.
2. Deterministically compiles the AST into **solver-ready logic** using a multi-pass generation process.
3. Produces executable logic compatible with various logic engine such as **Z3**.


Tested models:
- [Gemma-2-2b](https://huggingface.co/google/gemma-2-2b)
- [Gemma-2-9b](https://huggingface.co/google/gemma-2-9b)
- [Gemma-2-27b](https://huggingface.co/google/gemma-3-27b-it)
- [Llama3.1-8b](https://huggingface.co/meta-llama/Llama-3.1-8B)
- [Llama3.2-1b](https://huggingface.co/meta-llama/Llama-3.2-1B)
- [Llama3.2-3b](https://huggingface.co/meta-llama/Llama-3.2-3B)
- [Qwen2.5-0.5b](https://huggingface.co/Qwen/Qwen2.5-0.5B)
- [Qwen2.5-1.5b](https://huggingface.co/Qwen/Qwen2.5-1.5B)
- [Qwen2.5-3b](https://huggingface.co/Qwen/Qwen2.5-3B)
- [Qwen2.5-7b](https://huggingface.co/Qwen/Qwen2.5-7B)
- [Qwen2.5-14b](https://huggingface.co/Qwen/Qwen2.5-14B-Instruct)
- [Mistral-22b](https://huggingface.co/Vezora/Mistral-22B-v0.2)
- [Ministral-8b](https://huggingface.co/mistralai/Ministral-8B-Instruct-2410)


## Installation

Clone the repository and install dependencies:

```bash
git clone https://github.com/peng-gao-lab/llm-verification
cd llm-verification
pip install -r requirements.txt
```
Python ≥ 3.9 is recommended.

## Configuration

NL2Logic uses OpenAI models when the `openai` backend is selected.
To enable this, you need to set your OpenAI API key as an environment variable.
On top of that, NL2Logic also support [vLLM](https://github.com/vllm-project/vllm) and [ollama](https://ollama.com) backend.

### OpenAI API Key

Create a `.env` file in the project root or export the variable directly:

```bash
export OPENAI_API_KEY="your-openai-api-key"
```
Alternatively, you can create a .env file:
```
OPENAI_API_KEY=your-openai-api-key
```

## Public API

NL2Logic exposes a single user-facing class:
```
Pipeline
```

## Basic Usage

```
from pipeline import *

p = Pipeline(
    llm="vllm",              # or: openai, ollama
    model="google/gemma-2-9b-it",
    logging=True
)

text = "No one who does not want to be addicted to caffeine is unaware that caffeine is a drug."
ast = p.rephrase_and_parse(text)
```

The ```Pipeline``` handles:
- optional rephrasing into quantifier-explicit form
- recursive AST construction
- clause-level parsing with grammar-guided decisions

The `llm` parameter in the `Pipeline` class only specifies **which LLM service to use**. It does not start or run the model.
You must start the LLM server separately before running the pipeline.

Output Given by the Program:
```
Rephrased 'No one who doesn't want to be addicted to caffeine is unaware that caffeine is a drug.' to 'For every person x, if x does not want to be addicted to caffeine, then x is not unaware that caffeine is a drug'
└────Parsing 'For every person x, if x does not want to be addicted to caffeine, then x is not unaware that caffeine is a drug'
     Answer: B
     Quantified parser. Quantifier: ForAll, Variable: x
     └────Parsing 'if x does not want to be addicted to caffeine, then x is not unaware that caffeine is a drug'
          Answer: C
          Binary operator parser. Operator: If
          ├────Parsing 'x does not want to be addicted to caffeine'
          │    Answer: D
          │    Unary operator parser. Operator: Not
          │    └────Parsing 'x wants to be addicted to caffeine'
          │         Answer: A
          │         Transitive parser. Verb: want, Subject: x, Object: to be addicted to caffeine
          └────Parsing 'x is not unaware that caffeine is a drug'
               Answer: A
               Adjective parser. Adjective: unaware, Object: x
...
```

## Generating Solver Code

The returned AST can be converted into executable Z3 code:
```
z3_code = ast.convert_to_z3()
print(z3_code)
```
Executing the generated code will invoke the solver and print the satisfiability result.

---

## Datasets Used

Experiments in the paper use the following benchmarks:
- [LogicNLI](https://huggingface.co/datasets/tasksource/LogicNLI)
- [FOLIO](https://huggingface.co/datasets/yale-nlp/FOLIO)
- [ProofWriter](https://allenai.org/data/proofwriter)

## Citation

If you use NL2Logic in your research, please cite our paper:

```bibtex
@inproceedings{nl2logic,
    title = "NL2Logic: AST-Guided Translation of Natural Language into First-Order Logic with Large Language Models",
    author = "Putra, Rizky Ramadhana  and
      Pasha Basuki, Raihan Sultan  and
      Cheng, Yutong  and
      Gao, Peng",
    booktitle = "Findings of the Association for Computational Linguistics: EACL 2026",
    month = mar,
    year = "2026",
    address = "Rabat, Morocco",
    publisher = "Association for Computational Linguistics"
}
```