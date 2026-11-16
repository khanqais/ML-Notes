![[Pasted image 20260926170928.png|502]]

In this regression it make S shaped logistic function to predict something is True or False Right or wrong, etc.
![[Pasted image 20260926171442.png|329]]
If the weight is high there is high probability that it is obese vise versa.


# Logistic Regression

> [!abstract] Summary
> Logistic Regression is a **supervised classification algorithm** used when the output is categorical, for example spam/not spam, pass/fail, or disease/no disease. Despite having "regression" in its name, it predicts a **probability between 0 and 1**, then converts that probability into a class using a threshold.

---

## 1. Core Idea

Linear regression predicts a continuous number:

$$
\hat{y} = wx + b
$$

Logistic regression predicts the **probability of an event**:

$$
P(y=1 \mid x) = \sigma(wx+b)
$$

Here, $\sigma$ is the **sigmoid function**:

$$
\sigma(z) = \frac{1}{1+e^{-z}}
$$

It converts any real number $z$ into a value between 0 and 1.

## 2. Step-by-Step Example

Predict whether a student will **pass** an exam based on study hours.

| Hours studied | Passed? |
|---:|---:|
| 1 | No |
| 2 | No |
| 3 | Yes |
| 4 | Yes |
| 5 | Yes |

Encode the target as:

- $y = 0$: Fail
- $y = 1$: Pass

Suppose the trained model is:

$$
z = 2x - 5
$$

$$
P(\text{Pass}) = \frac{1}{1+e^{-z}}
$$

### Case 1: Student studies 3 hours

$$
z = 2(3) - 5 = 1
$$

$$
P(\text{Pass}) = \frac{1}{1+e^{-1}} \approx 0.73
$$

The model says there is a **73% probability** that the student will pass.

Using the usual threshold of 0.5:

$$
0.73 > 0.5
$$

> [!success]
> Predicted class: **Pass**

### Case 2: Student studies 1 hour

$$
z = 2(1) - 5 = -3
$$

$$
P(\text{Pass}) = \frac{1}{1+e^{3}} \approx 0.047
$$

Since $0.047 < 0.5$:

> [!failure]
> Predicted class: **Fail**

---

## 3. What the Model Learns

Logistic regression learns values for $w$ and $b$:

$$
z = w_1x_1 + w_2x_2 + \cdots + w_nx_n + b
$$

| Symbol | Meaning |
|---|---|
| $w$ | Weight for each feature |
| $b$ | Bias / intercept |
| $z$ | Linear score, also called the **logit** |
| $P(y=1)$ | Probability of the positive class |

- A **positive weight** increases the probability of class 1 as that feature increases.
- A **negative weight** decreases it.

> [!example] Loan default model
> Higher **income** may have a negative coefficient (more income reduces default probability), while a higher **loan amount** may have a positive coefficient.

---

## 4. Sigmoid Function

The sigmoid maps the linear score $z$ into a probability:

| $z$ | Sigmoid output |
|---:|---:|
| -10 | Close to 0 |
| -1 | 0.27 |
| 0 | 0.5 |
| 1 | 0.73 |
| 10 | Close to 1 |

> [!note]
> At $z = 0$, the model is equally unsure, so the probability is 0.5. This is the **decision boundary** for a standard threshold.

---

## 5. Classification Threshold

After getting the probability, we choose a class:

$$
\hat{y} =
\begin{cases}
1, & P(y=1) \geq 0.5 \\
0, & P(y=1) < 0.5
\end{cases}
$$

| Probability | Class | Example |
|---:|---:|---|
| 0.85 | 1 | Spam |
| 0.60 | 1 | Spam |
| 0.40 | 0 | Not spam |
| 0.10 | 0 | Not spam |

> [!tip] Threshold can be changed
> - **Medical diagnosis:** use a *lower* threshold, because missing a disease is more dangerous than a false alarm.
> - **Fraud detection:** tune the threshold based on the cost of false positives vs false negatives.

---

## 6. Loss Function

Linear regression uses **Mean Squared Error**. Logistic regression uses **binary cross-entropy loss**, also called **log loss**:

$$
\text{Loss} = -\frac{1}{n}\sum_{i=1}^{n}
\left[
y_i \log(\hat{p}_i) + (1-y_i)\log(1-\hat{p}_i)
\right]
$$

