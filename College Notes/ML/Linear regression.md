## Concept
Regression is a **supervised ML technique** used to predict a **continuous value**.

- Input: BMI ($X$)
- Output: Cholesterol ($Y$)
- Method: **Simple Linear Regression**

## Formula
$$
Y = a + bX
$$

- $Y$ = predicted value
- $X$ = input feature
- $a$ = intercept (value of $Y$ when $X = 0$)
- $b$ = slope (change in $Y$ for a 1-unit increase in $X$)

### Slope
$$
b = \frac{\sum (X-\bar{X})(Y-\bar{Y})}{\sum (X-\bar{X})^2}
$$

### Intercept
$$
a = \bar{Y} - b\bar{X}
$$

## Example: BMI → Cholesterol
Given:
$$
\bar{X} = 19.4, \quad \bar{Y} = 163
$$

$$
b = \frac{1514}{172.4} = 8.78
$$

$$
a = 163 - 8.78(19.4) = -7.37
$$

### Regression Equation
$$
\text{Cholesterol} = -7.37 + 8.78(\text{BMI})
$$

### Prediction for BMI = 27
$$
Y = -7.37 + 8.78(27)
$$

$$
\boxed{Y \approx 229.74}
$$

> [!note] Rounding
> Using the rounded values (8.78 and -7.37) gives 229.69. Using the unrounded slope (8.7819) and intercept (-7.368) gives 229.74. Keep extra decimals in intermediate steps.

---

# Regression Evaluation Metrics

Once a model is fitted, we need to measure **how far its predictions are from reality**. All metrics below are built from the **residuals**.

## Residual / Error
$$
e_i = Y_i - \hat{Y}_i
$$

- $Y_i$ = actual value
- $\hat{Y}_i$ = predicted value
- $e > 0$: model **under-predicted**
- $e < 0$: model **over-predicted**
- Residuals sum to ~0 in linear regression, so we can't just average them. That is why the metrics below take absolute values or squares.

## MAE — Mean Absolute Error
$$
MAE = \frac{1}{n}\sum |Y_i-\hat{Y}_i|
$$

- Average size of the error, ignoring direction
- Same unit as target (e.g. mg/dL)
- Every error counts **proportionally**, so it is **robust to outliers**
- Easy to explain: "on average, we're off by MAE units"

## MSE — Mean Squared Error
$$
MSE = \frac{1}{n}\sum (Y_i-\hat{Y}_i)^2
$$

- Squaring removes the sign and **punishes large errors heavily** (an error of 10 counts 100× more than an error of 1)
- Unit is **squared** (mg/dL)², so it is hard to interpret directly
- Smooth and differentiable, so it is the usual **loss function** for training

## RMSE — Root Mean Squared Error
$$
RMSE = \sqrt{MSE} = \sqrt{\frac{1}{n}\sum (Y_i-\hat{Y}_i)^2}
$$

- Brings the error back to the **original unit**
- Interpretation: the typical size of a prediction error
- Always $RMSE \ge MAE$. A **large gap** between them signals a few big errors (outliers).

### MAE vs RMSE: mini example
Errors: $2, 2, 2, 2, 12$

$$
MAE = \frac{2+2+2+2+12}{5} = 4
$$

$$
RMSE = \sqrt{\frac{4+4+4+4+144}{5}} = \sqrt{32} \approx 5.66
$$

The single outlier (12) pulls RMSE well above MAE.

## R-squared — Coefficient of Determination
$$
R^2 = 1-\frac{SSE}{SST}
$$

where:

$$
SSE = \sum (Y_i-\hat{Y}_i)^2 \quad \text{(unexplained variation, error of the model)}
$$

$$
SST = \sum (Y_i-\bar{Y})^2 \quad \text{(total variation, error of the "predict the mean" baseline)}
$$

**Idea:** compare your model against the dumbest possible model, which always predicts $\bar{Y}$.

| $R^2$ | Meaning |
|---|---|
| 1 | Perfect fit |
| 0 | No better than predicting the mean |
| < 0 | Worse than predicting the mean |

- Unitless, so it can be compared across different targets
- **Never decreases** when you add features, even useless ones. For multiple regression use **Adjusted $R^2$**:

$$
R^2_{adj} = 1 - (1-R^2)\frac{n-1}{n-p-1}
$$

where $n$ = number of samples and $p$ = number of features.

### Example
Here $n = 10$ and $SSE = 2316.20$.

$$
MSE = \frac{2316.20}{10} = 231.62
$$

$$
RMSE = \sqrt{231.62} = 15.22
$$

$$
R^2 = 1-\frac{2316.20}{15612} = 0.8516
$$

$$
\boxed{R^2 \approx 85.16\%}
$$

**Interpretation**
- BMI explains about **85.16%** of the variation in cholesterol.
- The remaining ~14.84% is due to other factors or noise.
- RMSE = 15.22 means predictions are typically off by about **15 units** of cholesterol.
- Adjusted $R^2 = 1 - 0.1484 \times \frac{9}{8} \approx 0.833$

## Which metric to use?

| Metric | Unit | Outlier sensitivity | Best for |
|---|---|---|---|
| MAE | Same as target | Low | Easy-to-explain average error |
| MSE | Squared | High | Training / optimization |
| RMSE | Same as target | High | Reporting error when big misses are costly |
| $R^2$ | None | Moderate | How much variance the model explains |

> [!tip]
> Report **RMSE (or MAE) together with $R^2$**. $R^2$ says how well the model explains variation, while RMSE/MAE says how large the errors actually are.

---

# Types of Regression
- **Simple Linear Regression**: one input feature
- **Multiple Linear Regression**: multiple input features
- **Polynomial Regression**: curved relationship
- **Ridge Regression**: L2 regularization
- **Lasso Regression**: L1 regularization, feature selection
- **Logistic Regression**: used for classification, not continuous prediction
- **Decision Tree Regression**: tree-based regression
- **Random Forest Regression**: ensemble of decision trees