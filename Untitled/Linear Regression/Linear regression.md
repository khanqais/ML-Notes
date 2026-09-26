
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

Step 2: Measuring Fit Quality with R²
R² tells you what fraction of the variation in your outcome variable is explained by your predictor(s).
To compute it, you compare two sums of squares:
SS(mean) — collapse all data onto just the y-axis (ignore the predictor entirely) and measure squared distances from each point to the overall mean. This is the "naive" baseline variation, with no explanatory model at all.
SS(fit) — measure squared distances from each point to your fitted regression line instead of the mean.
Dividing either sum of squares by the sample size n gives you a variance (average squared distance per data point) — variance is just sum of squares divided by the count of things being averaged.
![[Pasted image 20260926120944.png]]

This is the reduction in variance you achieve by using the predictor instead of just guessing the mean every time.
If R² = 1 (100%), the predictor lets you perfectly predict the outcome — zero residuals around the fit.
If R² = 0 (0%), the predictor tells you nothing — the fit does no better than just guessing the mean.
A value like R² = 0.6 means 60% of the variation in the outcome is explained by the predictor(s); the remaining 40% is unexplained (noise or other factors).

P value:
![[Pasted image 20260926124750.png|444]]
SS(fit) = 0 because you can always draw a straight line to connect any two points.
R² will always be 100% cuz the distance between these two points will always be equal. But a line can fit a small dataset impressively even when there is no real relationship

So read them together. An R^2 of 0.60 describes a substantial reduction in squared error _for the observed mice_, but its p-value depends partly on how many mice were measured and how much noise remains
the p-value asks whether the line using mouse weight improves on predicting every mouse’s size with the average size by more than we’d expect from chance















