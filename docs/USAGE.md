# Usage reference

This reference complements the [project overview](../README.md) and [quickstart](../QUICKSTART.md). Run commands from the repository root with the environment activated and `.env` configured.

## CLI

```bash
# Add documents
python main.py --ingest-file ./path/to/report.pdf
python main.py --ingest-dir ./path/to/documents

# Inspect the index
python main.py --stats

# Generate a report without saving it
python main.py --query "What are the key company metrics?"

# Save Markdown or text
python main.py --query "What products does the company offer?" --output --format markdown
python main.py --query "Summarize customer feedback" --output --format text
```

To clear the indexed collection, `python main.py --clear` asks for confirmation. Clearing the collection is a data-management operation, not part of the normal report-generation workflow.

## Configuration

Start from the checked-in `.env.example`. Keep actual credentials in a local `.env` file.

| Setting | Purpose |
|---|---|
| `OPENAI_API_KEY` | Provider credential |
| `OPENAI_MODEL` | Report/summary generation model |
| `OPENAI_EMBEDDING_MODEL` | Document/query embedding model |
| `CHROMA_PERSIST_DIRECTORY` | Local vector-index storage |
| `CHROMA_COLLECTION_NAME` | Collection name |
| `CHUNK_SIZE`, `CHUNK_OVERLAP` | Document splitting |
| `TOP_K_RESULTS` | Retrieved context size |
| `REPORT_OUTPUT_DIR` | Saved report directory |
| `ENABLE_COST_TRACKING`, `COST_LOG_FILE` | Cost-estimation logging |
| `LOG_LEVEL`, `LOG_DIR`, `LOG_FILE` | Application logging |

Supported document loading is implemented in `src/utils/document_loader.py`; use that module as the reference when adding a new input format. Model access and pricing are provider-dependent, so this documentation does not hard-code a current pricing table.

## Python interface

```python
from src.agent.graph import ReportGenerationGraph
from src.observability.logger import setup_logging
from src.retrieval.vector_store import VectorStore

setup_logging()
store = VectorStore()
graph = ReportGenerationGraph(store)
result = graph.generate_report("What are the key company metrics?")

print(result.get("summary"))
print(result.get("report"))
print(result.get("sources"))
```

Ingest material before querying. Check the returned error state as well as the generated text when integrating the graph into another application.

## HTTP interface

```bash
uvicorn src.api.main:app --host 127.0.0.1 --port 8000
```

- Interactive endpoint documentation: `http://127.0.0.1:8000/docs`.
- Health endpoint: `http://127.0.0.1:8000/health`.

The API exposes ingestion, collection statistics, report generation and cost-summary operations. Consult its request schemas rather than assuming that CLI flags map directly to HTTP fields.

## Tests

The repository contains tests for ingestion, retrieval, graph orchestration, API/CLI handling, exports and cost tracking. To run the suite in your configured development environment:

```bash
pytest
```

For a focused file:

```bash
pytest tests/test_vector_store.py
```

Mock-based tests check component behavior without establishing the quality of a live generated report. Treat retrieval relevance, source support and provider behavior as separate evaluation questions.
