# AI Memory Lakehouse Graph Mining Working Context

This repo is the data engineering / graph mining side of the graduation-project
direction. It is not the runnable Niko Agent harness. Its purpose is to design a
data backend that can later improve an agent's memory.

When talking to the project owner, use Vietnamese, refer to yourself as "em",
and refer to the user as "anh".

## Project Goal

Build a proof-of-concept data platform for AI memory:

```text
Public Jira data
  -> Bronze raw storage
  -> Silver cleaned/validated Parquet
  -> Gold graph-ready data
  -> Neo4j Knowledge Graph
  -> graph mining / analytics / GraphRAG demo
```

The business framing is issue/ticket tracking for IT Helpdesk, developer
support, or an internal AI support agent.

## Relationship With Niko_Agent

`Niko_Agent` is the runnable harness baseline:

- Telegram gateway.
- Fast/Deep agent routing.
- SQLite baseline memory.
- JSONL traces.
- Mini Ops dashboard.

This repo explains the future memory backend:

- normalize raw operational data;
- separate Semantic Memory and Episodic Memory;
- model relations as a Knowledge Graph;
- run graph mining;
- provide better retrieval context for the agent.

In the combined story:

```text
Niko_Agent proves the harness can run.
This repo proves how memory data should be stored, linked, mined, and retrieved.
```

## Current Dataset Direction

Primary dataset:

```text
Public Jira Dataset v7
```

Expected role:

- issue title/body/comment/resolution -> Semantic Memory;
- changelog/status/assignment/comment timeline -> Episodic Memory;
- project/component/actor/label/issue links -> Knowledge Graph relationships.

Optional later sources:

- Stack Exchange subset for additional semantic knowledge.
- GH Archive for future event-stream ingestion.

## Important Docs

- `README.md`: high-level overview and document index.
- `docs/Problem.md`: problem statement and project scope.
- `docs/Business-Domain.md`: Jira/support business domain for the sample data.
- `docs/Harness-Agent-Business/README.md`: memory backend as a business service
  for an agent harness.
- `docs/Harness-Agent-Business/Memory-Domain-And-Graph.md`: memory as its own
  domain and possible agent memory graph.
- `docs/Harness-Agent-Business/LLM-Value-Trust.md`: why LLM analysis has
  business value and how to discuss trust/security.
- `docs/Data-Sources.md`: dataset choices.
- `docs/Architecture.md`: mini-lakehouse / medallion architecture.
- `docs/Graph-Schema.md`: draft graph schema for Public Jira.
- `docs/Technology-Stack.md`: planned technologies.
- `docs/Roadmap.md`: implementation phases.
- `docs/Evaluation.md`: evaluation plan.

## Scope Boundaries

In scope:

- lakehouse-style Bronze/Silver/Gold design;
- MinIO + Parquet baseline storage;
- Public Jira ingestion and EDA;
- schema validation;
- Neo4j Knowledge Graph;
- graph mining on issue/ticket relations;
- dashboard or GraphRAG proof of concept.

Out of scope for the first version:

- a full production AI agent;
- a production-grade Jira connector;
- Delta/Iceberg as a hard requirement;
- guaranteed token reduction or hallucination reduction without experiment;
- a fully automatic procedural memory system.

## Business Framing To Preserve

Use this wording when explaining the project:

```text
Traditional analytics handles structured metrics well, but issue tracking data
contains text, comments, status history, and hidden relationships. The project
organizes this data into Semantic/Episodic Memory and a Knowledge Graph so an
agent or LLM can retrieve grounded context instead of reading scattered data.
```

Do not frame LLM as the source of truth. The source of truth is the data platform
and graph. LLM is the language/analysis layer on top of retrieved context.

## Editing Rules For Future Sessions

- Keep docs aligned with the current chosen dataset: Public Jira Dataset v7.
- Do not overcommit to a final graph schema before EDA confirms real fields.
- Use "mini-lakehouse" or "lakehouse-style" unless Delta/Iceberg is implemented.
- Distinguish the Jira/support business graph from a future agent memory graph.
- Keep Niko_Agent-specific runtime details in the Niko repo; only describe the
  integration boundary here.
- Do not add downloaded datasets, secrets, tokens, or local runtime files to git.

