# Lab 4: Model Testing with Weights & Biases and LLMs

In this lab, you'll gain hands-on experience using **Weights & Biases (W&B)** for interactive model evaluation and **LLMs** to generate targeted test cases.

You work on a team that automatically flags tweets. A sentiment model is **already running in production** — the **baseline**. Two newer models are proposed as replacements, **candidate_v1** and **candidate_v2**, and your job is to decide whether either should **ship** (turn off the baseline and use the candidate instead). The tempting shortcut is to compare one accuracy number and pick the winner. This lab is about why that shortcut gets people hurt, and what to do instead.

Steps 1–7 are **exploration**: you slice the predictions, find failure modes in W&B, and stress test a weak slice with tweets an LLM writes on purpose. Step 8 is **enforcement**: you turn what you found into automated tests with thresholds fixed in advance, then score an unseen model against them.

To receive credit for this lab, show your work to the TA during recitation.

## Deliverables
- [ ] **Five slices** in `slices.py` that you can justify, plus your notes in `saved_slice_notes`. Explain the hypothesis behind each slice: what about these tweets might confuse a model?
- [ ] **A W&B run** containing `df_long`, `slice_metrics`, `regression_metrics`, `df_eval`, and at least one slice chart. Walk the TA through it and answer: would you ship `candidate_v1`?
- [ ] **Ten LLM-generated tweets** targeting one weak slice (Step 7), and what they showed. Explain whether the results change your confidence in the candidate.
- [ ] **A frozen gate**: your `tests/manifest.yaml`, plus pytest output for `MODEL=baseline` and `MODEL=candidate_v1`.
- [ ] **A five-sentence memo on `candidate_v2`**: run the same gate on v2 and recommend ship / no-ship with one mitigation.

## Key ideas
You only need five concepts; everything in the lab is one of these.

1. **Slice.** A subgroup of the data sharing a property you care about — negation (`not`, `never`, `isn't`), hashtags, very long tweets, heavy emoji use. A model can score 70% overall and 30% on one slice; the average hides the hole.
2. **Regression.** The baseline got a tweet **right** and the candidate gets it **wrong**. This is not the same as "the candidate is less accurate" — it is a specific thing that used to work and now doesn't. Users notice regressions.
3. **Confident regression.** A regression where the candidate was *sure* (confidence ≥ 0.8) and still wrong. Worse than an unsure mistake, because nothing downstream can tell it was a guess.
4. **Threshold / gate.** A rule written down *in advance*: "accuracy on the negation slice must be at least 0.60." A **gate** is a set of such rules a model must pass. The rule is a promise about product quality, not a score you adjust until the model passes.
5. **p50 latency.** The median time to score one tweet. This model runs on a **live moderation stream**, so slow is a real failure even when accuracy is perfect. The product budget is **50 ms per tweet on one CPU thread**.

## The three models
All three have already been run over the same 500 tweets, with predictions saved in `tweets.csv`. You do **not** run the models yourself for Steps 1–6 — no GPU, no waiting, no network.

| Name | Role | Size | Column prefix |
| - | - | - | - |
| baseline | currently in production | 125M (RoBERTa-base) | `roberta` |
| candidate_v1 | a smaller, faster fine-tune | 124M (GPT-2) | `gpt2` |
| candidate_v2 | a much larger model | 435M (DeBERTa-v3-large) | `deberta` |

