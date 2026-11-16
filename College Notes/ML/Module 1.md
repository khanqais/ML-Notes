---
title: Machine Learning Exam Prep - Comprehensive Review
date: 2026-11-14
tags: [machine-learning, exam-prep, university, review]
---

# 🧠 Machine Learning Exam Revision

> [!summary] Overview
> This note covers key [[Machine Learning]] concepts based on exam questions, including core challenges, the ML development lifecycle, business applications, and algorithm selection criteria.

---

## 📌 Q1: What are the issues of Machine Learning?

> [!quote] Core Definition
> Machine learning is the process of training a model to predict unseen outcomes by learning patterns, **without being explicitly programmed**.

### Primary Challenges
*   **[[Data Bias]] & Imbalanced Datasets:** If trained on skewed data, the model's predictions will be unfair or perform poorly on minority groups.
*   **Poor Generalization (Data Mismatch):** When training data is too simple, the model fails to handle complex, real-world queries. Related to **Underfitting**.
*   **Data Scarcity & Cost of Labeling:** Supervised learning requires massive amounts of annotated data, which is time-consuming and expensive to create. 

### Additional Challenges
*   **[[Overfitting]]:** The model memorizes training noise rather than general patterns, failing on unseen data.
*   **Poor Data Quality:** Missing values, outliers, or noisy data lead to "Garbage In, Garbage Out."
*   **The "Black Box" Problem (Interpretability):** Complex models (like Deep Neural Networks) cannot easily explain *how* they arrived at a decision.
*   **High Computational Cost:** Training requires massive processing power, memory, and specialized hardware (GPUs).

---

## 📌 Q2: Steps of Developing Machine Learning Applications

> [!info] The 7-Step ML Development Lifecycle

1.  **Problem Definition:** Define the objective and determine if it requires [[Supervised Learning]], [[Unsupervised Learning]], or [[Reinforcement Learning]].
2.  **Data Collection (Acquisition):** Gather raw data from databases, APIs, or public datasets.
3.  **Data Preprocessing & Feature Engineering:**
    *   *Cleaning:* Handle missing values (Imputation), remove duplicates/outliers.
    *   *Encoding:* Convert text to numbers.
    *   *Feature Engineering:* Create new, useful columns from existing data.
4.  **==Data Splitting (Crucial)==:** Divide data into a **Training Set** (70-80%) to teach the model, and a **Testing Set** (20-30%) to evaluate it.
5.  **Model Selection & Training:** Choose an algorithm (e.g., [[Linear Regression]], [[Decision Trees]]) and feed the Training Set into it to learn patterns.
6.  **Model Evaluation:** Test the model using the unseen Testing Set.
    *   *Classification metrics:* Accuracy, Precision, Recall, F1-Score.
    *   *Regression metrics:* Mean Squared Error (MSE), R-Squared.
7.  **Hyperparameter Tuning & Deployment:** Adjust internal settings to improve accuracy, then deploy to a real-world application.

---

## 📌 Q3: Business Applications of Machine Learning

> [!example] 8 Real-World Business Use Cases

1.  **Algorithmic Trading:** High-frequency trading (HFT) models predict stock movements based on historical data and market sentiment.
2.  **Supply Chain & Logistics Optimization:** Predicting weather and traffic disruptions to reroute fleets and save fuel costs.
3.  **Customer Churn Prediction:** Analyzing behavior to flag users likely to cancel subscriptions, allowing for targeted retention discounts.
4.  **Recommendation Engines:** Driving revenue by suggesting products/media based on past behavior and collaborative filtering (e.g., Amazon, Netflix).
5.  **Fraud Detection:** Instantly blocking anomalous financial transactions based on historical fraud patterns.
6.  **Customer Service Automation ([[NLP]]):** Resolving repetitive customer queries via 24/7 intelligent chatbots, reducing operational overhead.
7.  **Dynamic Pricing Optimization:** Adjusting prices in real-time based on demand, competitors, and inventory to maximize profit (e.g., Uber, Airlines).
8.  **Predictive Maintenance:** Analyzing IoT sensor data from heavy machinery to predict failures before they happen, avoiding costly factory downtime.

---

## 📌 Q4: Email Spam Detection & Choosing the Right Algorithm

