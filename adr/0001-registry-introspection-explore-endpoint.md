# ADR-0001: Registry Introspection and Telemetry via `/explore`

## Status
Superseded by [ADR-0005](0005-registry-shared-query-model.md)

## Context
In v0.4 of the Agent Finder specification, we identified the need for "Registry Introspection"—allowing LLM orchestrators and clients to query a registry's capabilities (such as supported media types, protocols, and federation styles) before executing deep searches. Concurrently, our reference implementations proved the immense utility of "Registry Telemetry" (exposing aggregate statistics like total agent counts, protocol breakdowns, and category distributions) to support developer portals and directory dashboards.

We considered two approaches:
1. **Option A (Static GET Endpoint)**: A simple, high-performance `GET /stats` or `GET /info` endpoint returning global counts and supported media types.
2. **Option B (Dynamic POST Endpoint)**: A query-aware `POST /explore` endpoint that calculates dynamic counts/facets computed over specific, filtered queries matching the `POST /search` request structure.

## Decision
We adopted **Option B (the dynamic `POST /explore` endpoint)**.

This approach is mathematically a superset of Option A:
* If a client requires cheap global stats, they simply send an **empty query** (no `text` or `filter` arguments) to `POST /explore` asking for facets. The database counts over the entire dataset, returning the exact same telemetry structure.
* If a client requires dynamic faceted navigation (e.g., counting protocols matching a specific search subset), they pass their search query to `/explore` to retrieve real-time aggregate facet counts.

This keeps the core `/agents` browse endpoint strictly focused on returning individual records, while dedicating a unified, flexible endpoint for introspection and telemetry.

## Consequences
* **Dynamic Capability**: Clients and LLM planners can now perform faceted search queries and query-aware capability discovery.
* **Performance Optimization**: Registries that have resource constraints can pre-compute and cache the "empty query" response to `/explore`, giving it the exact same performance profile as a static `GET` endpoint.
* **Optionality**: The `/explore` endpoint is marked as **Optional** in the specification, protecting lightweight registry implementations (like solo local SQLite setups) from mandatory complex database aggregation.
