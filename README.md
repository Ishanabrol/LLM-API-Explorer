# 🤖 LLM API Explorer

### Understanding How Large Language Models Work at the API Level

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)
![Gemini](https://img.shields.io/badge/Gemini-2.5%20Flash-4285F4?logo=google)
![License](https://img.shields.io/badge/License-MIT-yellow)

> Before building AI applications, you need to understand what's happening underneath.
> This project strips away every abstraction and talks directly to the Gemini API —
> controlling system prompts, roles, temperature, and token limits by hand,
> logging every call, and evaluating outputs systematically.

---

[![Open in nbviewer](https://img.shields.io/badge/View%20Notebook-nbviewer-orange?logo=jupyter)](https://nbviewer.org/github/Ishanabrol/LLM-API-Explorer/blob/main/Notebook/LLM_Explorer%281%29.ipynb)

---

## 💼 Why This Project Exists

Most people start with LangChain, ChatGPT wrappers, or no-code AI tools without
understanding what's actually happening when they send a message to an LLM.
This leads to one outcome — they can't debug when things go wrong.

**This project fixes that by answering three foundational questions:**

**1. What actually happens when you call an LLM API?**
Every AI application — no matter how complex — does exactly one thing at its core:
sends text in and gets text out. The roles, parameters, and structure around that
call determine everything about the output quality. Most people skip learning this
and struggle forever with prompt debugging.

**2. Why does the same question give completely different answers?**
System prompts, temperature, and role structure are the most powerful levers
available to any AI developer. Before tuning model weights or adding vector
databases, you need to understand how these primitives behave. This project
makes that behaviour transparent and observable.

**3. How do you know which prompt actually worked best?**
Gut feeling is not evaluation. This project builds a structured logging and
scoring system — every API call is recorded with its parameters, reply, and
token count so outputs can be compared systematically, not guessed at.

---

## 📌 Project Overview

A hands-on Python playground that calls the Google Gemini API and experiments
with every core LLM parameter — producing a logged, evaluated record of how
each setting affects model output.

**Core Experiments:**
- System prompt comparison — same question, three completely different personas
- Temperature sweep — 0.0 (deterministic) to 2.0 (creative) on identical inputs
- Role system — multi-turn conversation with full history management
- Token pre-flight — count tokens before sending to avoid context surprises

**Output Artifacts:**
- Structured CSV log of every API call with parameters and scores
- Evaluation summary table comparing experiments side by side

---

## 🛠️ Tech Stack

| Category | Library | Purpose |
|---|---|---|
| **API** | google-genai | Official Gemini SDK — send and receive LLM calls |
| **Token Counting** | tiktoken | Count tokens before sending — cost and context awareness |
| **Data** | pandas | Log management, comparison tables, evaluation output |
| **Security** | python-dotenv | Load API key from .env — never hardcode secrets |
| **Environment** | Jupyter Notebook | Interactive cell-by-cell execution via Anaconda |

---

## 🎯 Key Findings

| Experiment | Observation |
|---|---|
| **System Prompts** | Same question — "simple" persona used child analogies, "expert" gave technical definitions, "comedian" opened like a stand-up set. Identical model, identical temperature. |
| **Temperature** | For a factual one-liner, 0.0 and 2.0 both consumed more thinking tokens than 0.7 and 1.5. Extremes make thinking models work harder even on simple questions. |
| **Roles / History** | A 3-turn conversation cost 877 tokens — because the entire history is resent every call. Token cost grows linearly with conversation length. |
| **Evaluation** | One-Line Summary prompt scored 9/10 (auto-scored). Formal Expert scored 6/10 — reply was cut off because thinking tokens consumed the output budget. |

> **Core insight:** System prompts are the most powerful lever available.
> Before adjusting temperature or top-p, get the system prompt right.

---

---

## 📊 Experiment Log Sample

| Experiment | Temperature | Words | Score | Reason |
|---|---|---|---|---|
| Formal Expert | 0.3 | 17 | 6/10 | Too brief — thinking tokens consumed output budget |
| Friendly Tutor | 0.7 | 14 | 7/10 | No clear analogy detected in truncated reply |
| One-Line Summary | 0.0 | 30 | 9/10 | One sentence ✅ — accurate and complete |

---

## ⚙️ Methodology

### Phase Structure

| Phase | What Was Built |
|---|---|
| **Phase 2** | First API call — connect to Gemini, read response, inspect token usage |
| **Phase 3** | System prompt experiments, temperature sweep 0.0→2.0, multi-turn role system |
| **Phase 4** | Pre-flight token counter using tiktoken — estimate cost before sending |
| **Phase 5** | CSV logging system — every call saved with parameters, reply, and token count |
| **Phase 6** | Evaluation harness — auto-score outputs against defined criteria, summary table |

### Reliability Engineering
- **Retry logic** — automatic retry with 15s delay on 503 (server busy) errors
- **Rate limiting** — 10s sleep between calls to respect free tier limits (20 RPD)
- **Token budgeting** — `max_output_tokens` set above expected reply length to
  account for gemini-2.5-flash thinking token overhead

### Token Counting (Pre-Flight)
```python
encoder = tiktoken.get_encoding("cl100k_base")

def estimate_call(system_prompt, user_message, max_output_tokens):
    input_tokens = len(encoder.encode(system_prompt + user_message))
    worst_case   = input_tokens + max_output_tokens
    # Gemini 2.5 Flash context window = 1,000,000 tokens
    remaining    = 1_000_000 - worst_case
```

---

## 🔑 Key Concepts Learned

**1. The Role System**
Every LLM call has three roles — `system` (sets AI behaviour), `user` (your message),
and `model` (the reply). The system prompt is invisible to the end user but controls
everything about tone, depth, and format of the response.

**2. Temperature vs Top-p**
Temperature controls how randomly the model picks the next token. Low temperature
picks the most likely word every time (safe, repetitive). High temperature picks
from a wider distribution (creative, unpredictable). For factual tasks, use 0.0-0.3.
For creative tasks, use 0.7-1.5.

**3. Gemini Has No Memory**
The model does not remember previous messages. To simulate a conversation, you must
resend the entire conversation history on every API call. This is how every chatbot
works — the "memory" is your responsibility to manage, not the model's.

**4. Thinking Tokens**
Gemini 2.5 Flash reasons internally before writing its reply. Those reasoning tokens
are invisible in the output but count against your `max_output_tokens` budget.
Always set output limits higher than the expected reply length.

**5. Log Everything**
You cannot remember which prompt gave which output after 20 experiments.
A structured log file with parameters, replies, and scores is not optional —
it is how serious ML projects are run.

---

## ⚠️ Limitations

- **tiktoken approximation** — tiktoken uses OpenAI's tokenizer (cl100k_base),
  not Gemini's native tokenizer. Token counts are close but not exact. For
  production systems, use Gemini's own `count_tokens` endpoint.

- **Free tier rate limits** — Gemini free tier allows 15 RPM and 20 RPD for
  Gemini 2.5 Flash. Experiments with many calls require delays between requests.
  Retry logic is built in but adds total runtime.

- **Manual scoring** — the evaluation harness uses rule-based auto-scoring
  (word count, sentence count, keyword detection). Production evaluation uses
  LLM-as-judge or human raters for more nuanced quality assessment.

- **Single model** — all experiments use Gemini 2.5 Flash. Cross-model
  comparison (GPT-4o, Claude, Llama) would require paid API access and is
  not covered here.

---

## 🚀 Future Work

- **LLM-as-judge evaluation** — replace rule-based scoring with a second
  Gemini call that scores each reply against defined rubrics automatically

- **Cross-model comparison** — extend the logging system to support multiple
  API providers (OpenAI, Anthropic, Cohere) and compare outputs across models
  on identical prompts

- **Prompt versioning** — add version tags to the log so prompt iterations
  can be tracked over time, similar to model versioning in MLflow

- **Top-p experiments** — add nucleus sampling (top-p) sweeps alongside
  temperature to observe interaction effects between the two parameters

- **Streaming responses** — replace `generate_content` with streaming mode
  to handle long outputs without timeout issues and display replies token by token

---

## 👤 Author

**Ishan Abrol**

*LLM API Explorer: Understanding LLM Primitives from the Ground Up*

---
