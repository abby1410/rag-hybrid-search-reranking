# Findings: Hybrid Retrieval and Reranking

## 1. Why combine retrieval signals?

- **Dense/vector retrieval** represents text as embeddings and finds semantically similar passages, even when wording differs.
- **BM25** is a lexical ranking method based on term frequency and inverse document frequency. It is useful for exact terms, names, identifiers, and domain vocabulary.
- **Hybrid retrieval** combines lexical and vector signals. In Weaviate, `alpha=0` is BM25-oriented, `alpha=1` is vector-oriented, and intermediate values blend both. `0.5` is a starting point, not a universal optimum.

## 2. Why rerank?

The two-stage pattern first retrieves a candidate pool efficiently, then applies a CrossEncoder to score each query–passage pair jointly. This can improve fine-grained ordering at additional inference cost. CrossEncoder scores are ranking scores, not calibrated probabilities.

A reranker can only reorder candidates it receives. If first-stage retrieval misses a relevant passage, reranking cannot recover it.

## 3. Models in this project

- Embedding model: `sentence-transformers/all-MiniLM-L6-v2` (384-dimensional embeddings).
- Reranker: `cross-encoder/ms-marco-MiniLM-L-6-v2`.

The bi-encoder produces query and passage representations separately for efficient retrieval. The CrossEncoder evaluates query and passage together.

## 4. Current notebook limitations

The notebook is a learning prototype with only two hard-coded example chunks. Its example relevance labels refer to chunk IDs that are not in the indexed corpus. It includes an MRR helper, but does not yet run a valid controlled comparison across BM25-only, vector-only, hybrid, and hybrid-plus-reranking.

The ingestion approach can add duplicate objects when rerun unless deterministic IDs or an explicit reset strategy are added. No latency, cost, Recall@K, or nDCG@K results are reported. Therefore, this project makes no measured retrieval-quality uplift claim.

## 5. A fair next experiment

Create a representative corpus and a fixed query set with human-judged relevant chunk IDs. Run all methods on the same data and queries:

| Method | What it isolates |
|---|---|
| BM25-only | Lexical baseline |
| Vector-only | Dense semantic baseline |
| Hybrid | Combined first-stage retrieval |
| Hybrid + CrossEncoder | Effect of second-stage reranking |

Track:
- **MRR@K:** rank of the first relevant result.
- **Recall@K:** whether relevant passages appear in the top K.
- **nDCG@K:** ranking quality with graded relevance.
- **Latency:** retrieval and reranking separately; include p50/p95.
- **Cost:** embedding, database, and reranking compute.

Tune `alpha`, candidate depth (`top-N`), and final context depth (`top-K`) on a validation set. Keep a separate test set for final comparison. Record corpus size, query count, model versions, Weaviate configuration, and hardware.

## 6. Practical takeaways

- Hybrid search is a strong baseline when semantic intent and exact vocabulary both matter.
- Reranking is useful when the first stage can cheaply retrieve a broad candidate pool and the second stage can afford pairwise scoring.
- Candidate depth is a trade-off: too small can hide relevant passages; too large increases reranking latency and cost.
- Measure retrieval and reranking separately. A two-document demo is not a benchmark.
