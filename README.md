# RigSmith PCs: ReAct Build Advisor with Configurable Personas

**EECE 503P/798S: Agentic Systems** · C4 – Agent Personas & Thinking Mechanisms · Adam Hijazi

This project extends my C3 business chatbot ([business_bot](https://github.com/adammhijazi/business_bot)) into a **ReAct agent built with LangGraph**. The Thought → Action → Observation loop is implemented by hand as a state graph; no prebuilt executor is used. Three personas (Friendly Advisor, Strict Expert, Cautious Helper) are each defined by prompt traits **and** a matching LLM configuration, and compared in logged experiments.

## Files

```text
c4_react_agent/
├── Hijazi_C4_ReAct_Agent.ipynb   # everything: use case, tools, instructions, personas, ReAct graph, experiments, reflection, chat app
├── me/                           # RigSmith company documents (from C3), used by the lookup_policy tool
├── requirements.txt
└── logs/                         # created at runtime: experiments.jsonl (all runs), leads.jsonl
```

## How to run (Google Colab, no GPU)

1. Get a free Gemini API key at <https://aistudio.google.com/apikey>.
2. In Colab: **File → Upload notebook** → `Hijazi_C4_ReAct_Agent.ipynb`.
3. Click the **🔑 Secrets** icon → add `GEMINI_API_KEY` → switch on **Notebook access**.
4. **Runtime → Run all.** When asked, upload the two files from `me/`.
5. The experiments take about **10–15 minutes**, because about 100 API calls are paced for the free tier. The results are saved as they go: if the free limit stops the run, wait a minute and run the cell again, and it continues where it left off.
6. The last cell prints a `gradio.live` link to the chat app.

## What's inside

| Requirement | Where in the notebook |
|---|---|
| Use case: goal, audience, tools, constraints | Section 1 |
| 6 tools: `search_parts`, `price_build`, `check_compatibility`, `estimate_delivery`, `lookup_policy`, `record_lead` | Section 2 |
| Instructions block: one purpose per rule, explicit input/output format, a worked example, no conflict with personas | Section 3 |
| 3 personas: prompt traits + LLM configuration (temperature, top-p, top-k, presence/frequency penalty, max tokens), with the reasoning behind each choice | Section 4 |
| Manual ReAct loop in LangGraph: `think` → (regex action detection) → `act` → `think` … → `finish`, with a stop sequence, format-error recovery and a step limit | Section 5 |
| Experiments: E1 personas × 5 tasks, E2 CoT / few-shot / system-vs-message, E3 temperature, top-p/top-k, penalties, max tokens, model type; automatic success checks; all runs logged | Section 6 |
| Results tables, side-by-side answers, reflection | Section 7 |
| Working assistant: Gradio chat with a persona selector and a visible ReAct trace | Section 8 |

## The ReAct graph

```text
START → start → think ──(Action detected)──► act ──► think
                  │──(format error)──► think   (an error observation is sent back to the model)
                  └──(Final Answer / step limit)──► finish → END
```

The model writes plain text in this format:

```text
Thought: ...
Action: tool_name
Action Input: {"json": "args"}
```

The code detects the action with regular expressions and runs the Python tool. It then appends `Observation: ...` and calls the model again. A stop sequence on `Observation:` stops the model from inventing tool results.

> The parts catalog and its prices are fictional, created for this course.
