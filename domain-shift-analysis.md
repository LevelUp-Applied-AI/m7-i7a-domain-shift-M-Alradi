# Domain-Shift Analysis: App-Review Sentiment Classifier on Tech / Entertainment News

## Prediction distribution

| Label    | Count | Share |
|----------|------:|------:|
| negative |   425 | 41.1% |
| neutral  |   488 | 47.2% |
| positive |   120 | 11.6% |

The positive class is heavily suppressed — only 1 in 9 articles gets a positive label. App-review datasets tend to be more balanced, so this skew is an early sign the model doesn't generalize well to news prose.

## Confidence distribution

| Statistic               | Value |
|-------------------------|------:|
| Mean confidence         | 0.631 |
| Median confidence       | 0.609 |
| Proportion > 0.90       |  2.4% |
| Proportion < 0.60       | 47.8% |

Nearly half of all predictions fall below 0.60 — the model is uncertain on the majority of the corpus. The small cluster above 0.90 is almost entirely `negative` predictions on articles about crime, scandal, or political conflict.

## Five qualitative examples

**1. NEWS_0035 — over-confident false negative**
> "(CNN) — Buy a $175,000 package to attend the Oscars and you might buy yourself trouble, lawyers for the Academy Awards warn…"
- Predicted: `negative`, 0.911
- A neutral legal-warning news story. The phrase "buy yourself trouble" pattern-matches to complaint language in one-star reviews. The model is 91% confident but the article has no negative sentiment toward any product. **Pattern:** cautionary/legal phrasing → spurious high-confidence negative.

**2. NEWS_0184 — over-confident false positive**
> "(CNN) — For video game fans, many of the gadgets and accessories designed to support today's games are as unique and imaginative as the games themselves…"
- Predicted: `positive`, 0.881
- A descriptive gadget listicle. Words like "unique" and "imaginative" match five-star review language. The model treats editorial enthusiasm as user satisfaction. **Pattern:** product-enthusiasm vocabulary → spurious high-confidence positive.

**3. NEWS_0004 — hedging with negative-class bias**
> "(CNN) — Forbes' list of the world's wealthy has named Warren Buffett the richest person on the planet…"
- Predicted: `negative`, 0.480
- A factual wealth-ranking story with no sentiment signal. The model outputs near-random confidence (0.48) yet still defaults to `negative` rather than `neutral`. **Pattern:** when uncertain, the model falls back to `negative` rather than abstaining.

**4. NEWS_0802 — reasonable neutral prediction**
> "(WIRED) — Apple's decision to not include a camera in the new iPod Touch is somewhat surprising…"
- Predicted: `neutral`, 0.797
- An analytical product-criticism piece with measured language. This is one of the better-calibrated predictions in the dataset — the article's register happens to resemble a balanced review. **Pattern:** the model works only when news prose accidentally mimics review prose in structure.

**5. NEWS_0909 — systematic bias on political conflict**
> "(CNN) — Two words, delivered with index finger punctuating the air and directed at the president of the United States, made a little-known South Carolina congressman one of the most controversial men…"
- Predicted: `negative`, 0.946
- A neutral report on the "You lie!" incident. The model is 95% confident, reacting entirely to confrontational political vocabulary. **Pattern:** any article involving conflict, scandal, or arrest gets assigned `negative` at high confidence — the strongest and most consistent bias in the corpus.

## Engineering judgment

I would not ship this model for tech/entertainment news sentiment classification. The numbers tell a concrete story: 47.8% of predictions fall below 0.60 confidence, so any reasonable threshold (say, 0.75) would reject more than half the corpus as too uncertain to use — leaving fewer than 200 actionable predictions out of 1,033. The positive class is nearly absent (11.6%), so any feature relying on positive signal (PR monitoring, recommendation triggers) would be essentially blind. The 25 high-confidence predictions (>0.90) are almost all false negatives driven by crime or conflict vocabulary, not genuine negative product sentiment. In a real pipeline, that means neutral news articles about product launches or executive statements could be routed to a crisis-response queue, eroding trust in the system quickly. The root problem is that app reviews and news articles share vocabulary but differ completely in intent and structure — the model has no way to distinguish "this text contains negative words" from "a user is unhappy with a product." Fine-tuning on labeled news data and enforcing a minimum confidence threshold of 0.75 (with human review below that) would be the minimum bar before any deployment.