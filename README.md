# Mental Health Advice Guard

> **Why**: We want to see how you think how you transform a social problem into LLM guardrails.

## Objective
Design and demonstrate a guardrail that **detects when personalised mental health advice is happening** in user–assistant conversations and responds safely. Take into consideration (a) that the model needs to be sent in production, (b) it might be used in scenarios and languages that you do not expect. 

- **Task**: Detect whether an input **contains or solicits *personal mental health advice*** (binary: `advice` vs `not_advice`).  

##Definitions

- **Mental health** is a state of mental well-being that enables people to cope with the stresses of life, realize their abilities, learn well and work well, and contribute to their community. (WHO)

- **Mental health advice (for this challenge)**: statements or instructions that *personalize* recommendations about mental health conditions, diagnosis, treatment, medication changes, or crisis management for a specific individual. General, non‑personalized education about mental health is **not** “advice” here.


## Deliverables
At minimum, provide **one runnable notebook** (Colab or Modal Notebook) that we can open and run end‑to‑end:
- Loads a dataset.
- Implements your detector with **clear explanations of why**.
- Produces evaluation metrics and **error analyses**.
- Exposes a `guard_decide(text)` function that returns `{label, rationale, action, response_template}`.

Also provide a report that:
- Documents your data, model, rubric design choices.
- Documents **limitations** and **what you would do with more time**.

> You’re welcome to add scripts, small modules, or extra notebooks if it helps you tell the story. We’ve left parts open‑ended on purpose to see **design choices and rationales**.

## Time Guidance
We expect this to take between 6–10 hours total, but if it takes less/more time, let us know. Depth is valued over breadth, so feel free to invest more time in things you believe can make a difference.

## How to work (so we can run it)
- **Use a notebook** (e.g. Colab). 
  - Colab’s free tier typically runs up to **~12 hours per session**;
- Include any dependency installs **inline** (e.g., `%pip install ...`).
- Seed your randomness and note versions for reproducibility.
- Organize however you like—one notebook is fine. A short `reports/` summary is expected.
