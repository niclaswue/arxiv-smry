---
title: "test"
date: 2023-01-10T11:02:49.000Z
author: "Mitko Gospodinov, Sean MacAvaney, Craig Macdonald"
weight: 2
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "Important disclaimer: the following content is AI-generated, please make sure to fact check the presented information by reading the full paper."
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: false
ShowBreadCrumbs: false
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: false
cover:
    image: "thumbnails/2206.13404.jpg" # image path/url
    alt: "Doc2Query--: When Less is More" # alt text
    caption: "The full paper is available [here](https://arxiv.org/abs/2301.03266)." # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
---

# Link to paper
The full paper is available [here](https://arxiv.org/abs/2301.03266).


# Abstract
- Doc2Query is prone to hallucination, which harms retrieval effectiveness and inflates the index size
- Filtering out harmful queries prior to indexing can improve retrieval effectiveness by up to 16%

# Paper Content

## Introduction
- Neural network models, particularly those based on contextualised language models, have been shown to improve search effectiveness
- In this work, we focus on one of these first-stage approaches: Doc2Query
- This approach trains a sequence-to-sequence model (e.g., T5 [33]) to predict queries that may be relevant to a particular text.
- Then, when indexing, this model is used to expand the document by generating a collection of queries and appending them to the document.
- Though computationally expensive at index time [34], this approach has been shown to be remarkably effective even when retrieving using simple lexical models like BM25 [28].
- Numerous works have shown that the approach can produce a high-quality pool of results that are effective for subsequent stages in the ranking pipeline [19,20,23,40]
- However, sequence-to-sequence models are well-known to be prone to generate content that does not reflect the input text -a defect known in literature as "hallucination" [25]
- We find that existing Doc2Query models are no exception.
- In this example, we see that many of the generated queries cannot actually be answered by the source passage (score ≤ 1).
- Original Passage: Barley (Hordeum vulgare L.), a member of the grass family, is a major cereal grain. It was one of the first cultivated grains and is now grown widely. Barley grain is a staple in Tibetan cuisine and was eaten widely by peasants in Medieval Europe.
- Barley has also been used as animal fodder, as a source of fermentable material for beer and certain distilled beverages, and as a component of various health foods.
- Based on this observation, we hypothesise that retrieval performance of Doc2Query would improve if hallucinated queries were removed.
- In this paper, we conduct experiments where we apply a new filtering phase that aims to remove poor queries prior to indexing.
- Given that this approach removes queries, we call the approach Doc2Query--(Doc2Query-minus-minus).
- Rather than training a new model for this task, we identify that relevance models are already fit for this purpose: they estimate how relevant a passage is to a query.
- We therefore explore filtering strategies that make use of existing neural relevance models.
- Through experimentation on the MS MARCO dataset, we find that our filtering approach can improve the retrieval effectiveness of indexes built using Doc2Query--by up to 16%; less can indeed be more.
- Meanwhile, filtering naturally reduces the index size, lowering storage and query-time computational costs.
- Finally, we conduct an exploration of the index-time overheads introduced by the filtering process and conclude that the gains from filtering more than make up for the additional time spent generating more queries.
- The approach also has a positive impact on the environmental costs of applying Doc2Query; the same retrieval effectiveness can be achieved with only about a third of the computational cost when indexing.

## Related Work
- The classical lexical mismatch problem is a key one in information retrieval
- Query reformulation -including stemming, query expansion models (e.g. Rocchio, Bo1 [1], RM3 [12]) -and document expansion [9,30,35] have been popular, as they avoid the costs associated with making additional processing for each document needed for document expansion.
- However, query expansion may result in reduced performance [11], as queries are typically short and the necessary evidence to understand the context of the user is limited.
- The application of latent representations of queries and documents, such as using latent semantic indexing [8] allow retrieval using not to be driven directly by lexical signals.
- More recently, transformer-based language models (such as BERT [6]) have resulted in representations of text where the contextualised meaning of words are accounted for.
- In dense retrieval, queries and documents are represented in embeddings spaces [14,37], often facilitated by Approximate Nearest Neighbour (ANN) data structures [13].
- However, even when using ANN, retrieval can still be inefficient or insufficiently effective [15].
- Others have explored approaches for augmenting lexical representations with additional terms that may be relevant.
- In this work, we explore Doc2Query [29], which uses a sequence-to-sequence model that maps a document to queries that it might be able to answer.
- By appending these generated queries to a document's content before indexing, the document is more likely to be retrieved for user queries when using a model like BM25.
- An alternative style of document expansion, proposed by MacAvaney et al. [19] and since used by several other models (e.g., [10,39,40]), uses the built-in Masked Language Modelling (MLM) mechanism.
- MLM expansion generates individual tokens to append to the document as a bag of words (rather than as a sequence).
- Although MLM expansion is also prone to hallucination,2 the bag-of-words nature of MLM expansion means that individual expansion tokens may not have sufficient context to apply filtering effectively.
- We therefore focus only on sequence-style expansion and leave the exploration of MLM expansion for future work.

## Experimental Setup
- We conduct experiments to answer the following research questions:
- RQ1 Does Doc2Query--improve the effectiveness of document expansion?
- RQ2 What are the trade-offs in terms of effectiveness, efficiency, and storage when using Doc2Query--?
- Datasets and Measures.
- We conduct tests using the MS MARCO [26] v1 passage corpus.
- We use five test collections: 3 (1) the MS MARCO Dev (small) collection, consisting of 6,980 queries (1.1 qrels/query); (2) the Dev2 collection, consisting of 4,281 (1.1 qrels/query); (3) the MS MARCO Eval set, consisting of 6,837 queries (held-out leaderboard set); (4/5) the TREC DL'19/'20 collections, consisting of 43/54 queries (215/211 qrels/query).
- We evaluate using the official task evaluation measures: Reciprocal Rank at 10 (RR@10) for Dev/Dev2/Eval, nDCG@10 for DL'19/'20.
- We tune systems4 on Dev, leaving the remaining collections as held-out test sets.
- We use the T5 Doc2Query model from Nogueira and Lin [28], making use of the inferred queries released by the authors (80 per passage).
- To the best of our knowledge, this is the highest-performing Doc2Query model available.
- We consider three neural relevance models for filtering: ELECTRA 5 [31], MonoT56 [32], and TCT-ColBERT7 [16], covering two strong cross-encoder models and one strong bi-encoder model.
- We also explored filters that use the probabilities from the generation process itself but found them to be ineffective and therefore omit these results due to space constraints.
- We use the PyTerrier toolkit [22] with a PISA [24,17] index to conduct our experiments.
- We deploy PISA's Block-Max WAND [7] implementation for BM25 retrieval.
- Inference was conducted on an NVIDIA 3090 GPU.
- Evaluation was conducted using the ir-measures package [18].

## Results
- RQ1: Relevance filtering can improve the retrieval of Doc2Query models
- Table 1 compares the effectiveness of Doc2Query with various filters
- We observe that all the filters significantly improve the retrieval effectiveness on the Dev and Dev2 datasets at both n = 40 and n = 80
- We also observe a large boost in performance on the Eval dataset
- 8 Though the differences in DL'19 and DL'20 appear to be considerable (e.g., 0.627 to 0.670), these differences are not statistically significant.
- Digging a little deeper, Figure 2 shows the retrieval effectiveness of Doc2Query with various numbers of generated queries (in dotted black) and the corresponding performance when filtering using the top-performing ELECTRA scorer (in solid blue). We observe that performing relevance filtering at each value of n improves the retrieval effectiveness.
- For instance, keeping only 30% of expansion queries at n = 80, performance is increased from 0.279 to 0.323 -a 16% improvement.
- In aggregate, results from Table 1 and Figure 2 answer RQ1: Doc2Query-filtering can significantly improve the retrieval effectiveness of Doc2Query across various scoring models, numbers of generated queries (n) and thresholds (p).
- Next, we explore the trade-offs in terms of effectiveness, efficiency, and storage when using Doc2Query--
- Table 1 includes the mean response time and index sizes for each of the settings. As expected, filtering reduces the index size since fewer terms are stored. For the best-performing setting (n = 80 with ELECTRA filter), this amounts to a 48% reduction in index size (1.41 GB down to 0.95 GB). Naturally, such a reduction has an impact on query processing time as well; it yields a 30% reduction in mean response time (30ms down to 23ms).
- Doc2Query--filtering adds substantial cost an indexing time, mostly due to scoring each of the generated queries.
- Table 2 reports the cost (in hours of GPU time) of the generation and filtering phases. We observe that ELECTRA filtering can yield up to a 78% increase in GPU time (n = 10). However, we find that the improved effectiveness makes up for this cost.
- To demonstrate this, we allocate the time spent filtering to generating additional queries for each passage. For instance, the 15 hours spent scoring n = 5 queries could instead be spent generating 6 more queries per passage (for a total of n = 11). We find that when comparing against an unfiltered n that closely approximates the total time when
- Filtering, the filtered results consistently yield significantly higher retrieval effectiveness.
- As the computational budget increases, so does the margin between Doc2Query and Doc2Query--, from 4% at 34 hours up to 12% at 216 hours.
- From the opposite perspective, Doc2Query consumes 2.9× or more GPU time than Doc2Query--to achieve similar effectiveness (n = 13 with no filter vs. n = 5 with ELECTRA filter). Since the effectiveness of Doc2Query flattens out between n = 40 and n = 80 (as seen in Figure 2), it likely requires a massive amount of additional compute to reach the effectiveness of Doc2Query-at n ≥ 10, if that effectiveness is achievable at all.
- These comparisons show that if a deployment is targeting a certain level of effectiveness (rather than a target compute budget), Doc2Query--is also preferable to Doc2Query.

## Conclusions
- The work demonstrated that there are untapped advantages in generating naturallanguage for document expansion.
- Specifically, we presented Doc2Query--, which is a new approach for improving the effectiveness and efficiency of the Doc2Query model by filtering out the least relevant queries.
- We observed that a 16% improvement in retrieval effectiveness can be achieved, while reducing the index size by 48% and mean query execution time by 30%.
- The technique of filtering text generated from language models using relevance scoring is ripe for future work.
