![[Pasted image 20261002193907.png|301]]
We want to use the variables (Chest Pain, Good Blood, Circulation, etc.) to predict if someone has heart disease.
![[Pasted image 20261002194316.png|423]]
Cross validation allows us to compare different machine learning methods and get a sense of how well they will work in practice.
![[Pasted image 20261002194649.png|481]]
![[Pasted image 20261002195110.png|483]]
**K-fold cross-validation** is a technique to evaluate a machine-learning model more reliably by using every part of the dataset for both training and validation. Instead of relying on one train-test split, the data is divided into `k` folds, and the model is trained and evaluated `k` times.

## Basic idea

In normal evaluation, you split data once into training and testing sets. The problem is that the test score can depend heavily on which samples happened to go into the test set.

K-fold cross-validation reduces this randomness:

1. Split the dataset into `k` roughly equal parts called **folds**.
2. Train the model on `k-1` folds.
3. Validate/test the model on the remaining one fold.
4. Repeat this `k` times so that every fold is used as the validation set exactly once.
5. Average the `k` validation scores to get the final performance estimate.

## Example: 5-fold validation

Suppose you have 1,000 records and choose `k = 5`. The dataset is divided into 5 folds of 200 records each.

|Iteration|Training data|Validation data|
|---|---|---|
|1|Folds 2, 3, 4, 5|Fold 1|
|2|Folds 1, 3, 4, 5|Fold 2|
|3|Folds 1, 2, 4, 5|Fold 3|
|4|Folds 1, 2, 3, 5|Fold 4|
|5|Folds 1, 2, 3, 4|Fold 5|

If the accuracy scores are 0.84, 0.86, 0.82, 0.85, and 0.83, then:

```
\text{Mean accuracy} = \frac{0.84 + 0.86 + 0.82 + 0.85 + 0.83}{5} = 0.84
```

So the model has an estimated cross-validation accuracy of **84%**.[[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/k-fold-cross-validation-in-machine-learning/)][[machinelearningmastery](https://machinelearningmastery.com/k-fold-cross-validation/)]

## Why use K-fold?

- It gives a more stable estimate of model performance because every record is validated once.
- It uses the available data efficiently, which is useful when the dataset is small.
- It helps detect overfitting: if training performance is high but validation performance is low, the model may not generalize well.
- It is useful for comparing models and tuning hyperparameters fairly.[[scikit-learn](http://scikit-learn.org/dev/modules/cross_validation.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/cross-validation-machine-learning/)][[towardsdatascience](https://towardsdatascience.com/k-fold-cross-validation-explained-in-plain-english-659e33c0bc0/)]

## K-fold vs train-test split

|Aspect|Single train-test split|K-fold cross-validation|
|---|---|---|
|Number of evaluations|One|`k`|
|Data used for validation|Only one fixed test portion|Every fold is validated once|
|Score reliability|Can vary based on the split|More stable because results are averaged|
|Data usage|Some data is never used for validation|All data is used for validation|
|Best for|Quick checks and large datasets|Reliable evaluation and model comparison|

K-fold is especially valuable when you have limited data, because it avoids wasting a large portion of the dataset only for testing.[[scikit-learn](http://scikit-learn.org/dev/modules/cross_validation.html)][[towardsdatascience](https://towardsdatascience.com/k-fold-cross-validation-explained-in-plain-english-659e33c0bc0/)]

## Choosing K

Common choices are `k = 5` or `k = 10`.[[scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.cross_validate.html)][[theknowledgeacademy](https://www.theknowledgeacademy.com/blog/k-fold-cross-validation-in-machine-learning/)]

|Value of K|Advantage|Disadvantage|
|---|---|---|
|3|Fast and uses less computation|Less reliable performance estimate|
|5|Good balance between reliability and speed|Requires 5 training runs|
|10|More reliable estimate|More computation time|
|`k = n`|Uses almost all data for training in each run|Very expensive; called Leave-One-Out CV|

A larger `k` gives more training data in each iteration and a more stable estimate, but it takes longer because the model must be trained `k` times.[[scikit-learn](http://scikit-learn.org/dev/modules/cross_validation.html)][[machinelearningmastery](https://machinelearningmastery.com/k-fold-cross-validation/)]
