
Linear regression boils down to three core steps: fit a line using least squares, quantify the fit's quality with R², and test whether that fit is statistically meaningful using a p-value derived from the F-statistic. Here's the full theory, explained in depth

Step 1: Fitting the Line (Least Squares)
Given data — say, mouse weight (predictor, x) and mouse size (outcome, y) — you want a line y = intercept + slope × x that best represents the relationship.
The process:
Draw a candidate line through the data.
For each data point, measure the vertical distance from the point to the line — this is called a residual.
![[Pasted image 20260926121027.png|352]]
Square each residual (squaring removes the effect of sign, so points above and below the line don't cancel out) and sum them all. This total is the sum of squares for that line.
Conceptually, you rotate/shift the line repeatedly, recomputing the sum of squared residuals each time.
Plotting sum-of-squares against rotation angle produces a curve, and the line that sits at the minimum of that curve is the least squares fit. In practice this minimum is found analytically via calculus (setting derivatives to zero) rather than by literal trial rotation, but the intuition holds

# Linear Regression: R² and Variation

## 1. What is Variation?

**Variation** means how much the actual values differ from each other.

**Example:** marks of students

| Student | Study Hours | Marks |
|---|---:|---:|
| A | 1 | 30 |
| B | 2 | 50 |
| C | 3 | 70 |
| D | 4 | 90 |

Marks range from 30 to 90, so there is **high variation**.

> [!note]
> If all students scored 60, there would be **no variation**.

---

## 2. Why Do We Need R²?

R² answers this question:

> [!question]
> Out of the total variation in the target variable, how much variation did my regression model explain?

**Example:**
- Goal: predict **marks** using **study hours**
- Marks vary because of study hours, sleep, health, exam difficulty, etc.
- R² tells us how much of that variation is explained by study hours.

---

## 3. Baseline: Predicting the Mean

If we know nothing about study hours, the best simple guess is the **mean marks**.

$$
\bar{y} = \frac{30 + 50 + 70 + 90}{4} = 60
$$

So without any model, we predict **60 marks for every student**.

- Actual marks: 30, 50, 70, 90
- Baseline prediction: 60, 60, 60, 60

**Errors:**

$$
30 - 60 = -30
$$

$$
50 - 60 = -10
$$

$$
70 - 60 = 10
$$

$$
90 - 60 = 30
$$

We **square** the errors so negative and positive errors do not cancel out:

$$
SS_{\text{total}} = (-30)^2 + (-10)^2 + 10^2 + 30^2
$$

$$
SS_{\text{total}} = 900 + 100 + 100 + 900 = 2000
$$

### Meaning

$$
SS_{\text{total}} = 2000
$$

This is the **total variation** in marks. It is also called:

- Total Sum of Squares
- $SS_{\text{total}}$
- $SS(\text{mean})$

> [!info]
> It represents the error if we **always predict the mean**.

General formula:

$$
SS_{\text{total}} = \sum_{i=1}^{n} (y_i - \bar{y})^2
$$

---

## 4. Variation Around the Regression Line

Suppose our regression model predicts marks using study hours.

**Fitted line:**

$$
\hat{y} = 20x + 10
$$

| Student | Actual Marks | Predicted Marks | Error |
|---|---:|---:|---:|
| A | 30 | 30 | 0 |
| B | 50 | 50 | 0 |
| C | 70 | 70 | 0 |
| D | 90 | 90 | 0 |

**Residual Sum of Squares:**

$$
SS_{\text{residual}} = 0^2 + 0^2 + 0^2 + 0^2 = 0
$$

The regression line predicted every value perfectly.

General formula:

$$
SS_{\text{residual}} = \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
$$

---

## 5. R² Formula

$$
R^2 = 1 - \frac{SS_{\text{residual}}}{SS_{\text{total}}}
$$

**For the perfect model:**

$$
R^2 = 1 - \frac{0}{2000} = 1
$$

> [!success]
> **R² = 1** means the model explained **100%** of the variation in the target variable.

---

## 6. Another Example

Suppose the model gives these predictions:

| Student | Actual Marks | Predicted Marks | Error |
|---|---:|---:|---:|
| A | 30 | 35 | -5 |
| B | 50 | 45 | 5 |
| C | 70 | 65 | 5 |
| D | 90 | 85 | 5 |

**Step 1: Residual sum of squares**

$$
SS_{\text{residual}} = (-5)^2 + 5^2 + 5^2 + 5^2 = 25 + 25 + 25 + 25 = 100
$$

**Step 2: Calculate R²**

$$
R^2 = 1 - \frac{100}{2000}
$$

$$
R^2 = 1 - 0.05 = 0.95
$$

### Interpretation

- Total variation in marks = $2000$
- Variation left after using the model = $100$
- Variation explained by the model = $2000 - 100 = 1900$
- Percentage explained = $\frac{1900}{2000} = 95\%$

> [!tip]
> Study hours explain **95%** of the variation in marks.

The remaining 5% may be due to:

- Sleep
- Health
- Exam difficulty
- Luck
- Other factors

---

## 7. R² in Simple Words

$$
R^2 = \frac{\text{Variation explained by model}}{\text{Total variation}}
$$

Another form:

$$
R^2 = \frac{SS_{\text{total}} - SS_{\text{residual}}}{SS_{\text{total}}}
$$

> [!abstract] Key idea
> R² measures **how much better the regression line is compared to always predicting the mean**.

---

## 8. R² Values

| R² Value | Meaning |
|---:|---|
| 1 | Model explains 100% of variation (perfect prediction) |
| 0.8 | Model explains 80% of variation |
| 0.6 | Model explains 60% of variation |
| 0 | Model is no better than predicting the mean |
| Negative | Model is worse than predicting the mean |

---

## 9. Pizza Analogy 🍕

Think of total variation as a **pizza**.

- Whole pizza = total variation in marks
- Slice eaten by the model = variation explained by study hours
- Leftover slice = unexplained variation

If $R^2 = 0.6$, the model explained **60%** of the pizza, and the remaining **40%** is unexplained variation.

---

## 10. Important Point

> [!warning]
> R² is **not accuracy**.

It tells us how much variation in the target variable is explained by the model.

**Example:** $R^2 = 0.6$ means:

> 60% of the variation in the outcome is explained by the predictor variables.
> The remaining 40% is unexplained.



P value:
![[Pasted image 20260926124750.png|444]]
SS(fit) = 0 because you can always draw a straight line to connect any two points.
R² will always be 100% cuz the distance between these two points will always be equal. But a line can fit a small dataset impressively even when there is no real relationship

So read them together. An R^2 of 0.60 describes a substantial reduction in squared error _for the observed mice_, but its p-value depends partly on how many mice were measured and how much noise remains
the p-value asks whether the line using mouse weight improves on predicting every mouse’s size with the average size by more than we’d expect from chance
















