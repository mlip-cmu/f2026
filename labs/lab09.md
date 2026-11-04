# Lab 9: Data, Pipeline & Model Versioning with DVC and MLflow

In this lab, you'll learn how to version data and pipelines with DVC, track experiments with MLflow, and serve and promote models through a model registry. You'll connect the two tools so that a registered model can be traced back to the exact code and data that produced it. By the end of the lab, you'll be able to answer a question every ML team eventually faces: where did this model come from, and should it be in production?

Clone [this starter repository](https://github.com/aku134/mlip-lab-9), which contains a pipeline that trains a Random Forest on the Breast Cancer Wisconsin dataset to classify tumors as malignant or benign.

## Deliverables

- [ ] **Deliverable 1:** Show the TA your DVC pipeline DAG, your DVC experiment comparison, and at least 4 complete MLflow runs. Using your completed [comparison table](#comparison-table), explain what each tool records and when you would use each one.

- [ ] **Deliverable 2:** Demonstrate to the TA two models served side by side and the `compare_serving.py` output. Explain your decision on whether to promote the challenger, and what a running server needs in order to pick up an alias change.

- [ ] **Deliverable 3:** Demonstrate to the TA a rebuilt run whose metrics exactly match version 1 of your registered model. Explain how you got from the registry to the exact code and data, and what would have broken this trace.

## Prerequisite: Setup

You need Git and Python 3.10+. Check with `python3 --version` before creating the venv (the macOS system Python is too old). If you hit setup issues, use your team's VM.

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

## Task 1: Version Data and Pipelines with DVC

Put the dataset under DVC and set up a local remote. Use `scripts/augment_data.py` to create a second version of the data, then switch back to the original. Run the pipeline defined in `dvc.yaml`, view its DAG, and see which stages re-run when something changes. Finally, run experiments with `max_depth` set to 1, 2 and 5, and compare them.

## Task 2: Track Experiments with MLflow

Runs from before this task appear empty in the MLflow UI, because nothing is logged until you add the code. Complete the six TODOs in `scripts/train.py` and `scripts/evaluate.py` to log params, lineage tags, the model, metrics and the confusion matrix, all in one run.

Run the same three experiments, plus 1–2 of your own. For each one, edit `params.yaml`, commit, then run `dvc repro`. Don't use `dvc exp run` here: each run is tagged with the current git commit, so the run must match a commit. Compare the runs in the MLflow UI, choose a best model, and fill in the comparison table:

### Comparison Table

| | DVC (`dvc exp`) | MLflow (manual logging) |
|---|---|---|
| **How do values get recorded?** | ____________ | ____________ |
| **What is recorded?** | ____________ | ____________ |
| **Where is it stored?** | ____________ | ____________ |
| **How do you compare runs?** | ____________ | ____________ |
| **How do you restore a past run?** | ____________ | ____________ |

## Task 3: Serve and Promote a Model

Register the `max_depth=1` model and your best model as versions of `breast-cancer-classifier`, with the aliases `champion` and `challenger`. Load the champion with `scripts/predict.py`.

Serve the champion on port 5002 and the challenger on port 5003 (use `--env-manager local` with `mlflow models serve`), then compare them with `scripts/compare_serving.py`. Decide whether to promote the challenger. If you do, move the alias and find out what it takes for a running server to pick up the change.

## Task 4: Trace and Reproduce a Model

Rebuild version 1 of the registered model from scratch, starting from the registry and using only what its run recorded. Confirm that the new run's metrics exactly match the original.

## Reflection Questions

The TA may also ask you about any of these:

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
