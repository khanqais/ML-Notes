knn() forms predictions using a single command. The function requires four inputs. 

1. A matrix containing the predictors associated with the training data, labeled train.X  below. 

2. A matrix containing the predictors associated with the data for which we wish to make predictions, labeled test.X  below. 

3. A vector containing the class labels for the training observations, labeled train.Direction below. 

4. A value for K, the number of nearest neighbors to be used by the classifier

![[Pasted image 20260928162232.png|457]]
If we take the value of k=3 then we the new value star will be class B but if we take the value of k=6 then the star will belong to class A.

Pros:
- Very simple 
- Training is trivial 
- Works with any number of classes 
- Easy to add more data 
- Few parameters: K  and Distance Metric





