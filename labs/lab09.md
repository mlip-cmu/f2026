# Lab 9: Data, Pipeline & Model Versioning with DVC and MLflow

In this lab you will take one ML pipeline and answer a question every ML team eventually faces: **"Where did this model come from, and should it be in production?"**

You will use two tools that do different, complementary jobs:

- **DVC** versions the **inputs**: data, pipeline and reproducibility.
- **MLflow** records the **outcomes**: runs, metrics and models, and helps you promote and serve them.

The pipeline trains a Random Forest on the Breast Cancer Wisconsin dataset to classify tumors as malignant or benign.

## Learning Objectives

1. Version datasets and run reproducible pipelines with DVC
2. Instrument training code to log to MLflow
3. Compare what DVC records implicitly with what MLflow records explicitly
4. Compare experiments and justify a model choice
5. Register and serve models, and compare them before promoting one
6. Trace a registered model back to the exact code and data that produced it

## The Lab

Use the [resources](#resources) and each tool's `--help` to work out how. If you get stuck, ask a TA.

### Part 1: DVC

- Put the data under DVC and set up a local remote.
- Use `scripts/augment_data.py` to create a second data version, then switch back to the original.
- Run the pipeline in `dvc.yaml`, view its DAG, and see which stages re-run when something changes.
- Run experiments with `max_depth` set to 1, 2 and 5, and compare them.

### Part 2: MLflow tracking

- Runs from before this part appear empty in the MLflow UI. That's expected, because nothing is logged until you add the code.
- Complete the six TODOs in `scripts/train.py` and `scripts/evaluate.py` to log params, lineage tags, the model, metrics and the confusion matrix. All of them should land in one run.
- Run the same three experiments, plus 1–2 of your own. For each one, edit `params.yaml`, commit, then run `dvc repro`. Don't use `dvc exp run` here: each run is tagged with the current git commit, so the run must match a commit.
- Compare the runs in the MLflow UI, choose a best model, and fill in the [comparison table](#comparison-table).

### Part 3: Registry and serving

- Register the `max_depth=1` model and your best model as versions of `breast-cancer-classifier`, with the aliases `champion` and `challenger`.
- Load the champion with `scripts/predict.py`.
- Serve the champion on port 5002 and the challenger on port 5003 (use `--env-manager local` with `mlflow models serve`), then compare them with `scripts/compare_serving.py`.
- Decide whether to promote the challenger. If you do, move the alias and find out what it takes for a running server to pick up the change.

### Part 4: Trace and reproduce

- Rebuild version 1 of the registered model from scratch, starting from the registry and using only what its run recorded.
- Confirm that the new run's metrics exactly match the original.

## Deliverables

Show the TA:

1. **DVC:** your pipeline DAG and experiment comparison.
2. **MLflow tracking:** at least 4 logged runs, plus your completed [comparison table](#comparison-table).
3. **Serving:** two models served side by side, and your promotion decision with reasons.
4. **Reproduction:** a rebuilt run whose metrics exactly match the original.
5. **Reflection:** be ready to discuss the [reflection questions](#reflection-questions).

## Setup

You need Git and Python 3.10+. Check with `python3 --version` before creating the venv (the macOS system Python is too old). DVC and MLflow work natively on macOS, Windows and Linux; if you hit setup issues, use your team's VM.

Clone [this repository](https://github.com/aku134/mlip-lab-9), then set up your environment:

```bash
git clone https://github.com/aku134/mlip-lab-9.git
cd mlip-lab-9
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

In a second terminal with the venv activated, start the MLflow UI and leave it running:

```bash
mlflow ui --backend-store-uri sqlite:///mlflow.db --port 5001
```

Then open http://localhost:5001.

## Comparison Table

Fill this in during Part 2, after running the same experiments with both tools:

| | DVC (`dvc exp`) | MLflow (manual logging) |
|---|---|---|
| **How do values get recorded?** | ____________ | ____________ |
| **What is recorded?** | ____________ | ____________ |
| **Where is it stored?** | ____________ | ____________ |
| **How do you compare runs?** | ____________ | ____________ |
| **How do you restore a past run?** | ____________ | ____________ |

## Reflection Questions

1. What does each tool record that the other does not?
2. What goes wrong if a teammate trains on untracked data or uncommitted code?
3. What did MLflow's explicit logging give you, and what did it cost?
4. Why load a model by alias rather than by file path or run ID?
5. Is a better score enough to promote a model? What else would you check?
6. Would you use DVC, MLflow, both or neither in your team project, and why?

## Resources

- [DVC Documentation](https://dvc.org/doc)
- [DVC Pipelines Guide](https://dvc.org/doc/user-guide/pipelines)
- [DVC Experiments Guide](https://dvc.org/doc/user-guide/experiment-management)
- [MLflow Tracking](https://mlflow.org/docs/latest/ml/tracking/)
- [MLflow Model Registry](https://mlflow.org/docs/latest/ml/model-registry/)
- [MLflow Model Serving](https://mlflow.org/docs/latest/ml/deployment/deploy-model-locally/)
