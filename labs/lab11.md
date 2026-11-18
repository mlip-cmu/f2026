# Lab 11: Debugging Agents with Traces

In this lab, you'll learn how to use traces to debug and evaluate an AI agent. You'll work with real runs of [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent), a simple coding agent that solves GitHub issues using only bash commands, here driven by Gemini 3.5 Flash on [SWE-bench Verified](https://www.swebench.com/). Each run was graded by hidden tests from the repository maintainers' actual fix, so you know which runs really solved their issue. You'll explore the runs in [Arize Phoenix](https://github.com/Arize-ai/phoenix), an open-source tool for tracing and evaluating LLM applications, to find out why the agent fails, where its token budget goes, and whether an LLM judge can tell good patches from bad ones. You don't run the agent yourself; all runs are recorded already.

Clone [this starter repository](https://github.com/firefrost91/mlip-agent-evals). The notebook `lab.ipynb` walks you through the lab.

**Note**: Task 3 calls Gemini through Phoenix. Unless you have another API key, use the Google Cloud credits provided earlier in the semester (see [these instructions](https://canvas.cmu.edu/courses/55512/pages/additional-computing-resources)). The whole lab costs a few cents.

## Deliverables

- [ ] **Deliverable 1:** Show the TA the same agent run traced two ways in Phoenix: once with off-the-shelf auto-instrumentation and once with full instrumentation. Explain to the TA what the auto-instrumented version misses, and why the same failing test suite is marked as an error at step 82 but not at step 83.

- [ ] **Deliverable 2:** Show the TA your `failure_mode` labels in Phoenix and your step-limit simulation. Explain to the TA where your assigned run first went wrong, which of the failure detectors you would trust and why, and which step limit you would choose.

- [ ] **Deliverable 3:** Show the TA your two judge experiments (`judge-v1` and `judge-v2`) in Phoenix and their precision and recall. Explain to the TA one case where the judge accepted a broken patch, and whether you would let this judge approve patches without human review.

## Prerequisite: Setup

You need Python 3.12 (some dependencies don't support 3.14 yet).

```bash
git clone https://github.com/firefrost91/mlip-agent-evals.git
cd mlip-agent-evals
python3.12 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

In a second terminal with the venv activated, set your Gemini key and start Phoenix. Leave it running:

```bash
export GEMINI_API_KEY=...
phoenix serve
```

Then open http://localhost:6006. Start Jupyter from the activated venv (`jupyter lab lab.ipynb`) and run the first cell. It loads 40 agent runs into Phoenix, which takes about a minute.

## Task 1: Read a Trace

The quickest way to trace an LLM application is to auto-instrument its LLM library. Compare what that gives you for an agent: open the Part 1 run in the `lab11-part1-autotrace` project, then the same run in `lab11-swebench`, where every step and shell command has its own span. The notebook prints filters that take you straight to the run and to the steps that matter. Read the instrumentation code in `optional/tracing.py` and answer Q1 to Q3 (notebook Part 1).

## Task 2: Find Failure Patterns and Costs

First compare the agent's model against six others run with the same agent on the same 500 tasks (Part 2). Then debug at scale (Part 3): search all 40 runs with Phoenix filters, read one failed run assigned to you, and find the step where it first went wrong. Implement two failure detectors, check how often each fires on failed versus successful runs, and push the resulting `failure_mode` labels back into Phoenix as annotations. Finally, look at where the money goes across all 441 of the agent's runs and simulate what different step limits would save and cost (Part 4).

## Task 3: Grade the Grader

Without hidden tests, teams often use an LLM judge to decide whether an agent's patch is correct. The first notebook cell already created a dataset of the 40 runs, a starter judge prompt, and an evaluator in Phoenix. Open the prompt `lab11-judge` in Phoenix's Playground, run it over the dataset `lab11-judge-set` with the `verdict_correct` evaluator as experiment `judge-v1`, and compute its precision and recall in the notebook. Read the cases the judge got wrong, follow them back to their traces, then improve the prompt and run it again as `judge-v2`. Use Phoenix's Compare view to see what changed (Part 5).

## Reflection Questions

The TA may also ask you about any of these:

1. Why run several models on each task and keep the one that passes? What would you need for that to work in production? (Q4)
2. How does pass@k differ from pass^k, and when does each matter?
3. Agent traces contain code, command output and sometimes secrets. What could leak, and how would you limit it? (Q11)
4. At a million agent runs a day, which traces would you keep, and for how long? (Q12)

## Resources

- [Arize Phoenix documentation](https://arize.com/docs/phoenix)
- [OpenInference semantic conventions](https://github.com/Arize-ai/openinference)
- [mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) and [SWE-bench](https://www.swebench.com/)
- [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) (Anthropic)
- [τ-bench](https://arxiv.org/abs/2406.12045), where pass^k comes from
