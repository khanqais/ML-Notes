
## Bias:
The inability for a machine learning method (like linear regression) to capture the [[True Relationship]] is called bias.
Because the Straight Line can’t be | curved like the “true” relationship, it | ‘has a relatively large amount of bias.
Because the Straight Line can’t be curved like the “true” relationship, it has a relatively large amount of bias.
![[Pasted image 20260926162343.png|350]]

## Variance

![[Pasted image 20260926162552.png|642]]

***Defination*** : Variance refers to the amount by which the predictions would change if we fit the model to a different training data set.

The Squiggly Line has low bias, since it is flexible and can adapt to the curve in the relationship between weight and hight. but the Squiggly Line has high variability, because it results in vastly different Sums of Squares for different datasets. In other words, it’s hard to predict how well
the Squiggly Line will perform with future data sets. It might do well sometimes, and other times it might do terribly.
In contrast, the Straight Line has relatively | high bias, since it can not capture the curve in | the relationship between weight and height.In other words, the Straight Line might only  give good predictions, and not great predictions. But they will be consistently good predictions. 

**Overfit**: Because the Squiggly Line fits the training set really well, but not the testing set, we say that the Squiggly Line is overfit.

Three commonly used methods for finding the sweet spot between simple and complicated models are: regularization, boosting and bagging.

