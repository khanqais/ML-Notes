Binary classification means predicting **one of two classes** ,for example, whether a transaction is _fraud_ or _legitimate_. Evaluation metrics tell you how well those predictions match the actual answers, and what kinds of mistakes the model makes

Suppose a model checks 100 transactions. We call _fraud_ the positive class and _legitimate_ the negative class
![[Pasted image 20260928152552.png|510]]
The four labels mean:
- **TP (true positive):** Fraud correctly flagged=15 transactions.
- **TN (true negative):** Legitimate transaction correctly left alone=70.
- **FP (false positive):** Legitimate transaction wrongly flagged as fraud=10.
- **FN (false negative):** Fraud missed by the model=5.

## Accuracy

**Question:** “Out of _all_ transactions, how many did the model classify correctly?”![[Pasted image 20260928153047.png|457]]
**When it helps:** Accuracy is an easy overall measure, particularly when the classes are reasonably balanced and the two kinds of error have similar consequences. It can be misleading when one class is rare. Imagine 1 fraud among 100 transactions: a model that calls _everything legitimate_ has 99% accuracy but catches no fraud

## Precision

**Question:** “When the model _says fraud_, how often is it right?”
![[Pasted image 20260928153915.png|455]]
The model raised 25 fraud alerts. Only 15 were real fraud, so **6 out of every 10 alerts** were correct. The remaining 10 were false positives—legitimate transactions flagged by mistake

## Recall

**Question:** “Of the transactions that _really were fraud_, how many did the model find?”
![[Pasted image 20260928154741.png]]
There were 20 actual frauds. The model caught 15 and missed 5, so it found **3 out of every 4 frauds**. Recall is also called _sensitivity_ or _true positive rate_.developers.google+1

**When it matters:** Prioritize recall when missing a positive case is costly—for example, when fraud that slips through causes significant loss
**Remember the denominator:** Recall starts with **actual positives**: TP+FNTP+FNTP+FN. Think, “Did I find the positives that were really there?”


## Precision versus recall

They sound similar, but their starting points differ:
- **Precision:** Of the 25 _flagged_ transactions, 15 were fraud → **60%**.
    
- **Recall:** Of the 20 _actual frauds_, 15 were flagged → **75%**.


## F1 score

**Question:** “Can I combine precision and recall into one score?”
![[Pasted image 20260928154907.png|529]]