### Spam Detection Use Case
*   **Technique:** [[Supervised Learning]] -> **Binary [[Classification]]**
*   **Why:** The model learns from historical data labeled as "Spam" (1) or "Not Spam" (0).
*   **How it Works:** Uses [[Natural Language Processing]] (NLP) to extract features (word frequencies like "Free", "Urgent").
*   **Best Algorithm:** **[[Naive Bayes]]**, as it calculates the probability of a category based on the specific text/words present.

### How to Choose the Right ML Algorithm
> [!tip] The No Free Lunch Theorem
> There is no single "best" algorithm. Selection depends on evaluating the following 5 criteria:

# Choosing a Machine Learning Algorithm

Choosing an ML algorithm depends on:
1. Problem type
2. Size and quality of data
3. Interpretability requirement
4. Training time and computational cost
5. Nature/linearity of data
6. Performance metrics and business constraints

There is no universally best algorithm. Start with a simple baseline, evaluate it using validation data, then increase complexity only if needed.

---

## 1. Determine the Problem Type

The first step is to identify whether the task is supervised or unsupervised, and whether the output is a number or a category.

### A. Labeled data + Predicting a number → Regression

Use **regression** when the target variable is continuous/numerical.

Examples:
- Predicting house price
- Predicting salary
- Predicting temperature
- Predicting sales revenue
- Predicting delivery time

| Situation | Suitable algorithm | Why |
|---|---|---|
| Simple linear relationship | Linear Regression | Easy to train, interpretable, works when output changes approximately linearly with input |
| Multiple correlated features | Ridge Regression | Adds L2 penalty, reduces overfitting caused by multicollinearity |
| Many irrelevant features / feature selection needed | Lasso Regression | Adds L1 penalty and can make some coefficients zero |
| Mixed feature selection + regularization | Elastic Net Regression | Combines L1 and L2 regularization |
| Non-linear relationship | Polynomial Regression | Fits curved relationships, but can overfit if degree is too high |
| Non-linear data with medium-sized dataset | Decision Tree Regressor | Captures non-linear patterns and feature interactions |
| Strong tabular data performance | Random Forest Regressor | Reduces variance through multiple decision trees |
| High accuracy on structured/tabular data | XGBoost / LightGBM / CatBoost | Gradient boosting usually performs very well on tabular data |
| Very large dataset | SGD Regressor | Efficient for large-scale linear regression |
| Complex patterns, images/text/time series | Neural Network | Learns complex representations, but needs more data and compute |

**Exam line:**  
Regression is used when the input data is labeled and the output is a continuous numerical value, such as predicting house prices.

---

### B. Labeled data + Predicting a category → Classification

Use **classification** when the target variable is discrete/categorical.

Examples:
- Spam vs. ham email
- Fraud vs. genuine transaction
- Disease positive vs. negative
- Customer churn: yes/no
- Image classification: cat/dog/car
- Sentiment: positive/negative/neutral

| Situation | Suitable algorithm | Why |
|---|---|---|
| Text classification, small/medium data | Naive Bayes | Fast, simple, works well with word-frequency/TF-IDF features |
| Binary classification with probabilities | Logistic Regression | Simple, interpretable, outputs probability |
| Linearly separable data | Logistic Regression / Linear SVM | Creates a clear decision boundary |
| Non-linear classification | SVM with RBF kernel | Can learn complex non-linear boundaries |
| Small-to-medium tabular dataset | Decision Tree | Easy to understand and visualize |
| Robust model with less tuning | Random Forest | Combines many trees, reduces overfitting |
| High accuracy on tabular data | XGBoost / LightGBM / CatBoost | Strong gradient-boosting algorithms |
| Very large dataset | SGD Classifier | Efficient for large-scale linear classification |
| Image, speech, text with large data | CNN / RNN / Transformer | Deep learning captures complex patterns |
| Need probability output | Logistic Regression, Naive Bayes, Neural Network with softmax | These can output class probabilities |

**Email spam detection:**  
Use **Naive Bayes + TF-IDF** as a simple baseline. For better accuracy, use **Logistic Regression** or **Linear SVM**. For deep learning, use a transformer-based model only when sufficient labeled data and compute are available.

**Exam line:**  
Classification is used when the input data is labeled and the output is a discrete category, such as spam or ham.

