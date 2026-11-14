---
tags: [machine-learning, classification, metrics, evaluation]
aliases: [Classification Metrics, Performance Metrics, Model Evaluation]
---

# Performance Metrics for Classification



---

## 1. Confusion Matrix

A **confusion matrix** compares actual labels with predicted labels. For binary classification, it has four outcomes:

| Actual \ Predicted | Predicted Positive | Predicted Negative |
|---|---:|---:|
| **Actual Positive** | True Positive (TP) | False Negative (FN) |
| **Actual Negative** | False Positive (FP) | True Negative (TN) |

- **TP:** Model correctly predicts positive.
- **TN:** Model correctly predicts negative.
- **FP:** Model incorrectly predicts positive (**Type I error**).
- **FN:** Model incorrectly predicts negative (**Type II error**).

> [!example] Spam classifier
> - **TP:** spam correctly marked as spam
> - **FP:** normal email wrongly marked as spam
> - **FN:** spam wrongly marked as normal
> - **TN:** normal email correctly marked as normal

---

## 2. Accuracy

**Accuracy** is the proportion of all predictions that are correct.

$$
\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
$$

It is simple and useful when both classes are roughly balanced and all errors have similar cost.

> [!warning] Misleading on imbalanced data
> If 99% of emails are non-spam, a model that predicts "not spam" for everything gets **99% accuracy** while detecting **no spam at all**.

**Use accuracy when:** the class distribution is balanced and both error types matter equally.

---

## 3. Precision

**Precision** answers: *Of all samples predicted positive, how many were actually positive?*

$$
\text{Precision} = \frac{TP}{TP + FP}
$$

It measures how trustworthy positive predictions are. High precision means the model makes few false-positive mistakes.

> [!tip] Use precision when false positives are costly
> In a spam filter, wrongly marking an important email as spam is serious, so precision matters.

---

## 4. Recall / Sensitivity

**Recall**, also called **sensitivity** or **True Positive Rate (TPR)**, answers: *Of all actual positive samples, how many did the model find?*

$$
\text{Recall} = \frac{TP}{TP + FN}
$$

High recall means the model catches most positive cases and makes few false-negative mistakes.

> [!tip] Use recall when false negatives are costly
> In disease detection, fraud detection, or security-threat detection, missing a positive case can be dangerous, so recall is usually prioritized.

---

## 5. Specificity

**Specificity**, or **True Negative Rate (TNR)**, answers: *Of all actual negative samples, how many did the model correctly identify?*

$$
\text{Specificity} = \frac{TN}{TN + FP}
$$

It is useful when correctly identifying negatives matters, such as confirming that healthy patients are not diagnosed with a disease.

---

## 6. F1 Score

The **F1 score** combines precision and recall using their **harmonic mean**. It is useful when you need a single metric that considers both false positives and false negatives.

$$
F1 = 2 \times \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}
$$

- F1 ranges from 0 to 1.
- A high F1 requires **both** precision and recall to be reasonably high.
- It is often preferred for imbalanced datasets because it focuses on the positive/minority class.

> [!example] Worked example
> If precision $= 0.90$ and recall $= 0.60$:
>
> $$
> F1 = 2 \times \frac{0.90 \times 0.60}{0.90 + 0.60} = 2 \times \frac{0.54}{1.50} = 0.72
> $$

---

## 7. Precision–Recall Trade-off

Precision and recall usually trade off against each other as the classification threshold changes.

| Threshold | Effect on positives | Recall | Precision |
|---|---|---|---|
| **Low** | More samples classified as positive | Increases ↑ | May fall ↓ |
| **High** | Fewer samples classified as positive | May fall ↓ | May increase ↑ |

> [!example] Choosing a side
> - **Cancer screening:** choose high **recall** to avoid missing patients.
> - **Spam filter:** choose high **precision** to avoid blocking legitimate emails.

---

## 8. ROC Curve and AUC

The **ROC (Receiver Operating Characteristic) curve** plots the True Positive Rate:

