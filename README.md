# LLM-wiki-testing

Testing the capabilities of LLM wiki infrastructure — a persistent, compounding knowledge base maintained by an LLM over scientific papers, based on Karpathy's pattern.

## Structure

```
raw/          # source PDFs (immutable)
wiki/
  sources/    # one page per ingested paper
  concepts/   # cross-paper concept pages
  entities/   # named things (datasets, tools, reactors, standards)
  index.md    # master table of all pages
  log.md      # append-only ingest history
WIKI_SCHEMA.md  # conventions and workflow spec
```

## How it works

Each paper is ingested by an LLM that reads the PDF, creates or updates markdown pages, and maintains cross-references across the wiki. There are three page types:

- **Sources** — one per paper; what it did, how, and what it found
- **Concepts** — ideas that appear across multiple papers (e.g., GraphRAG, chain-of-verification, RL for scientific discovery); updated incrementally as new papers add to them
- **Entities** — specific named things referenced by papers (datasets, tools, reactors, standards)

The log records every ingest operation. The index is the entry point for querying.

## Papers ingested

| Paper | Topic |
|---|---|
| Bunkova et al. 2026 | Text2Cypher pipeline over a reaction KG for synthesis retrieval |
| Alimin & Schweidtmann 2026 | GraphRAG over P&IDs via 4-tool agentic system (ChatP&ID) |
| Heyer et al. 2025 | RL (Q-learning) for automated mechanistic reactor model generation |
| Laub et al. 2026 | Hierarchical RL for reactor compartmentalization + ontology layer |
| Goldstein et al. 2025 | pyDEXPI: open-source Python framework for the DEXPI P&ID standard |
