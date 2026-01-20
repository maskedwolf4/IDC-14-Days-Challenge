# MLflow: Managing the Machine Learning Lifecycle


## 1. MLflow Components

MLflow is organized into four primary components, though most Data Scientists spend the majority of their time in **Tracking** and **Registry**.

### A. MLflow Tracking

The "Lab Notebook" for data scientists. It records and queries experiments: code, data, config, and results.

* **Goal:** "What parameters did I use to get that 98% accuracy last Tuesday?"

### B. MLflow Models

A standard format for packaging machine learning models that can be used in a variety of downstream tools.

* **The "Flavors" Concept:** MLflow saves models in a generic format that allows different tools (Docker, Spark, REST API) to understand them without knowing the underlying library (Scikit-Learn, TensorFlow, PyTorch).

### C. MLflow Model Registry

A centralized model store, API, and UI to collaboratively manage the full lifecycle of an MLflow Model.

* **Goal:** Version control for models. It manages transitions from `Staging`  `Production`  `Archived`.

### D. MLflow Projects (Less Common)

A format for packaging data science code in a reusable and reproducible way (usually via a `MLproject` file and Conda environment).

---

## 2. Experiment Tracking

In MLflow, an **Experiment** is a logical group of **Runs**.

* **Experiment:** "Customer Churn Prediction Q1"
* **Run:** A single execution of your training script.

### What is tracked in a Run?

1. **Parameters:** Key-value pairs of input settings (e.g., `learning_rate=0.01`, `n_estimators=100`).
2. **Metrics:** Key-value pairs of numeric results that change over time (e.g., `accuracy`, `rmse`, `loss`).
3. **Tags:** Metadata for organization (e.g., `developer="john_doe"`, `environment="dev"`).
4. **Artifacts:** Output files (e.g., the saved model file, confusion matrix images, `requirements.txt`).

### Code Example: Manual Tracking

```python
import mlflow

# 1. Set the Experiment Name
mlflow.set_experiment("/Users/me/Churn_Prediction")

# 2. Start the Run
with mlflow.start_run(run_name="RandomForest_v1"):
    
    # 3. Log Parameters (Inputs)
    params = {"n_estimators": 100, "max_depth": 5}
    mlflow.log_params(params)
    
    # ... Train your model here ...
    
    # 4. Log Metrics (Outputs)
    mlflow.log_metric("accuracy", 0.89)
    mlflow.log_metric("auc_score", 0.92)
    
    # 5. Log Artifacts (Files)
    with open("feature_importance.txt", "w") as f:
        f.write("Age, Income, Tenure")
    mlflow.log_artifact("feature_importance.txt")

```

---

## 3. Model Logging

Logging a model saves the actual binary object (pickle file, H5 file) alongside the run.

### Autologging (The Magic Button)

Most modern libraries (Scikit-Learn, Keras, PyTorch, XGBoost, LightGBM) support **Autologging**. You write one line of code, and MLflow automatically captures *everything*: parameters, metrics, and the model artifact.

```python
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier

# Enable Autologging
mlflow.sklearn.autolog()

with mlflow.start_run():
    clf = RandomForestClassifier(n_estimators=100)
    clf.fit(X_train, y_train)
    # No need to manually log params or metrics!

```

### Manual Model Logging

If you need custom control, you use `log_model`.

```python
# Save the model so it can be loaded as a generic Python function later
mlflow.sklearn.log_model(
    sk_model=clf, 
    artifact_path="my_model_folder",
    registered_model_name="Churn_Predictor" # Optional: Registers to Registry immediately
)

```

---

## 4. The MLflow UI

The UI is where you visualize and compare your experiments.

### Key Features

1. **Experiment List:** Sidebar showing all your projects.
2. **Run Comparison Table:** A spreadsheet view of every run. You can sort by `accuracy DESC` to instantly find the best model.
3. **Parallel Coordinates Plot:** A visualization that helps you see correlations between hyperparameters and metrics (e.g., "High `max_depth` usually leads to low `accuracy`").
4. **Artifact Viewer:** Allows you to browse the files (images, model binaries) associated with a run without touching the command line.

---

## 5. The Model Registry Workflow

The Registry is the bridge between "Data Science" and "DevOps".

### Lifecycle Stages

1. **None:** The model is registered but not assigned a stage.
2. **Staging:** The model is currently being tested (e.g., A/B testing or QA).
3. **Production:** The live version serving traffic.
4. **Archived:** Old versions kept for audit purposes.

### The Deployment Pattern

The most powerful feature of the Registry is that deployment code doesn't need to change when you retrain the model.

**Bad (Hardcoded Path):**
`model = load("/dbfs/mnt/models/churn/run_id_12345/model")`
*(If you retrain, you must update the code).*

**Good (Registry Reference):**
`model = load("models:/Churn_Predictor/Production")`
*(The code always pulls whatever model is currently tagged "Production". When Data Science promotes v2 to Production, the app updates automatically).*