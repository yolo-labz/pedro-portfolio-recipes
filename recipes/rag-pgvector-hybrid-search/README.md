# rag-pgvector-hybrid-search

Hybrid retrieval in Postgres 16: pgvector cosine similarity AND `tsvector` BM25
lexical, fused with Reciprocal Rank Fusion. One SQL statement, no separate
search server.

## Problem

Pure vector search misses exact-token matches (proper nouns, error codes, dates).
Pure BM25 misses paraphrase. Running Elasticsearch alongside Postgres doubles
the operational surface for a retrieval problem Postgres can answer.

## Snippet

```sql
-- Schema
CREATE EXTENSION IF NOT EXISTS vector;
CREATE TABLE docs (
  id        bigserial PRIMARY KEY,
  body      text NOT NULL,
  embedding vector(1024) NOT NULL,
  tsv       tsvector GENERATED ALWAYS AS (to_tsvector('portuguese', body)) STORED
);
CREATE INDEX docs_embed_idx ON docs USING hnsw (embedding vector_cosine_ops);
CREATE INDEX docs_tsv_idx   ON docs USING gin  (tsv);

-- Hybrid retrieval via RRF (k=60 is the classic constant).
WITH
vec AS (
  SELECT id, row_number() OVER (ORDER BY embedding <=> $1) AS rnk
  FROM docs ORDER BY embedding <=> $1 LIMIT 50
),
lex AS (
  SELECT id, row_number() OVER (ORDER BY ts_rank_cd(tsv, query) DESC) AS rnk
  FROM docs, plainto_tsquery('portuguese', $2) query
  WHERE tsv @@ query ORDER BY ts_rank_cd(tsv, query) DESC LIMIT 50
)
SELECT d.id, d.body,
       COALESCE(1.0/(60+v.rnk),0) + COALESCE(1.0/(60+l.rnk),0) AS score
FROM docs d
LEFT JOIN vec v USING (id)
LEFT JOIN lex l USING (id)
WHERE v.rnk IS NOT NULL OR l.rnk IS NOT NULL
ORDER BY score DESC LIMIT 10;
```

`$1` is the query embedding, `$2` is the raw text. Same query does both retrievals.

## Why

RRF is rank-based, so the two signals don't need calibrated scores — you avoid
the trap of "vector distance 0.34 vs BM25 score 8.2, which is better?". HNSW
over pgvector 0.8+ is competitive with dedicated vector DBs at <10M row scale,
and you get Postgres's transactional guarantees, backup story, and access
control for free.

## When NOT to use

Don't use when corpus >50M rows or query QPS >1k — at that scale dedicated
vector stores (Qdrant, Vespa) or search engines (OpenSearch) amortize the
specialization. Also skip RRF when you have calibrated scores from a reranker;
a cross-encoder rerank on the top 50 beats RRF on quality.

## Reference

- https://github.com/pgvector/pgvector
