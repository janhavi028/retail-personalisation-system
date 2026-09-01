# Recommender System Notes

## 1. Recommendation Problem

### Explicit vs Implicit Feedback

Explicit feedback:
- Ratings
- Likes/dislikes
- User-provided preferences

Implicit feedback:
- Purchases
- Clicks
- Views
- Add-to-basket events

Our dataset contains purchase interactions, so this is an
implicit-feedback recommendation problem.

An unobserved customer–article interaction does not necessarily
mean the customer dislikes the article. They may simply never
have encountered it.


## 2. Main Recommender Approaches

### Popularity-based
Recommends products based on aggregate behaviour.

Advantages:
- Simple
- Handles customers without interaction history
- Useful baseline

Limitations:
- No personalisation
- Can reinforce popularity bias


### Collaborative Filtering
Uses patterns in user-item interactions to infer preferences.

Examples:
- User-user collaborative filtering
- Item-item collaborative filtering
- Matrix factorisation such as ALS


### Content-based
Uses item/customer attributes rather than relying solely on
interaction patterns.

Useful for:
- New items
- Sparse interaction histories
- Incorporating product metadata


## 3. Production Recommendation Architecture

Candidate generation
        ↓
Ranking
        ↓
Re-ranking
        ↓
Top-K recommendations

Multiple candidate generators can be combined, for example:
- popularity
- ALS
- item-item similarity
- two-tower retrieval