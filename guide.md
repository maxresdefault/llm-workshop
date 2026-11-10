Development position — Explore / research | Active: Specialist Reasoning, Evidence Stewardship | Objective: Design a Jane Street-caliber reasoning and quantitative research LLM | Current: Identify the highest-leverage architecture, training methods, repositories, and benchmarks.

The direction I'd recommend is substantially more ambitious than adding Chain-of-Thought, a bigger context window, or a collection of autonomous agents.

Build a quantitative research engine around an LLM, then optimize the LLM against that engine.

Jane Street's own description of quantitative research emphasizes mathematical and logical thinking, analyzing large datasets, building and testing ML models, creating trading strategies, excellent code, research taste, and precision. They also identify OCaml and functional programming as important parts of their engineering culture.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.janestreet.com\&sz=32)

janestreet.com



That's a much better target than simply maximizing an LLM's reasoning benchmark score.

# 1. The highest-impact methods

Here is how I would rank the interventions.

| Priority | Method                                        | Expected value               |
| -------- | --------------------------------------------- | ---------------------------- |
| 1        | Execution-grounded reasoning and verification | Extremely high               |
| 2        | Adaptive test-time compute and search         | Extremely high               |
| 3        | Automated research/experiment loop            | Extremely high               |
| 4        | GEPA and automatic workflow optimization      | Very high                    |
| 5        | Domain-specific SFT and process reward models | Very high                    |
| 6        | Formal reasoning and type-system integration  | High                         |
| 7        | Long-term structured memory and retrieval     | High                         |
| 8        | Inference optimization                        | Medium; primarily speed/cost |

## 1. Execution-grounded reasoning

My first recommendation

The biggest mistake is allowing an LLM to do everything through natural language.

Instead, give it access to actual computational environments.

For example:

- Mathematical derivations → SymPy, Mathematica/Wolfram, numerical solvers.
- Statistical analysis → NumPy, SciPy, JAX.
- Financial data → DuckDB, Polars, SQL.
- Strategy development → Qlib, vectorized backtesting.
- Code verification → compiler, unit tests, property-based testing.
- Formal mathematics → Lean 4 and Mathlib.
- Research → searchable papers, datasets, documentation, and source provenance.

The LLM proposes operations. The execution environment determines whether they work.

An architecture might look like:

```
                 LLM
                  |
            Task decomposition
                  |
           Hypothesis generation
                  |
           Typed experiment spec
                  |
        +---------+----------+
        |         |          |
      Python     SQL        Lean
        |         |          |
        +---------+----------+
                  |
           Independent verifier
                  |
             Evaluation
                  |
           Experiment memory
                  |
          Next-step controller
```

The important change is that the model can no longer simply claim that its calculations are correct.

It must produce executable results.

This is particularly important for quant work, where small numerical errors, incorrect assumptions, and temporal leakage can invalidate an otherwise sophisticated strategy.

## 2. Adaptive test-time compute

This is probably the most promising way to extract substantially more intelligence from an existing model without training a new one.

Relevant papers:

