
We follow the [eBay blog](https://www.ebayinc.com/stories/blogs/tech/measuring-search-relevance/) by referring to the process of gathering data on search relevance as *relevance judgement*, which involves collection human judgement about the *relevance* of search terms. The basic task is as follows: we present a judge (a user) with a *search hit* for a given *search query*, and ask the judge the asses how relevant the result is on a four point scale.

The judgement process is subjective, and the notion of relevance may drift over time. Therefore, it is desirable to have a system of relevance ranking which learns what users deem relevant *over time*. The purpose of this document is to outline the metric through which we measure search relevance at any given time. To do this, we opt for a Normalised Discounted Cumulative Gain algorithm, which although seems a mouthful is just a common-sense measure.

Suppose that on our four-point scale, we give a 0 score for a irrelevant result, 1 for partially relevant, 2 for relevant, and 3 for perfect. If the first four results returned are judged to be relevant, irrelevant, perfect, and relevant, then the cumulative gain after these four results is simply the sum of the scores for each result: 2 + 0 + 3 + 2 = 7.

| Rank | Judgement (Gain) | Cumulative Gain |
|------|------------------|-----------------|
| 1    | 2                | 2               |
| 2    | 0                | 2               |
| 3    | 3                | 5               |
| 4    | 2                | 7               |

For the discounted part in NDCG, we know that the first result (rank 1) is more imporant than the second (rank 2), so we add in a weighting by summing the score and dividing by the rank:

| Rank | Judgement (Gain) | Discounted Gain | Discounted Cumulative Gain (DCG) |
|------|------------------|-----------------|----------------------------------|
| 1    | 2                | 2/1             | 2                                |
| 2    | 0                | 0/2             | 2                                |
| 3    | 3                | 3/3             | 3                                |
| 4    | 2                | 2/4             | 3.5                              |

The normalised part allows us to compute DCG values for different queries, as it is unfair to compare DGC values across queries. Normalization works like this: we figure out what the best possible score is given the results we’ve seen so far. In our previous example, we had 2, then 0, 3, and a 2. The best arrangement of these same results would have been: 3, 2, 2, 0, that is, if the “great” result had been ranked first, followed by the two “relevant” ones, and then the “irrelevant”. This best ranking would have a DCG score of 3 / 1 + 2 / 2 + 2 / 3 + 0 / 4 = 4.67. This is known as the “ideal DCG,” or iDCG.  Our NDCG is the score we got (3.5) divided by the ideal DCG (4.67), or 3.5 / 4.67 = 0.75. Now we can compare scores across queries, since we’re comparing percentages of the best possible arrangements and not the raw scores. It has some drawbacks, but we’ll duck those for now.

| Rank | Judgement (Gain) | Discounted Gain | Discounted Cumulative Gain (DCG) | Ideal Discounted Gain | Ideal Discounted Cumulative Gain (iDGC) | Normalised Discounted Cumulative Gain (NDGC) |
|------|------------------|-----------------|----------------------------------|-----------------------|-----------------------------------------|----------------------------------------------|
| 1    | 2                | 2/1             | 2                                | 3/1                   | 3.0                                     | 0.67                                         |
| 2    | 0                | 0/2             | 2                                | 2/2                   | 4.0                                     | 0.5                                          |
| 3    | 3                | 3/3             | 3                                | 2/3                   | 4.67                                    | 0.64                                         |
| 4    | 2                | 2/4             | 3.5                              | 0/4                   | 4.67                                    | 0.75                                         |

Once we've computed NDCG values for each distinct query, we can average over all queries to quantity the overall relevance of our search engine.