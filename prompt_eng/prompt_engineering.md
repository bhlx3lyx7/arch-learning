# Prompt engineering

Prompt engineering skills help to better understand the capabilities and limitations of large language models (LLMs).

## Introduction

### LLM Settings
- Temperature: the lower the more deterministic the result is
- Top P: nucleus sampling chooses the number of results with most probability
- Max Length: prevent long or irrelevant responses and control costs
- Stop Sequence: a string that stops the model from generating tokens, another way to control length of response
- Frequency Penalty: applies a penalty on the next token proportional to how many times that token already appeared in the resposne
- Presense Penalty: also applies a penalty on repeated tokens but, unlike the frequency penalty, the penalty is the same for all repeated tokens, no matter how many times it has appeared

*temperature and top_p are recommended not to alter both*

*frequency and presense penalty are recommended not to alter both*

### Prompt Elements
- Instruction
- Context
- Input data
- Output indicator

### General tips for designing prompts
- start simple: iterate the prompt is vital
- instruction: instruct the model what you want to achieve, such as "write", "classify", "summarize", "translate", "order", etc.
- specificity: the more descriptive and detailed the prompt is, the better the results; length of prompts should also be considered
- avoid impreciseness: it's often better ot be specific and direct
- To do or not to do: avoid saying what not to do, but say what to do, this encourages more specificity and focuses on the detailes that lead to good responses from the model

### Examples
- Text Summarization
- Information Extraction
- Question Answering
- Text Classification
- Conversation
- Code Generation
- Reasoning

## Techniques

### Zero-shot prompting
instruction without sample

### Few-shot prompting
few samples helps in-context learning during prompting

few-shot works better than zero-shot, but still can not handle complex reasoning problems, such as math analysis

### Chain of thought (CoT) prompting
give the steps to solve the problem, it can also combine with few-shot prompting, with some examples

in zero-shot prompting, it can be achieved by this prompt: Let's think step by step.

### Self-consistency
replace the naive greedy decoding used in chain-of-thought prompting

### Generated knowledge prompting
generate knowledge first, then answer the question

### Tree of thoughts (ToT)
a framework that generalizes over chain-of-thought prompting, encourages exploration over thoughts that serve as intermediate steps for general problem solving with language models

### Retrieval Augmented Generation (RAG)
RAG combines an information retrieval component with a text generator model. RAG can be fine-tuned and its internal knowledge can be modified in an efficient manner and without needing retraining of the entire model.

### Automatic Reasoning and Tool-use (ART)
a new framework that uses a frozen LLM to automatically generate intermediate reasoning steps as a program.

### Automatic Prompt Engineer (APE)
a framework for automatic instruction generation and selection.
1. LLMs as inference models
2. LLMs as scoring models
3. LLMs as resampling models

It discovers a better zero-shot CoT prompt "Let's work this out in a step by step way to be sure we have the right answer."

### Active-Prompt
1. The first step is to query the LLM with or without a few CoT examples. 
2. k possible answers are generated for a set of training questions. 
3. An uncertainty metric is calculated based on the k answers (disagreement used). 
4. The most uncertain questions are selected for annotation by humans. 
5. The new annotated exemplars are then used to infer each question.

### Directional Stimulus Prompting
prompt with directional hint, such as reference doc

### ReAct Prompting
a framework named ReAct to generate both reasoning traces and task-specific actions in an interleaved manner.

The ReAct framework can allow LLMs to interact with external tools to retrieve additional information that leads to more reliable and factual responses.

ReAct combined with chain-of-thought (CoT) that allows use of both internal knowledge and external information obtained during reasoning.

### Multimodal CoT Prompting
Multimodal CoT incorporates text and vision into a two-stage framework.
The first step involves rationale generation based on multimodal information. This is followed by the second phase, answer inference, which leverages the informative generated rationales.

### Graph Prompting
a new prompting framework for graphs to improve performance on downstream tasks.

## Applications

### PAL (Program-Aided Language Models)
instead of using free-form text to obtain solution it offloads the solution step to a programmatic runtime such as a Python interpreter.

### Generating Data
Using effective prompt strategies can steer the model to produce better, consistent, and more factual responses

### Generating Synthetic Dataset for RAG
- Synthetic Data for RAG Setup
- Domain-Specific Dataset Generation

### Tackling Generated Datasets Diversity

### Generating Code

### Graduate Job Classification Case Study

### Prompt Function
By encapsulating prompts into functions, you can create a series of functions to establish a workflow.

## Models
- Flan PaLM
- ChatGPT, trained using Reinforcement Learning from Human Feedback (RLHF).
- LLaMA
- GPT-4
- Mistral 7B
- Gemini
- [LLM Collection](https://www.promptingguide.ai/models/collection)

## Risks & Misuses

### Adversarial Prompting
- Prompt Injection
- Prompt Leaking
- Jailbreaking

### Factuality
LLMs have a tendency to generate responses that sounds coherent and convincing but can sometimes be made up. Improving prompts can help improve the model to generate more accurate/factual responses and reduce the likelihood to generate inconsistent and made up responses.

### Biases

## Notebooks
[link](https://www.promptingguide.ai/notebooks)

## Reference
- learning doc: https://www.promptingguide.ai/