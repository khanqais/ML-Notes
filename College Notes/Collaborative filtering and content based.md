Define Collaborative Filtering and explain how e-commerce platforms like Flipkart or Amazon use it to recommend products

**Collaborative filtering (CF)** is a recommendation technique that predicts what products a user may like by learning from the behavior of many users. It assumes that users with similar past actions—such as purchases, views, searches, ratings, carts, and wishlists—are likely to have similar future interests.[1][2]

## How it works

CF builds a **user–item interaction matrix**. Each row is a user, each column is a product, and each cell records an interaction such as a purchase, rating, click, add-to-cart, or product view. The system then finds similarities between users or between products and recommends items that similar users/items are associated with.[1][2]

Example:

| User | Phone case | Screen guard | Earbuds | Laptop bag |
|---|---:|---:|---:|---:|
| A | 1 | 1 | 0 | 0 |
| B | 1 | 1 | 1 | 0 |
| C | 1 | 0 | 1 | 1 |

If users A and B have very similar behavior, and B bought earbuds, CF may recommend earbuds to A.

## Types of collaborative filtering

| Type | Idea | E-commerce example |
|---|---|---|
| **User-based CF** | Find users with similar behavior, then recommend products they liked | “Customers like you also bought…” |
| **Item-based CF** | Find products that are frequently purchased, viewed, or rated together | “Customers who bought this item also bought…” |
| **Model-based CF** | Uses machine-learning models, such as matrix factorization, to learn hidden user and product preferences | Personalized homepage and product feeds |

Item-based CF is especially useful for e-commerce because product catalogs are huge and item relationships can be computed and updated efficiently. Amazon is widely associated with popularizing item-to-item collaborative filtering, which recommends products based on correlations between items rather than only similarities between customers.[3][4]

## How Flipkart or Amazon use it

When you browse or buy a product on Flipkart or Amazon, the platform collects behavioral signals such as:

- Product views and time spent on a product page
- Searches and clicked filters
- Products added to the cart or wishlist
- Purchases and returns
- Ratings and reviews
- Products purchased by other customers in the same session or over time

These signals are used to identify patterns such as “users who bought a phone often bought a screen guard and earbuds,” or “users who viewed running shoes also viewed sports socks.” CF then ranks likely products and displays them as personalized recommendations.[2][5]




---
tags:
  - recommender-systems
  - machine-learning
  - content-based-filtering
  - exam-notes
---

# Content-Based Product Recommendation (Extended Exam Notes)

## 1. Definition

**Content-based filtering (CBF)** recommends items that are **similar to items the user liked in the past**, by comparing **item features/properties** with a **user profile**. It does **not** need data from other users.[^1][^2]

> [!tip] One-line answer
> "Recommend items similar to what the user already liked, based on item attributes."

**Key idea:** *"If you liked this item, you will like items with similar content."*

---

## 2. Basic Terms

| Term | Meaning |
|---|---|
| **Item profile** | Feature vector describing a product (brand, price, RAM, genre, keywords) |
| **User profile** | Summary of the user's preferences, built from their interactions |
| **Feature extraction** | Converting attributes/text into numbers |
| **Similarity measure** | Function that scores how close two vectors are |
| **Top-N recommendation** | The N highest-scoring unseen items shown to the user |
| **Explicit feedback** | Ratings, likes, reviews |
| **Implicit feedback** | Clicks, views, purchases, add-to-cart, time spent |

---

## 3. Product Properties (Item Profile)

| Product type | Possible properties |
|---|---|
| Mobile phone | Brand, RAM, storage, camera MP, battery capacity, price, operating system |
| Shoes | Brand, size, color, material, category, price, style |
| Laptop | Processor, RAM, storage, screen size, GPU, brand, price |
| Book | Author, genre, language, publisher, price |
| Movie | Genre, director, actors, language, year |

Attributes are converted into a numerical feature vector:

$$
[\text{brand},\ \text{category},\ \text{price},\ \text{RAM},\ \text{battery}]
$$

### Types of item data