Each prefix has three columns: the prediction, `_score` (confidence, 0 to 1), and `_latency_ms` (time to score that tweet, measured on one machine, one thread, one tweet at a time — so your laptop's speed does not change the result). `label` is the correct answer: `negative`, `neutral`, or `positive`.

The two candidates were built with opposite bets: one is smaller and quicker, the other is three times the size and aims for accuracy. Neither bet is automatically right — that is what you are here to measure.

**Keep `INCLUDE_CANDIDATE_V2 = False` until Task G.** You write your thresholds while looking only at the baseline and v1, so v2 must pass rules you wrote before you ever saw its scores. That is how it works on a real team, and it stops you from quietly moving the goalposts.

## Getting started
- Clone the starter code from this [Git repository](https://github.com/firefrost91/Lab-4).
- Python 3.10 or newer. From the repo root, install the dependencies:
  ```bash
  pip install --upgrade wandb datasets transformers torch tqdm emoji pandas pyarrow scikit-learn pytest pyyaml
  ```
- Create a free W&B account at [wandb.ai](https://wandb.ai) with your CMU email, copy your key from [wandb.ai/authorize](https://wandb.ai/authorize), then run `wandb login` in a **terminal** (not the notebook) and paste the key when prompted.
- Open `lab4.ipynb` and run the cells in order.

**About downloads.** Steps 1–6 and Step 8 read the saved predictions in `tweets.csv`, so they need no model downloads and no GPU. Only **Step 7** downloads models, because your generated tweets are new — about 1 GB for `baseline` + `candidate_v1`, cached after the first run. Do Step 7 before you flip `INCLUDE_CANDIDATE_V2`: `candidate_v2` is a 1.7 GB download and roughly 6× slower per tweet, and you never need to run it yourself.

## What each file does
**You edit exactly three files.** Everything else is infrastructure — if you find yourself changing it, stop and re-read the task.

| File | Do you edit it? | What it is for |
| - | - | - |
| `slices.py` | **Yes — Task B** | Your metadata columns and slice definitions. |
| `tests/manifest.yaml` | **Yes — Task F** | The thresholds your gate enforces. |
| `lab4.ipynb` | **Yes — the `#TODO` cells** | The lab itself, Steps 1–8. |
| `tweets.csv` | No | 500 tweets, correct labels, and every model's saved predictions, confidences, and latencies. |
| `lab_helpers.py` | No | The model registry, plus prediction loading and latency stats for the tests. |
| `tests/test_slices.py` | No | The gate itself. Reads *your* manifest and checks one model. |
| `tests/conftest.py`, `pytest.ini` | No | Plumbing: writes `gate_results_{MODEL}.csv`, puts the repo root on the import path. |
| `scripts/add_model_predictions.py` | No | How `tweets.csv` was built. Only needed if you add a fourth model. |

Inside `lab4.ipynb` there are four spots to fill in, all marked `#TODO`:

1. **Step 6 — `saved_slice_notes`**: one or two sentences per slice, what you found and why.
2. **Step 7 — `generated_slice_description`, `generated_cases`, `intended_labels`**: your hypothesis and your 10 LLM-generated tweets with the labels you believe are correct.
3. **Step 1 — `INCLUDE_CANDIDATE_V2 = True`**: flip this in Task G only, *after* your manifest is frozen.
4. **Steps 4.5–5.5 — the cells marked "Replace with your metadata" / "Edit to work for your slices"**: usually nothing to do, since they already read `META_COLS` and `get_slices()` from `slices.py`. Touch them only if a slice of yours needs a column the pivots don't carry.

Step 3 also has a `#TODO`, but there is no code to write there — it is telling you to go edit `slices.py`.

## Task A: Get set up (Steps 1–2)
- Install the packages and confirm Python ≥ 3.10.
- Run `wandb.login()` in the notebook after logging in via terminal.
- Load `tweets.csv`. Leave `USE_HF_DATASET = False` — this reads saved predictions instead of downloading and running models.

## Task B: Define your slices (Step 3, edit `slices.py`)
The file ships with five example metadata columns (`emoji_count`, `has_hashtag`, `has_mention`, `has_negation`, `length_bucket`). These are examples, not your answer.

- Add metadata columns of your own. Ideas: ALL-CAPS text, question marks, URLs, sarcasm words, strong sentiment words, multiple sentences.
- Define **at least 5 slices** in `get_slices()`. For each, write down the hypothesis.
- Keep `META_COLS` in sync (the notebook checks you have at least 5), then re-run Step 3.

**Aim for slices with at least 30 tweets.** A slice of 4 tweets can swing from 25% to 100% by luck, so it cannot support a decision. The gate in Step 8 skips anything smaller for exactly this reason.

## Task C: Compute the numbers (Steps 4–5)
- Step 4 builds `df_long`: one row per tweet per model. With `INCLUDE_CANDIDATE_V2 = False` you should see only `baseline` and `candidate_v1`.
- Look at overall accuracy for both models, then compute accuracy **per slice**, per model.
- Compute regressions vs the baseline: regressed, improved, both wrong, both right, and confident regressions.

The interesting question is not "which number is bigger." It is *where* the candidate loses, and whether that place matters for moderation.

## Task D: Explore in W&B (Step 6)
- Log `predictions_table`, `slice_metrics`, `regression_metrics`, and `df_eval`.
- Open the run. In **Tables**, filter `predictions_table` by your metadata columns.
- Build a bar chart comparing baseline vs candidate accuracy across slices.
- Read a few tweets the candidate got wrong on your worst slice. Look for a pattern, not just a bad score.
- Write 1–2 sentences per slice in `saved_slice_notes`.

Be ready to answer: *why can overall accuracy be misleading here, and what did slicing reveal?*

## Task E: Stress test with an LLM (Step 7)
Slicing can only find failures on tweets you happen to have. If you suspect a weakness, you can **write new tweets on purpose** to test it.

- Pick the one slice that worried you most in Task D.
- Write your hypothesis in one or two sentences (`generated_slice_description`).
- Ask an LLM for **10 tweets** that probe it. Include easy cases, subtle cases, and near-identical pairs where one word flips the sentiment.
- Fill in `generated_cases` and `intended_labels` (10 each — you decide the correct label).
- Run the scoring cell. **This one needs internet** (~1 GB, first run only) because it downloads and runs the models on your new tweets. Do it while `INCLUDE_CANDIDATE_V2 = False`.
- Note whether the same failure repeats, or something new appears.

## Task F: Freeze the gate, then run it (Step 8)
Now you turn findings into rules. Write the thresholds **before** you run pytest on a candidate. If you tune a threshold until a model passes, the test no longer means anything.

- Add the slices you want to enforce to `tests/manifest.yaml`. The name must exactly match a key in `get_slices()`.
- For each, set a `threshold` and write a one-line `rationale` (why this number, and what breaks if we go lower).
- Leave `p50_ms_max: 50` (the product budget) and write why that number matters.

```yaml
slices:
  - slice: has_negation
    threshold: 0.60
    rationale: "A missed negation flips the meaning of the tweet."
latency:
  p50_ms_max: 50
  rationale: "Live moderation stream budget, one CPU thread."
```

Then, from the **repo root**:

```bash
MODEL=baseline pytest tests/ -v
MODEL=candidate_v1 pytest tests/ -v
```

- Notice that forgetting `MODEL=` fails immediately, on purpose — a gate should never silently test the wrong model.
- Notice which tests are **skipped** and why (a slice with under 30 tweets cannot support a decision).
- Each run writes `tests/gate_results_{MODEL}.csv`. Load it in the notebook and log it to W&B as `gate_results`.

A failing gate is a **result**, not a bug in your code. Write down exactly what failed, and notice that "wrong" and "slow" are different kinds of failure.

## Task G: Score `candidate_v2` against the gate you already wrote
- Set `INCLUDE_CANDIDATE_V2 = True` in Step 1 and re-run Steps 4–6 to see v2 in W&B. (Still no download — v2's predictions are already in `tweets.csv`. Just don't re-run Step 7.)
- Run `MODEL=candidate_v2 pytest tests/ -v` with the **same** manifest.
- `slices.py` and `tests/manifest.yaml` are **frozen from here on**. Editing either one now — or the tests — to make v2 look better is the exact failure this lab is about.

Then write **five sentences**:

1. Should v2 replace the baseline?
2. What did overall accuracy suggest, and what did the slices and the gate say?
3. Which single check decided it for you?
4. If you would not ship as-is, what is one mitigation? (Gather training data for the weak slice, keep the baseline on that slice, batch requests, quantize or distill to a smaller model, raise the latency budget with sign-off, …)
5. Your final recommendation.

A model can fail for two very different reasons: it is **wrong** where it matters, or it is **too slow to use**. Both block a ship. Across these three models you will see both failure modes, and no single model wins on both axes — so your recommendation has to say what you are trading away, and your mitigation should address the specific thing that failed. Making a big model faster (quantizing, distilling, batching, better hardware) is a different project from making a small model smarter.

## Questions your TA may ask
- What is a regression, in your own words? Why is it worse than a slightly lower average?
- Your worst slice: what is the hypothesis behind it?
- Why did we make you write thresholds before looking at `candidate_v2`?
- Why skip slices with fewer than 30 tweets?
- Can a model pass on accuracy and still be unshippable?

## Troubleshooting
- **`MODEL environment variable is not set`** — run pytest as `MODEL=baseline pytest tests/ -v`, from the repo root.
- **`Unknown slice 'x'`** — the name in `manifest.yaml` must match a key returned by `get_slices()` in `slices.py`.
- **A test says SKIPPED** — the slice has fewer than 30 tweets, so it cannot support a decision. Intentional. (The vs-baseline test also skips when `MODEL=baseline`, since comparing the baseline to itself is meaningless.)
- **A candidate fails the gate** — that is a finding, not a setup error. Report it instead of "fixing" the threshold.
- **Step 7 fails to download a model** — that cell needs internet. Steps 1–6 do not.
- **Accuracy changed after I edited `slices.py`** — expected. Slice membership changed, so slice accuracy changes. Overall accuracy should not move.

## Additional resources
- [W&B tables and panels](https://docs.wandb.ai/guides/app/features/panels/)
