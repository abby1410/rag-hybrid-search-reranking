# Hybrid Search and Reranking for RAG

A practical notebook exploring a two-stage retrieval pipeline: **hybrid search in Weaviate Cloud**, followed by **cross-encoder reranking with Hugging Face Sentence Transformers**.

> **Project status:** learning prototype. The notebook currently uses a tiny, two-chunk example corpus. It demonstrates the retrieval flow, but it does not establish a reliable quality improvement or a meaningful benchmark result.

## What this project demonstrates

- Dense passage embeddings with `sentence-transformers/all-MiniLM-L6-v2` (384 dimensions).
- Keyword retrieval through BM25 and vector retrieval through Weaviate hybrid search.
- The effect of Weaviate's `alpha` parameter on the lexical/vector blend.
- Second-stage scoring with `cross-encoder/ms-marco-MiniLM-L-6-v2`.
- Why retrieval candidate depth, relevance labels, ranking metrics, latency, and cost matter when evaluating RAG retrieval.

## Architecture

```text
User query
   ├── Dense query embedding ──────────────┐
   └── BM25 lexical query ─────────────────┤
                                           ▼
                                Weaviate hybrid retrieval
                                  (top-N candidates)
                                           │
                                           ▼
                             Hugging Face CrossEncoder
                             (query + passage pair scores)
                                           │
                                           ▼
                                  Reranked top-K
                                           │
                                           ▼
                                  RAG context (next step)
```

The embedding model is a bi-encoder: query and passage representations are produced separately, making it suitable for fast first-stage retrieval. The CrossEncoder reads each query–passage pair jointly, which can improve fine-grained relevance ordering at a higher per-candidate inference cost. Its scores are ranking scores, not calibrated probabilities.

## Run in Google Colab

1. Open the notebook in Google Colab.
2. In Colab, open **Secrets** and add `HF_TOKEN`, `WEAVIATE_URL`, and `WEAVIATE_API_KEY`.
3. Enable notebook access to those secrets.
4. Run the notebook from top to bottom.

You will need a Hugging Face token and a reachable Weaviate Cloud cluster with a valid API key.

## Important limitations

- The current corpus contains only two example chunks.
- The evaluation examples refer to chunk IDs that are not present in that corpus. Correct the relevance labels before calculating metrics.
- The notebook defines an MRR helper, but does not yet provide a valid, repeatable comparison of BM25-only, vector-only, hybrid, and hybrid-plus-reranking.
- Re-running ingestion may add duplicate objects because deterministic object IDs or a reset strategy are not yet implemented.
- No latency, cost, Recall@K, or nDCG@K results are reported here.

See [`FINDINGS.md`](FINDINGS.md) for interpretation and an evaluation plan. **No retrieval-quality uplift is claimed.**

## Security

Never commit `.env` files, API keys, access tokens, private documents, or customer data. If a credential was ever exposed in a public commit or shared notebook, revoke/rotate it at the provider. Inspect the staged diff before publishing.

## Next steps

1. Replace the two example chunks with a representative document corpus and stable chunk IDs.
2. Create a judged query set with correct relevant chunk IDs.
3. Compare BM25-only, vector-only, hybrid, and hybrid + reranking on the same queries.
4. Report MRR, Recall@K, nDCG@K, p50/p95 latency, and approximate cost.
5. Add deterministic ingestion and cleanup so the notebook can be rerun safely.

## License

No license has been added yet. Unless a license is added, standard copyright applies; reuse is not automatically granted.