- **Structured data:** well-defined attributes (price, RAM, brand, size).
- **Unstructured data:** free text (descriptions, reviews), images, audio.
- **Semi-structured data:** a mix of both (tags plus a description).

---

## 4. Architecture of a Content-Based System

Three main components:

1. **Content Analyzer**
   - Takes item data (text, attributes) and extracts features.
   - Produces the item representation (e.g., TF-IDF vector).
2. **Profile Learner**
   - Builds and updates the user profile from the user's feedback.
3. **Filtering Component**
   - Matches the user profile against item profiles.
   - Ranks items and returns the top-N recommendations.

```text
Item data --> Content Analyzer --> Item profiles ----+
                                                      +--> Filtering Component --> Top-N recommendations
User feedback --> Profile Learner --> User profile ---+
```

---

## 5. Steps of Recommendation

1. **Create product profiles** from product attributes and descriptions.
2. **Create a user profile** from interactions: purchases, views, searches, wishlist items, ratings, and add-to-cart actions.
3. **Compare profiles** using a similarity measure, commonly cosine similarity.
4. **Rank products** by similarity score.
5. **Show the top-ranked products** as recommendations.[^1][^3]

---

## 6. Feature Extraction Techniques

| Technique | Used for | Idea |
|---|---|---|
| **One-hot encoding** | Categorical data (brand, color) | One binary column per category value |
| **Normalization / scaling** | Numeric data (price, RAM) | Put values on the same range, e.g. $[0,1]$ |
| **Bag of Words (BoW)** | Text | Count of each word |
| **TF-IDF** | Text | Weight words by importance |
| **Word embeddings** (Word2Vec, GloVe) | Text | Dense vectors that capture meaning |
| **Transformer embeddings** (BERT) | Text | Context-aware vectors |
| **CNN features** | Images | Visual features, e.g. clothing style |

### TF-IDF (important for exams)

$$
\text{TF-IDF}(t,d) = \text{TF}(t,d) \times \text{IDF}(t)
$$

$$
\text{TF}(t,d) = \frac{\text{count of term } t \text{ in document } d}{\text{total terms in } d}
$$

$$
\text{IDF}(t) = \log\frac{N}{\text{df}(t)}
$$

where:

- $N$ is the total number of documents (products)
- $\text{df}(t)$ is the number of documents containing term $t$

**Meaning:** a word that is frequent in one product description but rare across all products gets a **high** weight. Common words (like "the") get a **low** weight.

---

## 7. Building the User Profile

**Method 1: Average of liked item vectors**

$$
\vec{u} = \frac{1}{n}\sum_{i=1}^{n} \vec{v}_i
$$

**Method 2: Rating-weighted average**

$$
\vec{u} = \frac{\sum_i r_i \, \vec{v}_i}{\sum_i |r_i|}
$$

where $r_i$ is the user's rating for item $i$ and $\vec{v}_i$ is the item's feature vector.

**Method 3: Learn a classifier** (Naive Bayes, Decision Tree, Logistic Regression) per user, using liked/disliked items as training data.

**Other points:**
- Implicit signals can be weighted (purchase > add-to-cart > click).
- Recent interactions can be given more weight (time decay).

---

## 8. Similarity Measures

### Cosine similarity (most common)

$$
\text{sim}(A,B)=\frac{A \cdot B}{\lVert A \rVert \, \lVert B \rVert}
$$

Range is 0 to 1 for non-negative features. A higher value means more similar.[^2][^4]

### Euclidean distance

$$
d(A,B)=\sqrt{\sum_{i=1}^{n}(A_i-B_i)^2}
$$

A **smaller** distance means more similar.

### Jaccard similarity (for sets or tags)

$$
J(A,B)=\frac{|A \cap B|}{|A \cup B|}
$$

### Pearson correlation

$$
r=\frac{\sum (A_i-\bar{A})(B_i-\bar{B})}{\sqrt{\sum (A_i-\bar{A})^2}\sqrt{\sum (B_i-\bar{B})^2}}
$$

### Predicted score for a user

$$
\text{score}(u, i) = \cos(\vec{u}, \vec{v}_i)
$$