Where:

- $y_i$: actual label (0 or 1)
- $\hat{p}_i$: predicted probability of class 1

### Intuition

- If actual $y = 1$, the loss is $-\log(\hat{p})$. A prediction close to 1 gives low loss.
- If actual $y = 0$, the loss is $-\log(1-\hat{p})$. A prediction close to 0 gives low loss.
- If the model is **confidently wrong**, the loss becomes very large.

> [!info]
> The model updates its weights using **gradient descent** to reduce this loss.

---

## 7. Decision Boundary

For binary logistic regression with two features:

$$
z = w_1x_1 + w_2x_2 + b
$$

The decision boundary is where $z = 0$:

$$
w_1x_1 + w_2x_2 + b = 0
$$

This creates a **straight-line boundary** in 2D. Points on one side are class 0, and points on the other side are class 1.

### Example

$$
z = 2x_1 + 3x_2 - 6
$$

Decision boundary:

$$
2x_1 + 3x_2 - 6 = 0
$$

- If $z > 0$, predict **class 1**.
- If $z < 0$, predict **class 0**.

---

## 8. Real-World Examples

| Problem | Input features | Output |
|---|---|---|
| Email spam detection | Number of links, suspicious words, sender domain | Spam / Not spam |
| Loan approval | Income, credit score, loan amount | Approved / Rejected |
| Disease diagnosis | Age, blood pressure, test result | Disease / No disease |
| Customer churn | Usage, complaints, tenure | Churn / Stay |
| Admission prediction | GPA, test score, extracurriculars | Admitted / Rejected |

Logistic regression is commonly used for binary outcomes such as purchase/no purchase, default/no default, and disease/no disease.

---

## 9. Linear vs Logistic Regression

| Aspect | Linear Regression | Logistic Regression |
|---|---|---|
| Output | Continuous value | Probability, then class |
| Example output | House price: ₹45 lakh | Probability of approval: 0.82 |
| Target type | Numeric | Categorical |
| Function used | Straight line | Sigmoid curve |
| Loss function | MSE | Binary cross-entropy |
| Use case | Predict price, marks, temperature | Predict spam, pass/fail, disease |

---

## 10. Important Assumptions

> [!warning] Assumptions
> - The relationship between features and the **log-odds** of the outcome is linear.
> - Observations are independent.
> - There is little multicollinearity among features.
> - The target should not have extreme class imbalance without proper handling.
> - Outliers can still affect the fitted coefficients.

---

## 11. Quick Formula Summary

| Term | Formula |
|---|---|
| Linear score (logit) | $z = w_1x_1 + \cdots + w_nx_n + b$ |
| Sigmoid | $\sigma(z) = \dfrac{1}{1+e^{-z}}$ |
| Probability | $P(y=1 \mid x) = \sigma(z)$ |
| Prediction | $\hat{y} = 1$ if $P \geq 0.5$, else $0$ |
| Log loss | $-\dfrac{1}{n}\sum \left[y\log\hat{p} + (1-y)\log(1-\hat{p})\right]$ |
| Decision boundary | $w_1x_1 + w_2x_2 + b = 0$ |

---

## 12. Interview-Ready Answer

> [!example] Interview Answer
> Logistic Regression is a supervised classification algorithm that predicts the probability of a categorical outcome. It computes a linear score from input features and passes it through a sigmoid function to produce a probability between 0 and 1. Based on a threshold, usually 0.5, it assigns a class. It is trained using binary cross-entropy loss and gradient descent. For example, it can predict whether a loan applicant will default using income, credit score, and loan amount.

---

## 13. Simple Analogy

Think of logistic regression as a **risk scorer**.

- It gives every student a pass/fail risk score.
- A score above 50% means "more likely to pass."
- A score below 50% means "more likely to fail."

It does not say "this student will score 72 marks." Instead, it says "this student has a **73% chance** of passing."

---

## Related Notes

- [[Linear Regression]]
- [[R-squared and Variation]]

## Sources

- [IBM: Logistic Regression](https://www.ibm.com/think/topics/logistic-regression)
- [Statistics By Jim: Logistic Regression](https://statisticsbyjim.com/regression/logistic-regression/)