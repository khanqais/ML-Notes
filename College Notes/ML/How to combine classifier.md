Classifiers can be combined using **ensemble learning**: train multiple models and combine their outputs to get better accuracy, stability, and generalization than a single classifier. The main approaches are **voting, bagging, boosting, stacking, and blending**.[[scikit-learn](https://scikit-learn.org/stable/api/sklearn.ensemble.html)][[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)]

## Why combine classifiers?

A single model may make mistakes because it is biased, has high variance, or fails on certain patterns in the data. Combining multiple classifiers reduces the chance that one weak or unstable model controls the final decision.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[uw-madison-datascience.github](https://uw-madison-datascience.github.io/machine-learning-novice-sklearn/04-ensemble-methods.html)]

For example, one model may classify an email as spam, while another says it is not spam. An ensemble combines these opinions and makes a final prediction using voting, averaging, or a learned combination rule.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.BaggingClassifier.html)]

## Voting

In **voting**, several independently trained classifiers make predictions, and their outputs are combined by voting or averaging. It is commonly used with different algorithms such as logistic regression, decision tree, SVM, KNN, and random forest.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[codesofphoenix](https://www.codesofphoenix.com/articles/sklearn/ensemble-methods)]

### Hard voting

Each classifier gives one final class label. The class with the maximum number of votes wins.

Example:

|Classifier|Prediction|
|---|---|
|Logistic Regression|Spam|
|Decision Tree|Spam|
|SVM|Not Spam|

Final prediction: **Spam**, because 2 out of 3 classifiers voted for it.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[codesofphoenix](https://www.codesofphoenix.com/articles/sklearn/ensemble-methods)]

### Soft voting

Each classifier outputs class probabilities. The ensemble averages these probabilities and selects the class with the highest average probability.

Example:

|Classifier|P(Spam)|P(Not Spam)|
|---|---|---|
|Logistic Regression|0.8|0.2|
|Decision Tree|0.6|0.4|
|SVM|0.7|0.3|
|Average|0.7|0.3|

Final prediction: **Spam**, because its average probability is higher. Soft voting usually performs better when the base models produce well-calibrated probabilities.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[codesofphoenix](https://www.codesofphoenix.com/articles/sklearn/ensemble-methods)]

### Weighted voting

Some classifiers can be given more importance than others:

```
\hat{y} = \arg\max_c \sum_i w_i P_i(y=c \mid x)
```

For example, if a random forest is more reliable than a KNN model, it can receive a higher weight. This is useful when validation performance differs across models.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/voting-in-machine-learning/)]

## Bagging

**Bagging** means **Bootstrap Aggregating**. Multiple models of the same type are trained in parallel on different random subsets of the training data, and their predictions are combined through majority voting or probability averaging.[[scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.BaggingClassifier.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/)]

### How it works

1. Create multiple bootstrap samples from the training dataset by sampling **with replacement**.
2. Train one classifier on each sample.
3. For classification, take a majority vote; for regression, average the predictions.
4. Use the combined result as the final prediction.[[scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.BaggingClassifier.html)][[fall-2023-python-programming-for-data-science.readthedocs](https://fall-2023-python-programming-for-data-science.readthedocs.io/en/latest/Lectures/Theme_3-Model_Engineering/Lecture_14-Ensemble_Methods/Lecture_14-Ensemble_Methods.html)]

Example: train 100 decision trees on 100 different random samples of the data. If 70 trees predict “Fraud” and 30 predict “Not Fraud,” the final prediction is **Fraud**.

### Why it works

Bagging mainly reduces **variance**. A single decision tree can overfit and change greatly with small changes in training data, but averaging many trees makes the final model more stable.[[scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.BaggingClassifier.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/)]

**Random Forest** is the most popular bagging-based classifier. It trains many decision trees on random data samples and also uses random feature subsets at each split, making the trees less correlated.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[machinelearningmastery](https://machinelearningmastery.com/demystifying-ensemble-methods-boosting-bagging-and-stacking-explained/)]

## Boosting

**Boosting** trains weak learners **sequentially**. Each new model focuses more on the examples that the previous models classified incorrectly. The final prediction is a weighted combination of all models.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/)]

### How it works

1. Train the first weak classifier.
2. Identify the samples it misclassified.
3. Give those samples higher importance or weight.
4. Train the next classifier to focus on those difficult samples.
5. Repeat for several rounds.
6. Combine all classifiers using weights based on their accuracy.[[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/)][[fall-2023-python-programming-for-data-science.readthedocs](https://fall-2023-python-programming-for-data-science.readthedocs.io/en/latest/Lectures/Theme_3-Model_Engineering/Lecture_14-Ensemble_Methods/Lecture_14-Ensemble_Methods.html)]

Example: In AdaBoost, misclassified samples receive higher weights, so the next weak learner pays more attention to them. Final classification is based on a weighted vote of all weak learners.[[scikit-learn](https://scikit-learn.org/stable/api/sklearn.ensemble.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/)]

### Boosting algorithms

|Algorithm|Main idea|
|---|---|
|AdaBoost|Increases the weight of misclassified samples|
|Gradient Boosting|Each new model fits the residual errors of the previous model|
|XGBoost|Optimized gradient boosting with regularization and efficient implementation|
|LightGBM|Fast gradient boosting using histogram-based splitting|
|CatBoost|Gradient boosting designed to handle categorical features well|

Boosting mainly reduces **bias** and often achieves high accuracy, but it can overfit if the number of learners or learning rate is not tuned properly.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[geeksforgeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/)]

## Stacking

**Stacking**, or **stacked generalization**, trains several different base classifiers and then trains a **meta-classifier** to combine their predictions. Instead of simply voting, the meta-model learns which base model to trust more in different situations.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[scikit-learn](https://scikit-learn.org/dev/modules/ensemble.html)]

### How it works

1. Split the training data into folds.
2. Train base models such as logistic regression, random forest, SVM, and gradient boosting.
3. Generate out-of-fold predictions from each base model.
4. Use those predictions as new input features for a meta-model.
5. Train the meta-model to make the final prediction.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[scikit-learn](https://scikit-learn.org/dev/modules/ensemble.html)]

Example:

|Base model|Prediction|
|---|---|
|Logistic Regression|0.72|
|Random Forest|0.85|
|SVM|0.60|

These values become inputs to a meta-classifier, such as logistic regression, which learns the final prediction.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[machinelearningmastery](https://machinelearningmastery.com/demystifying-ensemble-methods-boosting-bagging-and-stacking-explained/)]

### Important point

The meta-model must be trained on **out-of-fold predictions**, not predictions made on the same data used to train the base models. Otherwise, it can learn from leaked information and overfit. Scikit-learn’s `StackingClassifier` handles this using cross-validation.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[scikit-learn](https://scikit-learn.org/dev/modules/ensemble.html)]

## Blending

**Blending** is similar to stacking, but it usually uses a simple fixed validation split instead of full cross-validation.

1. Split data into training and validation sets.
2. Train base models on the training set.
3. Generate predictions on the validation set.
4. Train a meta-model using those validation predictions.
5. For final testing, base models predict on test data and the meta-model combines those predictions.

It is simpler and faster than stacking but may use less training data effectively because the validation set is held out.[[scikit-learn](https://scikit-learn.org/1.9/modules/ensemble.html)][[fall-2023-python-programming-for-data-science.readthedocs](https://fall-2023-python-programming-for-data-science.readthedocs.io/en/latest/Lectures/Theme_3-Model_Engineering/Lecture_14-Ensemble_Methods/Lecture_14-Ensemble_Methods.html)]

## Comparison

|Method|Training style|Combination rule|Main benefit|Main risk|
|---|---|---|---|---|
|Voting|Models trained independently|Majority vote or averaged probabilities|Simple and robust|Does not learn optimal model weights|
|Bagging|Models trained in parallel on bootstrap samples|Voting/averaging|Reduces variance and overfitting|May not help if base models are biased|
|Boosting|Models trained sequentially|Weighted combination|Reduces bias and improves accuracy|Can overfit; sensitive to noisy data|
|Stacking|Base models plus meta-model|Learned combination|Can learn the best way to combine models|More complex; risk of data leakage|
|Blending|Base models plus meta-model on validation set|Learned combination|Simple version of stacking|Uses less data for meta-model training|



