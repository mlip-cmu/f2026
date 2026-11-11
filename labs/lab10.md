# Lab 10: Explainability with Alibi & SHAP
In this lab, you'll learn about the Alibi Explain and SHAP libraries, and implement both global and local explanations of tabular and image classification models.
Alibi Explain is an open-source Python library for machine learning model inspection and interpretation. It provides high-quality implementations of black-box, white-box, local, and global explanation methods for classification and regression models.
SHAP (SHapley Additive exPlanations) is based on game theory and helps quantify the contribution of each feature to a model's prediction, offering both global and instance-level interpretability.
Complete all the Deliverables mentioned below and show them to a TA for credit.
 
## Notebook structure
- Section 1: Data and Model Setup
- Section 2: Explainability with Alibi
  - 2.1 Partial Dependence Variance (feature importance)
  - 2.2 Partial Dependence plots
  - 2.3 Anchor explanations on images
- Section 3: SHAP Comparison
- Section 4: Reflection and Discussion
## Deliverables
- Finish all the TODOs in Section 2 of the notebook
- Generate Partial Dependence (PD) plots and discuss your findings with the TA. Answer:
     - Q1: Which features have higher importance in the prediction? (Section 2.1)
     - Q2: What can you conclude from the PD plots? (Section 2.2)
     - Q3: Discuss the two-feature PD results (heatmaps) with the TA. (Section 2.2)
- Run Anchor explanations on the image data, then try at least one different parameter setting (e.g. `n_segments` or `threshold`). Show your results and discuss the parameters and findings with the TA. (Section 2.3)
- Complete the SHAP Comparison and Reflection sections:
     - Q4: Which features have the strongest SHAP impact, and do they match the PD findings? (Section 3)
     - Q5: In 2 to 3 sentences, summarize what your model relies on and whether any features raise fairness or bias concerns. (Section 4)
NOTE: You only need to write short answers. Most cells can run directly.
 
## Getting Started
Clone [this](https://github.com/firefrost91/mlip-explainability-lab) repository and follow the instructions in the notebook.
 
### Install Dependencies
**Use Python 3.9 to 3.11.** The pinned versions (`tensorflow<2.15`, `numpy==1.23.5`) do not install on Python 3.12 or newer.
```
conda create -n lab10 python=3.11
conda activate lab10
pip install -r requirements.txt
```
_(If you run into major unsolvable issues, run the notebook on Google Colab instead.)_
 
**Heads up on runtime:** each Anchor explanation runs InceptionV3 on thousands of perturbed images, so it takes about 5 to 7 minutes per image on a laptop CPU. That's normal, not a hang. Colab with a GPU is much faster.
 
### Possible Issues with installing Alibi
`TypeError: issubclass() arg 1 must be a class` \
**Solution:** https://stackoverflow.com/questions/76313592/import-langchain-error-typeerror-issubclass-arg-1-must-be-a-class
<br><br>
If you hit any other issues, please contact a TA so we can add them to this list (with a solution if it's solved).
 
## References
1. https://docs.seldon.io/projects/alibi/en/stable/examples/pdp_regression_bike.html
2. https://docs.seldon.io/projects/alibi/en/stable/methods/PartialDependence.html
3. https://docs.seldon.io/projects/alibi/en/stable/examples/anchor_image_imagenet.html
4. https://shap.readthedocs.io/en/latest/
