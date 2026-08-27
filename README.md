# End-to-End Machine Learning Project Architecture

![System Architecture](watermarked_img_11169013272961002435.png)

This repository contains a complete, production-ready Machine Learning pipeline, transitioning from Exploratory Data Analysis (EDA) in Jupyter Notebooks to a fully modular, deployable system.

---

## Step-by-Step Implementation Guide

### 1. Environment Setup & Version Control

Initialize your repository and isolate dependencies using a virtual environment.

```bash
git init
conda create -p venv python=3.8 -y
conda activate ./venv
```

_Create a `.gitignore` to prevent pushing virtual environments and large datasets to GitHub._

### 2. Project Packaging

Create `requirements.txt` for dependencies and `setup.py` to build your project as a Python package. Add `-e .` to the end of `requirements.txt` to trigger `setup.py` automatically.

### 3. Core Directory Structure

Organize your project into a modular architecture:

```text
├── artifacts/            # Output data and pickled models
├── notebooks/            # Jupyter notebooks for EDA and prototyping
├── src/
│   ├── components/       # Core pipeline modules
│   │   ├── data_ingestion.py
│   │   ├── data_transformation.py
│   │   └── model_trainer.py
│   ├── pipeline/         # High-level pipelines
│   │   ├── train_pipeline.py
│   │   └── predict_pipeline.py
│   ├── exception.py      # Custom exception handling
│   ├── logger.py         # Runtime execution logging
│   └── utils.py          # Helper functions (save/load objects)
├── app.py                # Flask Web Application
├── requirements.txt
└── setup.py
```

### 4. Exception Handling & Logging

- **`exception.py`**: Extend the base exception class using the `sys` module to capture exact file names and line numbers for errors.
- **`logger.py`**: Configure Python's `logging` module to maintain daily `.log` files tracking the execution pipeline.

### 5. Data Ingestion (`data_ingestion.py`)

- Define configurations (e.g., `DataIngestionConfig`) for output paths.
- Read raw data from the source (CSV, MongoDB, API).
- Perform a Train/Test split (`train_test_split()`).
- Save `train.csv` and `test.csv` to the `artifacts/` folder.

### 6. Data Transformation (`data_transformation.py`)

- Create **Scikit-Learn Pipelines** for numerical and categorical features.
    - _Numerical_: `SimpleImputer` (median) ➔ `StandardScaler`.
    - _Categorical_: `SimpleImputer` (mode) ➔ `OneHotEncoder` ➔ `StandardScaler`.
- Combine them using a `ColumnTransformer`.
- Apply `fit_transform()` on training data and `transform()` on test data.
- Save the transformer object as `preprocessor.pkl`.

### 7. Model Training & Tuning (`model_trainer.py`)

- Initialize multiple regression/classification algorithms (Random Forest, XGBoost, Linear Regression, etc.).
- Use `GridSearchCV` or `RandomizedSearchCV` in `utils.py` to find the best hyperparameters.
- Evaluate model performance using metrics like R² Score, MAE, or RMSE.
- Save the highest-performing model as `model.pkl`.

### 8. Prediction Pipeline (`predict_pipeline.py`)

- Create a custom data class to map incoming web form inputs to pipeline features.
- Load `preprocessor.pkl` to scale and encode the new data.
- Pass the transformed data to `model.pkl` to fetch predictions.

### 9. Web Application Integration (`app.py`)

- Build a **Flask** API with routes for rendering the UI and handling predictions (e.g., `@app.route('/predict', methods=['GET', 'POST'])`).
- Connect HTML form inputs (`home.html`) to the `predict_pipeline`.

### 10. CI/CD Deployment (AWS Elastic Beanstalk)

- Create an `.ebextensions/python.config` file to set the WSGI path to your Flask app.
- Initialize an **AWS Elastic Beanstalk** Python environment.
- Use **AWS CodePipeline** to connect your GitHub repository to Elastic Beanstalk.
- Any pushes to the `main` branch will automatically trigger a new build and deployment.
