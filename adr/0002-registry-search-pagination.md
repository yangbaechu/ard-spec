# ADR-0002: Registry Search API Pagination

## Status
Accepted

## Context
In v0.4.2 of the Agent Finder specification, the `POST /search` API allows clients to execute semantic natural language queries against the registry. As the federated registry index scales to thousands or millions of entries, queries can match a very large subset of capabilities.

We identified that:
1. Returning an unbounded list of search matches on a single request is a severe performance risk, causing network latency, high serialization overhead, and client-side memory crashes.
2. Search engines waste substantial CPU cycles calculating dynamic relevance scores (e.g., TF-IDF, BM25, or cosine similarity distances) and sorting records that the client may never parse or display.

## Decision
We explicitly mandated and documented strict root-level pagination parameters for the `POST /search` request payload:
* **`pageSize` (Integer)**: Optional. Specifies the maximum number of ranked results to return per page (default: 10, max: 100).
* **`pageToken` (String)**: Optional. A base64 encoded string mapping to the underlying database offset/cursor to retrieve the next page.

The response schema matches this structure, returning standard catalog entries wrapped in a `results` array alongside a root-level `pageToken` when additional pages are available.

This aligns the `POST /search` pagination logic directly with the browse-level `GET /agents` pagination logic.

## Consequences
* **Bound Result Delivery**: Prevents unbounded response sizes, ensuring predictable latency and low memory footprint on both client and server.
* **Optimized Database Scans**: Registries can apply `LIMIT` and `OFFSET` early in the search query pipeline, preventing waste on scoring and ranking low-relevance records.
* **Ecosystem Consistency**: Standardizes pagination logic across both semantic search and deterministic listing interfaces.
