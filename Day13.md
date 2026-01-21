# Advanced Machine Learning Workflows

---

## 1. Training Multiple Models (Model Selection)

In Machine Learning, there is no "one size fits all" algorithm. The standard workflow involves training multiple candidate models to determine which architecture best fits your specific dataset.

### The "Champion-Challenger" Approach

1. **The Baseline (Simple):** Always start with a simple, interpretable model (e.g., Linear Regression or Logistic Regression). This sets the "floor" for performance. If a complex neural network is only 0.1% better than a simple regression, the regression is usually preferred for its speed and interpretability.
2. **The Challenger (Complex):** Train sophisticated models (e.g., Random Forest, Gradient Boosted Trees, XGBoost) to see if capturing non-linear relationships improves metrics significantly.

### Common Model Families

* **Linear Models:** Fast, interpretable, but assume linear relationships. (Good for simple baselines).
* **Tree-Based Models:** (Decision Trees, Random Forests). Can handle non-linear data and interactions automatically. Robust to outliers.
* **Ensemble Methods:** (Gradient Boosting). combine many "weak learners" to build a strong predictor. Often the winners of Kaggle competitions.

---

## 2. Hyperparameter Tuning

Once you select the best algorithm (e.g., Random Forest), you must tune its settings.

* **Parameters:** Learned by the model during training (e.g., the slope  in ).
* **Hyperparameters:** Set *manually* by the data scientist before training (e.g., "Number of Trees" or "Max Depth").

### Tuning Strategies

#### A. Cross-Validation (k-Fold)

To ensure the selected hyperparameters don't just "get lucky" on one specific split of data, we use Cross-Validation.

* Split data into  folds (usually 5).
* Train on 4 folds, test on 1. Repeat 5 times.
* Average the score. This gives a much more reliable estimate of model performance.

#### B. Grid Search

Brute-force testing. You define a grid of values, and the machine tries **every single combination**.

* *Example:* `Trees=[10, 50, 100]` and `Depth=[5, 10]`.
* *Total runs:*  models trained.
* *Pros:* Guaranteed to find the best combo in your grid.
* *Cons:* Computationally expensive.

#### C. Random Search

Instead of trying every combo, randomly select combinations from a distribution.

* *Pros:* Often finds a better model faster than Grid Search because it explores the search space more efficiently.

---

## 3. Spark ML Pipelines

In production, you cannot just "run a model." You must replicate the exact data processing steps (imputing, scaling, encoding) that you did during training. **Pipelines** automate this chain.

### Core Components

1. **DataFrame:** The dataset (Structured data).
2. **Transformer:** An algorithm that transforms one DataFrame into another (e.g., `StringIndexer`, `VectorAssembler`). It has a `.transform()` method.
3. **Estimator:** An algorithm that learns from data to produce a Transformer (e.g., `LogisticRegression`, `StringIndexer`). It has a `.fit()` method.
4. **Pipeline:** Chaining multiple Transformers and Estimators into a single workflow.

### Code Example: PySpark Pipeline

```python
from pyspark.ml import Pipeline
from pyspark.ml.feature import StringIndexer, VectorAssembler
from pyspark.ml.classification import RandomForestClassifier
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
from pyspark.ml.evaluation import BinaryClassificationEvaluator

# 1. Define Stages
# Stage A: Convert string category "color" to numbers
indexer = StringIndexer(inputCol="color", outputCol="color_idx")

# Stage B: Combine all features into a single vector column (Required for Spark ML)
assembler = VectorAssembler(
    inputCols=["color_idx", "age", "salary"], 
    outputCol="features"
)

# Stage C: The Model
rf = RandomForestClassifier(labelCol="label", featuresCol="features")

# 2. Build Pipeline
pipeline = Pipeline(stages=[indexer, assembler, rf])

# 3. Hyperparameter Tuning Setup
paramGrid = ParamGridBuilder() \
    .addGrid(rf.numTrees, [10, 50]) \
    .addGrid(rf.maxDepth, [5, 10]) \
    .build()

crossval = CrossValidator(estimator=pipeline,
                          estimatorParamMaps=paramGrid,
                          evaluator=BinaryClassificationEvaluator(),
                          numFolds=3)

# 4. Train (Fit) the entire pipeline
# This runs indexer -> assembler -> model training -> cross validation
cvModel = crossval.fit(trainingData)

# 5. Predict on new data
predictions = cvModel.transform(testData)

```

---

## 4. Feature Importance

After training a complex model, you need to answer: *"Which features drove the decision?"*

### A. Native Feature Importance (Tree-Based)

Most tree algorithms (Random Forest, XGBoost) calculate importance based on **Gini Impurity** or **Information Gain**.

* *Logic:* "When we split the tree on variable X, how much did it clean up the groups?"
* *Spark Access:* `model.featureImportances`

### B. SHAP (Shapley Additive exPlanations)

The gold standard for interpretability. It is model-agnostic (works for any model).

* *Logic:* It uses game theory to calculate the marginal contribution of each feature to the prediction.
* *Advantage:* It tells you *directionality* (e.g., "High Age lowers the prediction"), whereas native importance only tells you *magnitude* ("Age is important").

### Extracting Importance in PySpark

```python
# Extract the best model from the CrossValidator
best_model = cvModel.bestModel

# The Random Forest is the last stage in our pipeline (Stage index 2)
rf_model = best_model.stages[2]

# Get feature importance vector
importances = rf_model.featureImportances

# Map to column names
import pandas as pd
feature_list = ["color_idx", "age", "salary"]
df_importance = pd.DataFrame(
    list(zip(feature_list, importances)), 
    columns=["Feature", "Importance"]
).sort_values(by="Importance", ascending=False)

print(df_importance)

```