Items with the highest score are recommended.

---

## 9. Example: Mobile Phone Recommendation

The user views or buys this phone:

```text
Phone A:
Brand: Samsung
Category: Smartphone
RAM: 8 GB
Storage: 128 GB
Battery: 5000 mAh
Camera: 108 MP
Price: ₹20,000
```

User preference profile built from this interaction:

```text
Brand preference: Samsung
Category: Smartphone
Preferred RAM: 8 GB
Preferred storage: 128 GB
Preferred battery: around 5000 mAh
Preferred price: around ₹20,000
```

Candidate products:

| Product | Brand | RAM | Storage | Battery | Price | Similarity |
|---|---|---:|---:|---:|---:|---:|
| Phone B | Samsung | 8 GB | 128 GB | 5000 mAh | ₹19,500 | High |
| Phone C | Samsung | 6 GB | 128 GB | 5000 mAh | ₹18,000 | Medium |
| Phone D | Xiaomi | 8 GB | 128 GB | 5000 mAh | ₹20,000 | Medium |
| Phone E | iPhone | 6 GB | 64 GB | 3000 mAh | ₹60,000 | Low |

**Phone B** is recommended first because it matches brand, RAM, storage, battery, and price range.[^1][^3]

### Numerical cosine similarity example

Use binary features: `[Samsung, Xiaomi, 8GB RAM, 128GB storage, 5000mAh]`

| Product | Vector |
|---|---|
| Phone A (liked) | $[1,0,1,1,1]$ |
| Phone B | $[1,0,1,1,1]$ |
| Phone C | $[1,0,0,1,1]$ |
| Phone D | $[0,1,1,1,1]$ |

**Phone A vs Phone D:**

$$
A\cdot D = 0+0+1+1+1 = 3,\quad \lVert A\rVert = \sqrt{4}=2,\quad \lVert D\rVert=\sqrt{4}=2
$$

$$
\cos(A,D)=\frac{3}{2\times 2}=0.75
$$

**Phone A vs Phone C:**

$$
A\cdot C = 3,\quad \lVert C\rVert=\sqrt{3},\quad \cos(A,C)=\frac{3}{2\sqrt{3}}\approx 0.866
$$

**Phone A vs Phone B:** $\cos(A,B)=1$ (identical features).

**Ranking:** Phone B (1.00) > Phone C (0.87) > Phone D (0.75)

---

## 10. Text-Based Similarity Example

```text
Product A: "Wireless Bluetooth earbuds with noise cancellation and 30-hour battery life"
Product B: "Bluetooth neckband earphones with fast charging and long battery backup"
Product C: "Wired gaming mouse with RGB lighting"
```

Keywords such as `wireless`, `Bluetooth`, `earbuds`, `noise cancellation`, and `battery` are extracted. **TF-IDF** turns them into vectors, then cosine similarity finds the closest products. Product B is more likely to be recommended after Product A than Product C.[^1][^4]

---

## 11. Real-World Example (Flipkart / Amazon)

If you view a **Nike running shoe**, the system may recommend:

- Another Nike running shoe
- A running shoe with similar cushioning and price
- Sports shoes in the same size and color range
- Running accessories such as socks or a water bottle

The recommendation is based on the **product's own features** (brand, category, material, size, color, price, description), not primarily on other customers' behavior.[^5][^6]

---

## 12. Advantages

- **No data from other users needed**, so there are no privacy issues from sharing user data.
- **No item cold-start problem:** a new product can be recommended as soon as its features are known.
- **Works for users with unique tastes**, since recommendations are not based on a crowd.
- **Transparent and explainable:** "Recommended because it is also a Samsung phone with 8 GB RAM."
- **Personalized** to each user.
- **Can recommend unpopular or niche items** that few users have rated.
- **Easy to implement** for small systems.
- **Users control the profile:** changing preferences changes recommendations directly.

## 13. Limitations / Disadvantages