---

### C. Unlabeled data + Finding groups → Clustering

Use **clustering** when data has no labels and the goal is to discover natural groups.

Examples:
- Customer segmentation
- Document grouping
- Image grouping
- Anomaly detection
- Market segmentation

| Situation | Suitable algorithm | Why |
|---|---|---|
| Spherical, similarly sized clusters | K-Means | Simple, fast, scalable |
| Unknown or varying cluster shapes | DBSCAN | Finds arbitrary-shaped clusters and detects outliers |
| Hierarchical grouping needed | Hierarchical Clustering | Creates a tree/dendrogram of clusters |
| Overlapping probabilistic clusters | Gaussian Mixture Model | Assigns soft probabilities instead of hard clusters |
| Very large dataset | Mini-Batch K-Means | Faster and more memory-efficient than standard K-Means |
| High-dimensional sparse text | K-Means on TF-IDF / embeddings | Common approach for document clustering |

**Exam line:**  
Clustering is an unsupervised learning technique used when data is unlabeled and the goal is to group similar data points together.

---

## 2. Size and Quality of Data

The amount and quality of data strongly affect model choice.

### Small dataset

Small data usually means limited samples, noisy labels, or few examples per class.

Use simple, low-variance models:
- Linear Regression
- Logistic Regression
- Naive Bayes
- k-NN
- Decision Tree with pruning
- Regularized SVM

**Why:**  
Complex models have many parameters and can memorize the small training dataset, causing overfitting.

**Example:**  
For spam detection with only 1,000 labeled emails, start with Naive Bayes or Logistic Regression using TF-IDF.

### Medium dataset

Use models that can capture non-linear patterns:
- Random Forest
- Gradient Boosting
- SVM
- Neural Network with regularization

**Example:**  
For customer churn prediction with 50,000 records, try Logistic Regression first, then Random Forest or XGBoost.

### Massive dataset

Large datasets can support more complex models:
- Deep Neural Networks
- CNNs for images
- Transformers for text
- Gradient Boosting for tabular data
- SGD-based models for scalable linear models

**Why:**  
Deep learning needs many examples to learn meaningful patterns and avoid overfitting.

**Example:**  
For image classification with millions of labeled images, use CNNs or transfer learning with pretrained models.

### Poor-quality data

If data has missing values, noise, imbalance, or incorrect labels:
- Clean and preprocess the data first
- Handle missing values
- Remove duplicates
- Treat outliers carefully
- Balance classes using resampling or class weights
- Use robust models such as Random Forest or Gradient Boosting

**Important:**  
A complex model cannot fix bad data. Data quality often matters more than algorithm choice.

---

## 3. Interpretability

Interpretability means the ability to explain why a model made a prediction.

Use interpretable models when decisions affect people, money, health, safety, or legal compliance.

Examples:
- Loan approval
- Medical diagnosis
- Insurance risk
- Fraud detection
- Hiring decisions
- Credit scoring

| Requirement | Suitable algorithms |
|---|---|
| High interpretability | Linear Regression, Logistic Regression, Decision Tree |
| Medium interpretability | Random Forest with feature importance, Gradient Boosting with SHAP |
| Low interpretability | Deep Neural Networks, complex ensembles |

### Interpretable models

- **Linear Regression:** Coefficients show how each feature affects the prediction.
- **Logistic Regression:** Coefficients show how each feature affects the probability of a class.
- **Decision Tree:** Rules can be followed easily, such as “If income > X and credit score > Y, approve loan.”

### Black-box models

- Deep Neural Networks
- Large ensembles
- Complex SVMs
- Transformer models

These may provide higher accuracy but are harder to explain.

**Solution for black-box models:**  
Use explanation techniques such as:
- Feature importance
- SHAP values
- LIME
- Partial dependence plots

**Exam line:**  
If explainability is important, choose interpretable models such as Linear Regression, Logistic Regression, or Decision Trees. If accuracy is more important and explanations are not required, complex models such as Neural Networks or ensemble methods may be used.

---

## 4. Training Time and Computational Cost

Choose an algorithm based on available hardware, latency requirements, and retraining frequency.

