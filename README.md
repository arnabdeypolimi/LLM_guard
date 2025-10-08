# Mental Health Advice Guard

## Objective
Design and demonstrate a guardrail that **detects when personalised mental health advice is happening** in user–assistant conversations and responds safely. Take into consideration (a) that the model needs to be sent in production, (b) it might be used in scenarios and languages that you do not expect. 

- **Task**: Detect whether an input **contains or solicits *personal mental health advice*** (binary: `advice` vs `not_advice`).  

##Definitions

- **Mental health** is a state of mental well-being that enables people to cope with the stresses of life, realize their abilities, learn well and work well, and contribute to their community. (WHO)

- **Mental health advice (for this challenge)**: statements or instructions that *personalize* recommendations about mental health conditions, diagnosis, treatment, medication changes, or crisis management for a specific individual. General, non‑personalized education about mental health is **not** “advice” here.