| Limitation | Explanation |
|---|---|
| **Over-specialization / filter bubble** | Keeps recommending very similar items, with little variety or surprise |
| **Limited serendipity** | Rarely suggests unexpected but useful items |
| **User cold-start problem** | A new user with no history has no profile |
| **Feature dependency** | Quality depends on how good and complete the item attributes are |
| **Hard to analyze some content** | Images, audio, and video need complex feature extraction |
| **Does not capture quality or popularity** | Two similar items may differ greatly in quality |
| **Cannot use "wisdom of the crowd"** | Ignores what similar users liked |
| **Feature engineering effort** | Manual selection and weighting of features may be required |

A major limitation is the **filter bubble**: if a user buys only Samsung phones, the system may keep recommending similar Samsung phones and may never introduce a better product from another brand.[^7][^8]

---

## 14. Content-Based vs Collaborative Filtering

| Aspect | Content-Based | Collaborative Filtering |
|---|---|---|
| Based on | Item features + user's own history | Behavior of many users |
| Needs other users' data | No | Yes |
| New item (cold start) | Handled well | Problem (no ratings yet) |
| New user (cold start) | Problem | Problem |
| Diversity / serendipity | Low | Higher |
| Explainability | High | Lower |
| Needs item metadata | Yes | No |
| Popularity bias | Low | High |
| Scalability | Good | Can be heavy with many users |
| Example | "Similar to the phone you viewed" | "Customers who bought this also bought..." |

---

## 15. Hybrid Recommender Systems

Real platforms (Amazon, Netflix, Flipkart) **combine** content-based and collaborative filtering, plus business rules, to balance relevance, novelty, popularity, and inventory availability.[^7][^8]

Common hybrid approaches:

- **Weighted:** combine scores from both methods.
- **Switching:** use CBF for new items or users, CF otherwise.
- **Mixed:** show both lists side by side.
- **Feature combination:** feed collaborative signals as features into a content model.
- **Cascade:** one method filters, the other refines the ranking.

---

## 16. Machine Learning Approaches in CBF

- **K-Nearest Neighbors (KNN):** find the items nearest to those the user liked.
- **Naive Bayes:** classify items as like or dislike from their words/features.
- **Decision trees / Random forest:** learn rules for what the user likes.
- **Logistic regression / SVM:** binary like/dislike classifier.
- **Neural networks / deep learning:** learn embeddings from text and images.
- **Rocchio algorithm:** classic relevance-feedback method that moves the user profile toward liked items and away from disliked ones.

---

## 17. Evaluation Metrics

| Metric | Meaning |
|---|---|
| **Precision@K** | Fraction of the top-K recommended items that are relevant |
| **Recall@K** | Fraction of all relevant items that appear in the top-K |
| **F1-score** | Harmonic mean of precision and recall |
| **MAP** | Mean Average Precision across users |
| **NDCG** | Rewards relevant items placed higher in the ranking |
| **RMSE / MAE** | Error between predicted and actual ratings |
| **Coverage** | Share of catalog items the system can recommend |
| **Diversity** | How different the recommended items are from each other |
| **Novelty / Serendipity** | How new and unexpected the recommendations are |

$$
\text{Precision@K}=\frac{\text{relevant items in top-}K}{K}
$$

$$
\text{Recall@K}=\frac{\text{relevant items in top-}K}{\text{total relevant items}}
$$

---

## 18. Applications

- **E-commerce:** similar products (Amazon, Flipkart)
- **Movies / music / video:** Netflix, Spotify, YouTube (genre, artist, tags)
- **News and articles:** similar topics and keywords
- **Books:** same author or genre
- **Jobs:** match resumes with job descriptions
- **Online courses:** similar topics and skill levels
- **Fashion:** visual similarity using image features

---

## 19. Ways to Overcome the Limitations

- Add **diversity** to the ranking (re-rank to reduce near-duplicates).
- Add **random or exploratory** recommendations to increase serendipity.
- Use a **hybrid** approach with collaborative filtering.
- Ask new users for **initial preferences** (onboarding questions) to handle user cold start.
- Use **deep learning embeddings** for richer content features.
- Use **popularity-based** recommendations for brand-new users.

---

 