| Requirement | Suitable algorithms |
|---|---|
| Fast training on CPU | Logistic Regression, Naive Bayes, Linear SVM, Decision Tree |
| Fast prediction/low latency | Logistic Regression, Naive Bayes, Decision Tree, small tree ensembles |
| Frequent retraining | SGD Classifier, SGD Regressor, Online Learning models |
| Large-scale data | SGD, Mini-Batch K-Means, LightGBM |
| High accuracy on tabular data | XGBoost, LightGBM, CatBoost |
| GPU available and large data | Deep Neural Networks, CNN, Transformer |
| Real-time inference on mobile/edge | Small models, quantized models, decision trees, lightweight neural networks |

**Example:**  
For spam detection, the model may need frequent retraining as spammers change their tactics. Therefore, Logistic Regression, Naive Bayes, or SGD-based models may be more practical than a heavy deep-learning model.

**Exam line:**  
If the model must be trained frequently or deployed on limited hardware, choose simple and computationally efficient algorithms. Deep learning should be used when sufficient data, GPU resources, and acceptable latency are available.

---

## 5. Linearity of the Data

Check whether the relationship between input features and output is linear or non-linear.

### Linear relationship

Use:
- Linear Regression
- Logistic Regression
- Linear SVM
- Linear models with regularization

**Example:**  
If house price increases approximately proportionally with house area, Linear Regression is suitable.

### Non-linear relationship

Use:
- Polynomial Regression
- Decision Trees
- Random Forest
- Gradient Boosting
- Kernel SVM
- Neural Networks

**Example:**  
If customer purchase behavior depends on complex interactions between age, income, location, and browsing history, tree-based models or neural networks may perform better.

### Text data

Use:
- TF-IDF + Naive Bayes
- TF-IDF + Logistic Regression
- TF-IDF + Linear SVM
- Word embeddings + Neural Network
- Transformer models

**Example:**  
For spam detection, TF-IDF converts email text into numerical features. Then Naive Bayes, Logistic Regression, or Linear SVM can classify the email.

### Image data

Use:
- CNN
- Transfer learning with pretrained CNN
- Vision Transformer for large-scale tasks

### Time-series data

Use:
- ARIMA for simple statistical forecasting
- Prophet for trend/seasonality
- LSTM/GRU for sequential dependencies
- Transformer-based models for advanced sequence modeling
---


---

## Quick Decision Table

| Problem                   | Data type                                                        | Recommended starting algorithm                |                                               |         |
| ------------------------- | ---------------------------------------------------------------- | --------------------------------------------- | --------------------------------------------- | ------- |
| Predict house price       | Numerical target                                                 | Linear Regression                             |                                               |         |
| Predict salary            | Numerical target                                                 | Linear Regression / Random Forest Regressor   |                                               |         |
| Spam detection            | Text, binary label                                               | Naive Bayes + TF-IDF, Logistic Regression     |                                               |         |
| Fraud detection           | Tabular, imbalanced                                              | Logistic Regression, XGBoost, class weighting |                                               |         |
| Customer churn            | Tabular, binary label                                            | Logistic Regression, Random Forest, XGBoost   |                                               |         |
| Disease diagnosis         | Tabular/medical data                                             | Logistic Regression, Decision                 | Task                                          | Metrics |
| Regression                | MAE, MSE, RMSE, R-squared                                        |                                               |                                               |         |
| Binary classification     | Accuracy, Precision, Recall, F1-score, ROC-AUC, Confusion Matrix |                                               |                                               |         |
| Multiclass classification | Accuracy, Macro F1, Weighted F1, Confusion Matrix                |                                               |                                               |         |
| Clustering                | Silhouette Score, Davies-Bouldin Index, Inertia                  |                                               |                                               |         |
| Imbalanced classification | Precision, Recall, F1-score, PR-AUC                              | ion                                           | Collaborative Filtering, Matrix Factorization |         |

---

## Final Exam Answer

