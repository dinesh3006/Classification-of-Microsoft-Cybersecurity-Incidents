# Microsoft: Classifying Cybersecurity Incidents with Machine Learning

This project develops a machine learning model to enhance Security Operation Center (SOC) efficiency by automatically classifying cybersecurity incidents. Using the Microsoft GUIDE dataset, the model predicts the triage grade of an incident as **True Positive (TP)**, **Benign Positive (BP)**, or **False Positive (FP)**.

The primary objective is to create a reliable predictive tool that helps SOC analysts prioritize high-risk incidents, thereby reducing manual triage time and improving overall security posture.

## Project Workflow

The project follows a standard machine learning pipeline from data ingestion to final evaluation.

1.  **Data Loading:** Loaded the `train.csv` (4.7M records) and `test.csv` (4.1M records) datasets.
2.  **Data Preprocessing:**
    * **Handling Missing Data:** Columns with over 50% missing values (e.g., `MitreTechniques`, `ActionGrouped`) were dropped to prevent imputing unreliable data.
    * **Correlation Analysis:** A correlation matrix was used to identify and remove highly-correlated features (e.g., `AccountName`, `FileName`) to prevent multicollinearity.
3.  **Feature Engineering:**
    * The `Timestamp` column was converted into new features (`Day`, `Month`, `Hour`) to capture potential time-based patterns in security incidents.
4.  **Data Encoding:**
    * `LabelEncoder` was applied to all categorical features (`Category`, `IncidentGrade`, `EntityType`, etc.).
    * **Crucial Step:** Each fitted encoder was **saved as a `.pkl` file**. This is a best practice to ensure the *exact same* integer mapping is applied to the test data and any future production data.
5.  **Feature Selection:**
    * `SelectKBest` was used to identify the **top 15 most influential features**, reducing model complexity and improving training speed.
6.  **Model Benchmarking:**
    * The processed training data was split into an 80% training set and a 20% validation set.
    * Several models were trained and evaluated, with XGBoost identified as the top performer.
7.  **Final Evaluation:**
    * The saved XGBoost model (`xgboost_model.pkl`) and the `.pkl` encoders were loaded.
    * The `test.csv` dataset underwent the *exact same* preprocessing, encoding, and feature selection steps.
    * The model's final performance was then calculated on this unseen test set.

## Final Results

### Model Benchmarking (on Validation Set)

The XGBoost Classifier significantly outperformed all other models during the benchmarking phase.

| Model                          | Validation Accuracy |
| :---                           | :---:               |
| **XGBoost (Selected Model)**   | **91.7%**           |
| LGBM                           | 89.5%               |
| Random Forest                  | 77.6%               |
| Decision Tree                  | 77.4%               |
| Logistic Regression (Baseline) | 52.6%               |

### Final Test Set Performance

The saved XGBoost model was evaluated on the final, unseen `test.csv` dataset.

* **Final Accuracy:** **89.3%**
* **Macro-F1 Score:** **0.89**

#### Classification Report (Test Set)

| Class               | Precision | Recall   | F1-Score |
| :---                | :---:     | :---:    | :---:    |
| 0 (BenignPositive)  | 0.87      | 0.93     | 0.90     |
| 1 (FalsePositive)   | 0.88      | 0.81     | 0.85     |
| 2 (TruePositive)    | 0.93      | 0.90     | 0.91     |
| | | | |
| **Macro Avg**       | **0.89**  | **0.88** | **0.89** |
| **Weighted Avg**    | **0.89**  | **0.89** | **0.89** |

* **Handling Class Imbalance:** The dataset is imbalanced. Future iterations should use techniques like `stratify=y` during the `train_test_split` or apply class weighting (e.g., `scale_pos_weight` in XGBoost) to improve recall for the minority "FalsePositive" class.