$$
TPR = \frac{TP}{TP + FN}
$$

against the False Positive Rate:

$$
FPR = \frac{FP}{FP + TN}
$$

at different decision thresholds. It shows the trade-off between detecting positives and incorrectly flagging negatives.

**AUC (Area Under the ROC Curve)** summarizes the ROC curve into one number:

| AUC value | Interpretation |
|---:|---|
| 1.0 | Perfect classifier |
| 0.9 – 1.0 | Excellent |
| 0.8 – 0.9 | Good |
| 0.7 – 0.8 | Fair |
| 0.5 | Random guessing |
| Below 0.5 | Worse than random |

> [!info]
> AUC is the **probability that the model ranks a randomly chosen positive sample higher than a randomly chosen negative sample**. It is threshold-independent and useful for comparing binary classifiers.

---

## 9. Precision–Recall Curve and AUPRC

The **Precision–Recall curve** plots precision against recall across thresholds. **AUPRC** is the area under this curve.

> [!tip] Prefer AUPRC for rare positives
> It is often more informative than ROC-AUC when the positive class is rare, such as fraud detection, rare disease detection, or anomaly detection. ROC-AUC can look good even when the model performs poorly on the minority class, whereas precision-recall analysis directly reflects performance on that class.

---

## 10. Log Loss / Cross-Entropy

**Log loss**, also called **logarithmic loss** or **cross-entropy loss**, evaluates the quality of predicted **probabilities** rather than only final labels.

For binary classification:

$$
\text{Log Loss} = -\frac{1}{N}\sum_{i=1}^{N}
\left[y_i \log(p_i) + (1-y_i)\log(1-p_i)\right]
$$

Where:

- $y_i$: true label (0 or 1)
- $p_i$: predicted probability of the positive class

Key points:

- **Lower** log loss is better.
- It penalizes **confident wrong predictions** heavily.
- It is useful for models that output probabilities, such as logistic regression, neural networks, and gradient-boosting classifiers.

> [!example]
> If the true label is $1$, predicting probability $0.99$ gives a **small** loss, while predicting $0.01$ gives a **very large** loss.

---

## 11. Multi-class Metrics

For more than two classes, metrics are calculated per class and then averaged.

| Averaging method | Meaning |
|---|---|
| **Macro average** | Computes the metric for each class, then takes the unweighted mean; treats all classes equally |
| **Micro average** | Aggregates TP, FP, and FN across all classes, then computes the metric; dominated by frequent classes |
| **Weighted average** | Computes per-class metrics and weights them by class support |

> [!note] Support
> **Support** is the number of actual samples in a class. A classification report normally shows precision, recall, F1-score, and support for every class.

---

## 12. Choosing the Right Metric

| Scenario | Recommended metric | Reason |
|---|---|---|
| Balanced classes, equal error cost | Accuracy | Simple overall correctness measure |
| False positives are costly | Precision | Avoids incorrect positive predictions |
| False negatives are costly | Recall | Avoids missing positive cases |
| Need balance between FP and FN | F1 score | Combines precision and recall |
| Comparing classifiers across thresholds | ROC-AUC | Threshold-independent ranking measure |
| Rare positive class | Precision-recall curve / AUPRC | Focuses on minority-class performance |
| Model outputs probabilities | Log loss | Evaluates calibration and confidence |

---

## 13. Quick Formula Summary

| Metric | Formula |
|---|---|
| Accuracy | $\dfrac{TP + TN}{TP + TN + FP + FN}$ |
| Precision | $\dfrac{TP}{TP + FP}$ |
| Recall (TPR) | $\dfrac{TP}{TP + FN}$ |
| Specificity (TNR) | $\dfrac{TN}{TN + FP}$ |
| FPR | $\dfrac{FP}{FP + TN}$ |
| F1 Score | $2 \times \dfrac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}}$ |
| Log Loss | $-\dfrac{1}{N}\sum \left[y\log p + (1-y)\log(1-p)\right]$ |