> The selection of a machine-learning algorithm depends on the problem type, dataset size and quality, interpretability requirement, computational cost, and the nature of the data. If labeled data is used to predict a continuous value, regression algorithms such as Linear Regression, Ridge, Lasso, Random Forest Regressor, or XGBoost are suitable. If labeled data is used to predict a category, classification algorithms such as Logistic Regression, Naive Bayes, Decision Tree, Random Forest, SVM, or XGBoost are suitable. If data is unlabeled and groups must be discovered, clustering algorithms such as K-Means, DBSCAN, or Hierarchical Clustering are used.  
>
> For small datasets, simple models such as Naive Bayes, Logistic Regression, and Linear Regression are preferred to avoid overfitting. For large datasets, complex models such as Random Forest, Gradient Boosting, and Deep Neural Networks can be used. If interpretability is important, Linear Regression, Logistic Regression, and Decision Trees are preferred. If only accuracy matters, ensemble methods or neural networks can be used. For linear relationships, linear models are suitable, while non-linear relationships require Polynomial Regression, Kernel SVM, tree-based models, or Neural Networks. Finally, the model should be evaluated using validation data and appropriate metrics before deployment.

Explain the terms overfitting, underfitting, bias & variance tradeoff w.r.t. Machine Learning
Write how should be writing in exam

**Overfitting** and **underfitting** describe how well a machine-learning model generalizes to unseen data. The **bias–variance tradeoff** explains the balance between a model being too simple (high bias) and too complex (high variance); the goal is to choose a model complexity that minimizes test/generalization error.[1][2]

## Definitions

### Overfitting

**Overfitting** occurs when a model learns the training data too well, including its random noise and outliers. It gives very high accuracy on training data but performs poorly on new/test data because it fails to generalize.[1][3]

**Example:** If a spam-detection model memorizes exact phrases from the training emails instead of learning general spam patterns, it may classify new spam emails incorrectly.

**Symptoms:**
- Very low training error.
- High testing/validation error.
- Large gap between training and test performance.[1][2]

**Causes:**
- Model is too complex.
- Too many features or parameters.
- Insufficient training data.
- Training for too many epochs.
- Noise present in the dataset.[4][5]

**Solutions:**
- Use regularization such as L1/L2 regularization.
- Use cross-validation.
- Reduce model complexity.
- Add more training data.
- Use early stopping.
- Remove irrelevant features.
- Use dropout in neural networks.[2][6]

### Underfitting

**Underfitting** occurs when a model is too simple to learn the underlying pattern in the data. It performs poorly on both training and test data.[1][3]

**Example:** If a spam classifier uses only one feature, such as the number of capital letters, it may not capture enough information to distinguish spam from ham.

**Symptoms:**
- High training error.
- High testing/validation error.
- Model cannot capture important relationships in the data.[1][2]

**Causes:**
- Model is too simple.
- Too few features.
- Insufficient training time.
- Incorrect assumptions about the data.
- Excessive regularization.[4][6]

**Solutions:**
- Use a more complex model.
- Add useful features.
- Train for more epochs.
- Reduce regularization.
- Improve data quality and quantity.[6][7]

## Bias and variance

### Bias

**Bias** is the error caused by overly simplified assumptions made by the model. A high-bias model ignores important patterns in the data and usually leads to **underfitting**.[1][2]

**Example:** Using a straight line to fit curved data creates high bias because the model cannot represent the curve.

### Variance

**Variance** is the sensitivity of a model to changes in the training data. A high-variance model changes greatly when trained on different samples and usually leads to **overfitting**.[1][2]

**Example:** A very high-degree polynomial may pass through almost every training point, but its predictions on new data may vary significantly.

## Bias–variance tradeoff

The **bias–variance tradeoff** means that increasing model complexity generally reduces bias but increases variance, while decreasing complexity increases bias but reduces variance. A good model balances both to achieve low error on unseen data.[2][8]

| Model condition | Bias | Variance | Training performance | Test performance | Problem |
|---|---:|---:|---|---|---|
| Too simple | High | Low | Poor | Poor | Underfitting |
| Balanced | Low | Low | Good | Good | Best generalization |
| Too complex | Low | High | Very good | Poor | Overfitting |

The total expected prediction error can be viewed as:

$$
\text{Total Error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible Error}
$$

Here, **irreducible error** is noise that cannot be removed by any model.[2][9]

## How to detect them

Use a **train-validation/test split** or **cross-validation** and compare performance:

- **Underfitting:** Low training score and low validation score.
- **Overfitting:** High training score but low validation score.
- **Good fit:** High training score and high validation score, with a small gap between them.[2][10]

