---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
---

# Overall Harness

In this lab, you will use OpenClaw itself to see how a raw model becomes a full agent turn.

The point is not to learn every OpenClaw feature. The point is to learn the boundary between a model, a provider, a runtime, and the execution constraints around an agent.

## The question

A chat model can answer text. An agent system needs more than that.

When you ask an agent to answer a question about a workspace file, the system has to decide:

* which model provider to call;
* which model to use;
* which session history to include;
* which workspace files and bootstrap instructions to load;
* which tools are visible;
* which tool calls are allowed;
* where the transcript is persisted.

## The path you will observe

```text
Colab notebook
  -> OpenClaw CLI
  -> provider/model
  -> runtime/harness
  -> workspace + context + tools + session
  -> reply + transcript
```

The notebook prints each command before running it and labels what that command is supposed to prove. Treat the output as a white-box observation of which layer is active.

OpenClaw calls the implementation that provides an agent runtime a **harness**. The harness is not the loop itself. The loop is the sequence the runtime executes. Constraints are also not the harness. They are policies and boundaries the runtime has to respect.

```text
user message
  -> provider/model selection
  -> runtime/harness
  -> context assembly
  -> model inference
  -> optional tool calls
  -> final reply
  -> session transcript
```

## What counts as the harness?

Use this table while you run the notebook.

| Layer | Meaning in this lab | Is it the harness? |
| --- | --- | --- |
| Provider | The service or backend that serves models, such as DeepSeek or Ollama. | No |
| Model | The selected model reference. | No |
| Runtime / harness | The implementation that executes a prepared agent turn. | Yes |
| Agent loop | The flow executed by the runtime. | No |
| Context | The prompt, history, tools, and workspace-derived content sent into the turn. | No |
| Workspace | Files and bootstrap instructions the agent can use. | No |
| Constraints | Tool policy, permissions, sandbox, and context limits. | No, but the runtime touches them |

In this first chapter we only touch constraints lightly. Guardrails get their own chapter.

## Open the Colab lab

Open the runnable notebook:

[Open in Colab](https://colab.research.google.com/github/SleepyLGod/openclaw-teaching/blob/main/labs/overall-harness/openclaw_overall_harness.ipynb)

The notebook has two paths:

* **DeepSeek API path**: the stable path for a live demo. It uses `DEEPSEEK_API_KEY`.
* **Ollama path**: the free local-model path inside Colab. It may be slower or less stable on small models.

Both paths use OpenClaw. There is no custom toy agent loop in this lab.

This chapter intentionally does not teach full Guardrails, Eval, Multi-Agent, or Skills. It only introduces the overall runtime boundary those later chapters build on.

## What you should see

| Step | What it proves |
| --- | --- |
| `openclaw --version` | OpenClaw CLI exists in the Colab runtime. |
| `openclaw models list` | The selected provider can be inspected. |
| `openclaw infer model run` | The provider/model path works. |
| `openclaw agent --local` | A full runtime/harness turn can run. |
| Fresh `--session-key` | The observation is not hidden by stale session history. |

## Experiment 1: model probe

The first command is a provider/model probe.

```bash
openclaw infer model run --local --model <model-ref> --prompt "Reply exactly: COURSE_OK" --json
```

This checks whether OpenClaw can reach the selected model. It does not prove that a full agent turn will work.

A model probe does not need the same amount of context, tool schema, workspace state, or session handling as an agent turn.

## Experiment 2: full agent turn

The second command runs through the OpenClaw agent runtime.

```bash
openclaw agent --local --session-key harness-demo \
  --message "Read notes.txt and answer in one sentence: what does this lab teach?"
```

This is the key comparison.

`infer model run` checks the model path. `agent --local` uses the runtime/harness to assemble context, session state, workspace instructions, and tool surface.

## Experiment 3: change the workspace

The notebook copies these files into the OpenClaw workspace:

```text
SOUL.md
USER.md
AGENTS.md
notes.txt
```

Then it uses a fresh session key and asks the agent to describe its role.

A fresh session matters because session history can hide what changed. If you change bootstrap files, use a new session key when you want a clean observation.

## What to observe

While running the notebook, answer these questions.

1. Which command is only a provider/model smoke test?
2. Which command runs a full agent turn?
3. Which provider did you use: DeepSeek or Ollama?
4. Which model did you use?
5. Which workspace files affected the answer?
6. Why can `infer model run` succeed while `agent --local` fails?
7. Which parts are harness/runtime concerns, and which parts are guardrails that we will study later?

## Analysis

The harness is the runtime implementation that turns a prepared prompt and model choice into an agent turn. It is not the same thing as the loop, the prompt, or the policy layer.

The first principle is simple:

> A model returns text. A harness runs an agent turn.

OpenClaw makes this visible because you can run a narrow model probe and then run a full agent turn against the same backend.

That difference is the foundation for the next chapters: Guardrails, Eval, and Multi-Agent.
