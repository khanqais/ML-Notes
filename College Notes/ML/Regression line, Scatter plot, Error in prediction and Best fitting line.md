A **scatter plot** shows the actual data points, a **regression line** is the straight line used to model their trend, **prediction error** measures how far a real value is from the predicted value, and the **best-fitting line** is the line that minimizes those errors—usually by minimizing squared errors.

## Scatter plot

A scatter plot displays each observation as a point `(x, y)`. For example, if `x` is hours studied and `y` is marks obtained, every student is one dot on the graph.

It helps you see:

- Whether the relationship is positive (both increase together), negative (one increases while the other decreases), or has no clear pattern.
- Whether the relationship looks roughly linear.
- Whether there are outliers—points far away from the general trend.
## Regression line

A **regression line** is a straight line that represents the average relationship between the independent variable `x` and dependent variable `y`. It is used to estimate `y` for a given value of `x`.

For simple linear regression, the line is:

```
\hat{y} = a + bx
```

- `\hat{y}`: predicted value of `y`
- `a`: intercept, the predicted `y` when `x = 0`
- `b`: slope, the expected change in `y` for a one-unit increase in `x`

Example: if the regression line is `\hat{y} = 40 + 5x`, then studying 2 extra hours is predicted to increase marks by 10, on average.

## Error in prediction

The **error in prediction**, also called a **residual**, is the difference between the actual observed value and the value predicted by the regression line:

```
e_i = y_i - \hat{y}_i
```

- If `e_i > 0`, the actual value is above the line.
- If `e_i < 0`, the actual value is below the line.
- If `e_i = 0`, the line predicted the value exactly.

For example, if a student actually scored 82 but the line predicted 78, then:

```
e = 82 - 78 = 4
```

So the prediction error is 4 marks.

## Best-fitting line

The **best-fitting line** is the regression line that gives the smallest total prediction error. In ordinary least-squares regression, we square each residual and add them:

```
SSE = \sum_{i=1}^{n}(y_i - \hat{y}_i)^2
```

The line with the **smallest sum of squared errors (SSE)** is called the best-fitting line or least-squares regression line

We square the errors because:

- Negative and positive errors would otherwise cancel each other.
- Squaring penalizes large errors more heavily.
- It gives a stable mathematical method to find the optimal slope and intercept.
## Simple example

Suppose these are study hours and marks:

|Hours studied `x`|Actual marks `y`|Predicted marks `\hat{y}`|Error `y - \hat{y}`|
|---|---|---|---|
|1|45|43|2|
|2|52|50|2|
|3|58|57|1|

Here, the regression line predicts values close to the actual values. The small residuals mean the line fits the data well. The best-fitting line is the one that would make the total of squared residuals as small as possible.