[Scaling LLM Test-Time Compute Optimally Can Be More Effective than Scaling Model Parameters for Reasoning](https://arxiv.org/abs/2408.03314?utm_source=chatgpt.com)

Snell et al., ICLR 2025.

The research investigates allocating additional inference computation to reasoning tasks. Their compute-optimal approach improved efficiency by more than 4× relative to a best-of-N baseline on their evaluated mathematical problems.

[When To Solve, When To Verify](https://arxiv.org/abs/2504.01005?utm_source=chatgpt.com)

An especially interesting paper. It investigates whether computation should be spent generating more solutions or verifying existing ones.

One finding: self-consistency was more compute-efficient than generative reward-model verification across most of their practical inference budgets.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://arxiv.org\&sz=32)

arXiv+1



### What I would implement

Instead of a fixed reasoning budget:

1. Generate several materially different candidate solutions.
2. Measure agreement and uncertainty.
3. Execute inexpensive validity checks.
4. Identify where candidates disagree.
5. Allocate additional computation specifically to those areas.
6. Have an independent process challenge the surviving candidate.
7. Stop when further computation is unlikely to change the answer.

For quantitative research, this becomes:

- Generate competing market hypotheses.
- Implement each hypothesis.
- Test each independently.
- Investigate contradictions.
- Allocate more experiments to promising but uncertain hypotheses.

This is substantially more interesting than generating 50 nearly identical chains of thought.

## 3. Research and development loops

Particularly relevant to quantitative research

Study Microsoft's RD-Agent.

[rdagent · PyPI](https://images.openai.com/static-rsc-4/hig2lEBmwpkYqsEvOqw3cwetH-JEDTPJSssPFUqltFhwgzx8ZF_lTWZ-DPN_fZeWYqB_TnnJA9-0Nq-VoQbz_Qy-UYOJPn_AmzRBdorENALRCypIeSWmXeCX2EZdW-AhUbRHSbKgH51lz7LeLQxN_DSGH9CCRXbBeUoQ_D-4CLQ?purpose=inline)

[pypi.org](https://pypi.org/project/rdagent/?utm_source=chatgpt.com)

Repository: [microsoft/RD-Agent](https://github.com/microsoft/RD-Agent/blob/main/README.md?utm_source=chatgpt.com)

Paper: [R&D-Agent-Quant: A Multi-Agent Framework for Data-Centric Factors and Model Joint Optimization](https://arxiv.org/abs/2505.15155?utm_source=chatgpt.com).

This is one of the closest existing projects to what I think you're describing.

It decomposes quantitative research into:

- Research: propose hypotheses and research directions.
- Development: implement the hypotheses.
- Feedback: evaluate experiments and inform the next iteration.
- Scheduling: use a multi-armed bandit to select promising research directions.

Its NeurIPS 2025 paper reports improvements over classical factor libraries in its evaluated setting.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.microsoft.com\&sz=32)

Microsoft+1



Why this is interesting:

You aren't just asking an LLM to answer questions about finance.

You're allowing it to conduct an iterative research program.

I'd take this architecture considerably further by introducing explicit hypothesis states, experiment dependencies, and independent verification.

# 2. Automatic optimization of the LLM itself

## GEPA: one of the most interesting recent developments

High priority

Paper: [GEPA: Reflective Prompt Evolution Can Outperform Reinforcement Learning](https://arxiv.org/abs/2507.19457?utm_source=chatgpt.com).

Repository: [Stanford DSPy](https://github.com/stanfordnlp/dspy?utm_source=chatgpt.com).

GEPA uses evolutionary optimization and natural-language reflection to improve LLM workflows.

Its process:

1. Execute a particular LLM workflow.
2. Capture its execution traces.
3. Inspect errors, failures, and feedback.
4. Generate improved instructions.
5. Evaluate the new candidate.
6. Preserve complementary high-performing candidates using a Pareto frontier.

The ICLR 2026 paper reports improvements over GRPO averaging six percentage points across six tasks, with up to 35× fewer rollouts in its comparisons. These are benchmark-specific results, not a guarantee of universal improvement.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://proceedings.iclr.cc\&sz=32)

ICLR Proceedings+1



This is particularly attractive because it doesn't initially require expensive model training.

### How I'd use it

Create separate DSPy modules for:

- Mathematical reasoning.
- Quantitative hypothesis generation.
- Code generation.
- Statistical evaluation.
- Adversarial reviewing.
- Paper retrieval.
- Experiment interpretation.

Give each module its own evaluation function.

Then optimize the complete workflow, not just the system prompt.

One particularly interesting extension would be optimizing the interactions between modules.

For example, a research module that generates slightly less novel hypotheses but substantially more testable hypotheses might produce better overall research results.

That is something an end-to-end evaluator can discover.

# 3. Financial reasoning and reward modeling

There are some especially relevant financial LLM projects.

## QuantEval

Paper: [QuantEval: A Benchmark for Financial Quantitative Tasks in Large Language Models](https://arxiv.org/abs/2601.08689?utm_source=chatgpt.com).

This benchmark evaluates three different capabilities:

- Financial knowledge.
- Quantitative mathematical reasoning.
- Quantitative strategy coding.

It also executes generated strategies using a backtesting framework.

One particularly useful result concerns DianJin-R1-7B:

| Training stage | Knowledge QA | Reasoning | Strategy coding Sharpe MAE |
| -------------- | ------------ | --------- | -------------------------- |
| Initial        | 50.3         | 42.8      | —                          |
| SFT            | 53.2         | 47.8      | 0.85                       |
| SFT + GRPO     | 52.2         | 48.4      | 0.72                       |

Lower Sharpe MAE is better.

Notice the trade-off: reinforcement learning improved reasoning and strategy coding but slightly reduced knowledge QA. This is precisely why you need multiple independent evaluation dimensions.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://arxiv.org\&sz=32)

arXiv+1



## DianJin-R1

Repository: [Alibaba Qwen DianJin](https://github.com/aliyun/qwen-dianjin?utm_source=chatgpt.com).

This project is worth studying for its financial reasoning training pipeline.

It uses:

- Reasoning-augmented supervised fine-tuning.
- GRPO reinforcement learning.
- Financially oriented training datasets.
- Explicit formatting and correctness rewards.

It is a useful reference for domain specialization, rather than blindly fine-tuning your base model on financial text.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



## Fin-PRM: particularly interesting

Paper: [Fin-PRM: A Domain-Specialized Process Reward Model for Financial Reasoning](https://www.ijcai.org/proceedings/2026/683?utm_source=chatgpt.com).

This is a financial-domain process reward model.

Instead of evaluating only the final answer, it evaluates:

- Correctness of individual reasoning steps.
- Consistency across an entire reasoning trajectory.
- Financial knowledge grounding.

It uses approximately 3,000 financial reasoning trajectories with step-level and trajectory-level supervision.

The paper investigates using the reward model for:

- Selecting better training examples.
- Reward-guided Best-of-N inference.
- Reinforcement learning.

This is very close to the kind of specialized evaluator I'd eventually want for your system.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.ijcai.org\&sz=32)

ijcai.org



# 4. Mathematical formalization and proof

I'd give the LLM access to a formal mathematical environment.

[Peiyang Song](https://images.openai.com/static-rsc-4/CqKWbF8j5CyiwDliu1CikPzKK0yXsBfe1YM6w5bR1I4Gp5mXm2zHnXqiXEKo0knryNIfnCF714quqnnYWEInU4fbWq0KR-Jk4hnLetxlsoH5ZfLgln_GJoyQQMejE4iDd5BUoCILlugqXEC-gJWh3Lv1UoNvmQzRqDjebusKedw?purpose=inline)

[peiyang-song.github.io](https://peiyang-song.github.io/?utm_source=chatgpt.com)

Start here:

- [LeanDojo-v2](https://github.com/lean-dojo/LeanDojo-v2?utm_source=chatgpt.com)
- [Lean 4](https://github.com/leanprover/lean4?utm_source=chatgpt.com)
- [Mathlib4](https://github.com/leanprover-community/mathlib4?utm_source=chatgpt.com)

LeanDojo-v2 is particularly compelling because it combines:

- Theorem retrieval.
- Structured proof-state extraction.
- SFT and GRPO training.
- Retrieval-assisted theorem proving.
- External model inference.

It can use an LLM to propose proofs, then Lean to determine whether they are actually valid.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



### Why this matters beyond mathematics

Consider giving the model a task:

"Find the optimal configuration satisfying these constraints."

It should ideally:

1. Formulate the constraints.
2. Construct the candidate solution.
3. Prove relevant invariants.
4. Run numerical checks.
5. Report remaining assumptions.

You won't formalize every statistical hypothesis in Lean. But for symbolic reasoning, algorithm correctness, and mathematical transformations, this is a powerful additional layer.

# 5. Repositories I'd actually investigate

Here is my shortlist, ordered by practical relevance.

## Core architecture

[Magentic-One: A Generalist Multi-Agent System for Solving Complex Tasks - Microsoft Research](https://images.openai.com/static-rsc-4/ClIiAGc_j8OZr0d2v3LyamqIJfwanksRIeAxZ8j8J9Y7yo3FJqSeLwJFaeglZESxOkVnuPinYqNwlJivrbfo_jqYJa-aygXKmJv3BGw3GpMlo4mlo5rBd_vAK8MHGRa0SCcyFwwA5fRQODnqCyWWHrE1ghMvwaIeRxdmVHsoWFs?purpose=inline)

1\. [Microsoft RD-Agent](https://github.com/microsoft/RD-Agent/blob/main/README.md?utm_source=chatgpt.com)

Research/development loops, experiment generation, evaluation feedback, adaptive scheduling.

Highest priority

[GitHub - microsoft/qlib: Qlib is an AI-oriented Quant investment platform that aims to use AI tech to empower Quant Research, from exploring ideas to implementing productions. Qlib supports diverse ML modeling paradigms, including supervised learning, market dynamics modeling, and RL, and is now equipped with https://github.com/microsoft/RD-Agent to automate R&D process.](https://images.openai.com/static-rsc-4/U_dbR3ofYQ5r4-SsCCkT0Ow3b6K2hUk7tdLpILNZYGSpvw8qkUWShkmC1lzG2Yf1ha9011SkQsQyigJcGtG96FYp4rynNex1_q1wY0dYXQ3V29rhl3mS0-WeFkD9X-Z23qJbPQHy45dO84tsQmcQVOYBR2A1MURthQNh5v8XoBk?purpose=inline)

2\. [Microsoft Qlib](https://github.com/microsoft/qlib?utm_source=chatgpt.com)

Full quantitative ML pipeline, alpha research, risk modeling, portfolio optimization, backtesting and execution.

[GitHub - stanfordnlp/dspy: DSPy: The framework for programming—not prompting—language models · GitHub](https://images.openai.com/static-rsc-4/9hoplrIOXaU-eUN7tIEIImpm2GHJBVluumNSgPibvi2AYcFB9-RQpp0agWebU13XUOFH2Y0d72fmr42_ESHP_8E1w36imEs_q_vbPwUONWLduXO4oiFbRoNgHACVQ_0BMRXi1WfLFANUyTzQVyh4Jl0RICys_B3JFKP8wUD68EI?purpose=inline)

3\. [Stanford DSPy](https://github.com/stanfordnlp/dspy?utm_source=chatgpt.com)

Declarative LLM programming, workflow optimization, GEPA, prompt and module evolution.

Highest priority

[Software Agent SDK - OpenHands Docs](https://images.openai.com/static-rsc-4/NUI5U4zdAzbHQnv5vKXkMhLdzgHUsWpK87nQKWFF7GS2sztDESYH5tPsrYHBwb1a_bZzAq6Zy8eX257BfJWSG0nyi8qNqCkedrmgiGn2tLCRKQBGO-vu1XjEgBZ0Gn_X8zbD0xfcOml_sK98qs5cWMzGU0U-qvJrZ5oJiWPXmOo?purpose=inline)

4\. [OpenHands Software Agent SDK](https://github.com/OpenHands/software-agent-sdk?utm_source=chatgpt.com)

Code-writing agents, sandboxed workspaces, execution, tools, and multi-step engineering tasks.

[qwen-dianjin/DianJin-R1/src/rl/grpo.sh at master · aliyun/qwen-dianjin · GitHub](https://images.openai.com/static-rsc-4/5yU8EShsH-DnYhNlrKPf5bGKRpWADpxvjZ535fsKTrxElojeIzfFOqTVgcBfZWGFQKED8a5ucYz1fos1ITbgHoc9S1X7RH4KzQMzAOeKBYRE_rMQO2URe4WxY_-sJ9Gk8bWCqjxOlQi2mdrlbZ2BGR2MY5MmLQOE5Y1yvN8VCd4?purpose=inline)

5\. [Qwen DianJin](https://github.com/aliyun/qwen-dianjin?utm_source=chatgpt.com)

Domain-specific financial reasoning, financial reward models, training data, and agentic tool use.

## More specialized tools

| Repository                                                                                        | Purpose                                |
| ------------------------------------------------------------------------------------------------- | -------------------------------------- |
| [LeanDojo-v2](https://github.com/lean-dojo/LeanDojo-v2?utm_source=chatgpt.com)                    | Formal reasoning                       |
| [PrimeIntellect Verifiers](https://github.com/PrimeIntellect-ai/verifiers?utm_source=chatgpt.com) | RL environments and verifiable rewards |
| [Nous Atropos](https://github.com/nousresearch/atropos?utm_source=chatgpt.com)                    | Interactive LLM reinforcement learning |
| [vLLM](https://github.com/vllm-project/vllm?utm_source=chatgpt.com)                               | High-throughput inference              |
| [Speculators](https://github.com/vllm-project/speculators?utm_source=chatgpt.com)                 | Speculative decoding                   |
| [Jane Street OSS](https://github.com/janestreet?utm_source=chatgpt.com)                           | OCaml engineering libraries            |

For the last entry, I'd particularly examine Jane Street's:

- [Base](https://github.com/janestreet/base?utm_source=chatgpt.com)
- [Core](https://github.com/janestreet/core?utm_source=chatgpt.com)
- [Async](https://github.com/janestreet/async?utm_source=chatgpt.com)

These are useful examples of the functional and strongly typed engineering ecosystem Jane Street publicly maintains.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub+1



# 6. The research papers worth reading

A curated reading list:

| Paper                                                                                    | Why it matters                                 |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------- |
| [DeepSeek-R1](https://arxiv.org/abs/2501.12948?utm_source=chatgpt.com)                   | RL-induced reasoning, reflection, verification |
| [Scaling Test-Time Compute](https://arxiv.org/abs/2408.03314?utm_source=chatgpt.com)     | Compute allocation and inference scaling       |
| [When To Solve, When To Verify](https://arxiv.org/abs/2504.01005?utm_source=chatgpt.com) | Solution generation vs verification            |
| [GEPA](https://arxiv.org/abs/2507.19457?utm_source=chatgpt.com)                          | Evolutionary workflow optimization             |
| [RD-Agent-Quant](https://arxiv.org/abs/2505.15155?utm_source=chatgpt.com)                | Automated financial research                   |
| [QuantEval](https://arxiv.org/abs/2601.08689?utm_source=chatgpt.com)                     | Quantitative finance reasoning benchmark       |
| [Quant-Arena](https://arxiv.org/abs/2610.30113?utm_source=chatgpt.com)                   | End-to-end strategy research evaluation        |
| [Fin-PRM](https://www.ijcai.org/proceedings/2026/683?utm_source=chatgpt.com)             | Domain-specific process reward modeling        |

One paper deserves special emphasis: Quant-Arena.

It evaluates models on an entire quantitative research loop, from hypotheses and feature engineering to a frozen strategy evaluated on held-out market data.

In its nine-model experiment, only one of 38 iterative configurations beat the equal-weight market reference in the sealed 2024 period. Its analysis also finds that improvements often came from market timing rather than stronger stock selection.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://arxiv.org\&sz=32)

arXiv+1



That is an excellent reminder that writing sophisticated quantitative code and discovering a genuinely profitable signal are very different abilities.

# 7. What I would build differently: a relational experiment engine

This is where I think there is an opportunity to go beyond simply assembling existing projects.

I would give the system a structured representation of the research process.

Instead of storing all research as conversation transcripts, represent it as a typed graph.

Research knowledge graph

Hypothesis

A testable claim about a system or market

formalized as

Constraints + predictions + assumptions

implemented through

Experiment / executable artifact

produces

Results + verification evidence

updates

Research direction / next hypothesis

Each hypothesis would have:

- Preconditions.
- Predicted effects.
- Supporting evidence.
- Contradicting evidence.
- Dependencies.
- Experiment specifications.
- Falsification conditions.
- Confidence and uncertainty.
- Links to prior experiments.

Relationships could include:

`derived_from`, `contradicts`, `implies`, `depends_on`, `tested_by`, `supersedes`, `generalizes_to`.

This prevents the LLM from repeatedly rediscovering failed approaches and makes it possible to search across related experiments.

It also gives you structured data for later training.

An especially interesting extension: Use a multi-armed bandit or evolutionary controller to select the next experiment based on expected information gain, expected utility, and computation cost.

This would make the research process itself an optimization problem.

# 8. How I would implement it on your hardware

Your RTX 3070 Laptop has 8 GB VRAM, and you've been experimenting with quantized 27B models.

I would avoid attempting to train a large reasoning model locally or run numerous simultaneous agent instances against that GPU.

Instead:

Practical hybrid configuration

Local model

Primary worker

Small quantized model for routing, extraction, summarization, and inexpensive candidate generation.

External reasoning model

High-capability API model for difficult mathematical problems, research hypotheses, and difficult code.

Local computational environment

Python, DuckDB, Polars, SymPy, Lean, and isolated code execution.

Optimization

DSPy/GEPA first; then carefully controlled SFT and RL experiments.

For local fine-tuning, investigate Unsloth, Hugging Face TRL, and QLoRA. A small 1.5B–3B specialized model is a much more realistic initial training target on your GPU. Even 7B QLoRA requires careful memory management and compatible training configurations.

Do the expensive RL rollouts remotely or on more capable hardware.

# 9. The evaluation system is arguably the most important part

I wouldn't optimize the model without first building a serious benchmark.

Create a private, reproducible collection of tasks.

| Capability            | What should be measured                            |
| --------------------- | -------------------------------------------------- |
| Mathematics           | Exact correctness, proof validity                  |
| Programming           | Compilation, test pass rate, correctness           |
| Quantitative analysis | Calculation accuracy, statistical validity         |
| Research              | Hypothesis quality, evidence coverage              |
| Strategy development  | Out-of-sample returns, Sharpe, drawdown            |
| Tool use              | Correct tool selection and execution               |
| Reasoning             | Multi-step correctness and contradiction detection |
| Efficiency            | Cost, latency, tokens per successful task          |

For financial strategies, enforce:

- Strict chronological train/validation/test separation.
- No lookahead or survivorship leakage.
- Transaction costs and slippage.
- Market impact and capacity constraints.
- Walk-forward validation.
- Baseline comparisons.
- Frozen final strategies before out-of-sample evaluation.

Do not let the model repeatedly inspect the final test period.

Your benchmark should measure actual improvements, not whether the model produces more convincing explanations.

# 10. My recommended implementation sequence

I'd approach this in five stages.

1. Stage 1 — Build the evaluator

   Create 100 difficult tasks spanning math, code, financial reasoning, and research. Establish reproducible scoring.
2. Stage 2 — Build the research engine

   Start with RD-Agent + Qlib, add deterministic tools and sandboxed execution, and implement an experiment ledger.
3. Stage 3 — Implement reasoning optimization

   Use DSPy and GEPA to optimize the individual agent modules and their complete workflow. Introduce adaptive test-time compute.
4. Stage 4 — Specialize the model

   Build high-quality task-specific SFT data. Experiment with domain-specific process rewards and GRPO once you have reliable verifiable environments.
5. Stage 5 — Research automation

   Add experiment scheduling, evolutionary hypothesis generation, adversarial reviews, and automatic discovery of new research directions.

## My top three recommendations

First

Microsoft RD-Agent + Qlib

The foundation for an automated quantitative research system.

Second

DSPy + GEPA

The foundation for continuously improving the LLM's workflows without immediately retraining its weights.

Third

LeanDojo + Fin-PRM

The foundation for more rigorous mathematical and domain-specific intermediate reasoning verification.

My strongest overall thesis: The real opportunity isn't an LLM that thinks longer. It's an LLM that can generate competing models of a problem, transform them into executable experiments, obtain independent evidence, discover why they failed, and use those failures to improve its next attempt.

That's much closer to the research-engineering process you're trying to emulate than a conventional reasoning chatbot.

A useful next step would be a monthly scan of new reasoning, quantitative-agent, and LLM optimization papers and repositories, filtered for things that could materially improve this architecture.

🔬Monthly quant-LLM research scanTap to set up

Development position at close — Explore / research | Active: Specialist Reasoning, Evidence Stewardship | Objective: Design a Jane Street-caliber reasoning and quantitative research LLM | Current: Prioritized architecture, implementation repositories, research papers, hardware strategy, and evaluation methodology | Boundary: Research synthesis complete | Human input: None.