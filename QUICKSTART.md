# Quickstart

This guide takes you from a local checkout to a report generated from the supplied sample material. It does not require Docker; use the container alternative below if you prefer the existing container setup.

## Prerequisites

- Python 3.11 or later.
- An OpenAI API key with access to the embedding and generation models you select.
- Network access for provider requests and dependency installation.

## 1. Set up the project

```bash
git clone https://github.com/alirazawatoo/company-report-rag-pipeline.git
cd company-report-rag-pipeline
python -m venv .venv
```

Activate the environment:

```bash
# macOS / Linux
source .venv/bin/activate
```

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install the project dependencies:

```bash
pip install -r requirements.txt
```

Copy `.env.example` to `.env`, then set `OPENAI_API_KEY` locally. Review `OPENAI_MODEL` and `OPENAI_EMBEDDING_MODEL` for the models your account can access. Do not put a real API key in a committed example.

On macOS/Linux, the copy command is:

```bash
cp .env.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

## 2. Ingest sample material

```bash
python scripts/ingest_sample_data.py
python main.py --stats
```

The collection statistics show whether chunks are available for retrieval. The chunk count depends on the supplied material and chunk settings; there is no fixed success count to match.

## 3. Ask a question

```bash
python main.py --query "What are the company's revenue trends?" --output --format markdown
```

Review the executive summary, report and source list. Exported reports go to the directory configured by `REPORT_OUTPUT_DIR` (by default, `./reports`). Provider calls may incur charges; reported costs are estimates.

## 4. Use your own documents

```bash
python main.py --ingest-file ./path/to/company-report.pdf
python main.py --ingest-dir ./path/to/company-documents
```

Keep each question aligned with the material you ingested. Documents are processed locally for indexing, but embedding and generation calls use the configured external provider.

## Container alternative

The repository includes a `Dockerfile`, `docker-compose.yml` and `docker-run.sh` convenience wrapper. The wrapper expects Docker and the `docker-compose` command. From the repository root, after creating `.env`:

```bash
./docker-run.sh build
./docker-run.sh ingest /app/data/raw_data
./docker-run.sh query "What are the company's revenue trends?"
```

## If there is no usable result

| Symptom | Check |
|---|---|
| Empty vector store | Run `python main.py --stats`, then ingest documents |
| Provider error | Check the local key, account access, selected models and provider limits |
| Report lacks the requested information | Confirm that relevant source material was indexed; narrow the question |
| Unexpected storage location | Review `CHROMA_PERSIST_DIRECTORY` and `REPORT_OUTPUT_DIR` in `.env` |

For configuration, other commands, the Python interface and test commands, see [the usage reference](docs/USAGE.md).
