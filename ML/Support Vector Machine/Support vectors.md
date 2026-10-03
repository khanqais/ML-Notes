## What problem does SVM solve?

Suppose we have two classes:

- Positive class: +1
- Negative class: −1

Each data point has features. For example, a flower may be represented by:

x=(x1,x2)
where:
- x1​ = petal length
- x2​ = petal width
SVM tries to find a boundary that separates the two classes.
Feature x₂
   ↑
   |       +       +       +
   |      +       +       +
   |
   |------------------------------→ Feature x₁
   |      -       -       -
   |        -       -       -

![[Pasted image 20261002235916.png|427]]

## Support vectors

The points closest to the decision boundary are called **support vectors**.
They determine the position and orientation of the optimal hyperplane
![[Pasted image 20261003000116.png]]
The decision boundary is primarily determined by the closest points from each class, not by every point equally


## Kernel trick

To solve nonlinear problems, SVM can transform the data into a higher-dimensional feature space.

Suppose the original features are x1x_1x1​ and x2x_2x2​. We can add a new feature:

![[Pasted image 20261003000244.png|120]]​

A circular pattern in two dimensions may become linearly separable in a space involving zzz