Decision Trees are easy to build, easy to use and easy to interpret......but in practice they are not that awesome.
Trees have one aspect that prevents them from being the ideal tool for predictive learning, nam ely inaccuracy. In other words, they work great with the data used to create them, but they are not flexible when it comes to classifying new samples. 
![[Pasted image 20261111225810.png]]
In the bootstrapped dataset we can choose duplicates data entry also and leave some of them from the original dataset
![[Pasted image 20261111225844.png|365]]

![[Pasted image 20261111225938.png]]
![[Pasted image 20261111225954.png]]![[Pasted image 20261111230045.png]]![[Pasted image 20261111230108.png]]Ideally, you’d do this 100’s of times, but we only have space to show 6... but you get the idea.
Using a bootstrapped sample and considering only a subset of the variables at each step results in a wide variety of trees. The variety is what makes random forests more effective than individual decision trees.

![[Pasted image 20261111230248.png|396]]
The first tree says “Yes”...
The second tree also says “Yes”... 
![[Pasted image 20261111230339.png|594]]
In this case, “Yes” received the most votes, so we will conclude that this patient has heart disease.

**Bootstrapping** the data plus using the aggregate to make a decision is called “Bagging” 


We allowed duplicate entries in | the bootstrapped dataset... 
As a result, some entry was not included in the bootstrapped dataset.
Typically, about 1/3 of the original | data does notend up in the bootstrapped dataset.
(psst... if the original dataset were larger, we’d have more than just 1 entry over here...) 

Ultimately, we can measure how accurate our random forest is by the proportion of Out-Of-Bag samples that were correctly classified by the Random Forest..
The proportion of Out-Of-Bag samples that were incorrectly classified is the “Out-Of-Bag Error”

Remember when we built our first tree and we only ~~ used 2 variables (columns of data) to make a decision at each step? 
Now we can compare the Out-Of-Bag error for a random forest built using only 2 variables per step