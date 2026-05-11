# LLM Wiki Schema

## Architecture
- raw/ → immutable source PDFs
- wiki/ → LLM-generated markdown files
- WIKI_SCHEMA.md → this file, conventions and workflows

## Wiki conventions
- Every ingested paper gets a summary page
- index.md updated on every ingest
- log.md append-only record of all operations
- Cross-references maintained across pages

## Input types (for Gráfico integration)
- ChatMessage
- UserFile  
- AgentFile (.xyz)
- All tagged with session_id

## Workflows
- Ingest: read source → extract key info → update wiki pages → update index → append to log
- Query: read index → find relevant pages → synthesize answer
- Lint: check contradictions, orphan pages, missing cross-references