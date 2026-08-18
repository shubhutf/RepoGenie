# RepoGenie

> AI assistant for repository understanding and docs.

RepoGenie is a command-line tool that scans a codebase and generates a professional `README.md` for it automatically. It's built for developers who want documentation that stays in sync with their code without spending hours writing it by hand. Under the hood, RepoGenie extracts source files, summarizes each one using an LLM, and assembles those summaries into a cohesive, well-structured document — all through a multi-stage pipeline (initialize → generate → selectively update) with results cached locally for fast incremental runs.

RepoGenie started as a hands-on exploration of how to build reliable AI agents for real codebases — how to feed a language model the right context, at the right size, without drowning it in noise. It's actively evolving as I extend it.

## Quick Start

```bash
# Clone the repository
git clone https://github.com/shubhutf/RepoGenie.git
cd RepoGenie

# Install dependencies with uv
pip install uv
uv sync

# Provide your API key
echo "GROQ_API_KEY=your-key-here" > .env

# Initialize the project (scans repo, creates config + cache folder)
uv run repogenie init

# Run the full documentation generation pipeline
uv run repogenie run
```

*Optional Docker alternative:*
```bash
docker run --rm -v "$(pwd)":/app -w /app -e GROQ_API_KEY=your-key-here repogenie:latest repogenie run
```

## Environment Variables

```dotenv
# .env
GROQ_API_KEY=            # API key required for LLM inference (never commit this file)
```

The `.env` file is read automatically — make sure it's listed in `.gitignore` and never pushed to version control.

## Core Features

- **Project initialization** — scans the repository, updates `.gitignore`, and creates a baseline `repogenie.yaml` configuration.
- **Multi-stage generation pipeline** — extracts source files, summarizes each one with an LLM, caches the results, and synthesizes a final `README.md`.
- **Selective update** — refreshes documentation for a single file or the whole project without re-processing files that haven't changed.
- **Model discovery** — lists available language models so you can pick the one that fits your project size and budget.
- **Local caching** — persists intermediate file summaries in a hidden `.repogenie/` directory to speed up every run after the first.

## Libraries & Tools

**Runtime libraries**

- `click` — CLI framework
- `rich` — colored console output and tables
- `pydantic` — data validation and schema enforcement
- `PyYAML` — YAML configuration parsing
- `pathspec` — `.gitignore` pattern matching
- `httpx` — HTTP client for LLM API calls

**Development tools**

- `pytest` — test runner
- `black` — code formatting

## Tech Stack

| Layer                  | Technology         | Purpose                                 |
|-------------------------|---------------------|------------------------------------------|
| Language                | Python 3.9+         | Core implementation                       |
| CLI framework           | Click                | Command parsing and subcommands           |
| UI / Logging            | Rich                 | Colored console UI and structured logs    |
| Schema validation       | Pydantic             | Typed configuration and payload models    |
| Config format           | YAML (PyYAML)        | Human-readable project settings           |
| LLM integration         | Groq API (httpx)     | Generates the documentation text          |
| File pattern matching   | pathspec             | Applies `.gitignore` rules                |
| Caching                 | JSON files           | Stores intermediate file summaries        |

## How It Works

1. **CLI entry point** loads environment variables and reads `repogenie.yaml`.
2. **`init`** scans the repository, updates `.gitignore`, and writes the initial configuration manifest.
3. **`run`** extracts source files, chunks larger ones, and sends each chunk to the LLM using a dedicated summarization prompt.
4. **Cache layer** stores per-file summaries in `.repogenie/cache.json` so repeat runs skip unchanged files entirely.
5. **README synthesis** gathers all cached summaries — never the raw source — and feeds them into a final prompt that writes the actual `README.md`.
6. **`update`** selectively refreshes the cache for one file or path, then optionally regenerates the README from the updated cache.

Every stage logs its progress with structured output so you can see exactly what's happening at each step.

## Usage

```bash
# Initialize a new RepoGenie project
repogenie init

# List available models
repogenie models

# Set a default model
repogenie set default <model_id>

# Generate documentation for the entire repository
repogenie run

# Update documentation for a single file
repogenie update path/to/file.py

# Regenerate README from the existing cache without re-summarizing everything
repogenie update .
```

Run `repogenie --help` for the full list of flags, including per-run model overrides.

## Project Structure

```
RepoGenie/
├─ repogenie/
│  ├─ components/      # Core pipeline logic (init, run, update, models)
│  ├─ pipelines/       # CLI supervisor / command routing
│  ├─ prompts/         # LLM prompt templates
│  ├─ schema/          # Pydantic data models
│  └─ utils/           # Helpers: file extraction, caching, logging, LLM client, scanner
├─ .gitignore
├─ repogenie.yaml       # Project configuration (generated by `init`)
├─ .env                 # Environment variables — GROQ_API_KEY (never committed)
├─ README.md
├─ main.py              # Entry script that forwards to the CLI
└─ pyproject.toml
```

- `repogenie/components/` — the high-level operations the CLI invokes (init, run, update, models).
- `repogenie/utils/` — reusable helpers for file extraction, caching, logging, and the LLM client.
- `repogenie/prompts/` — the text templates that drive summarization and final README generation. These are treated as first-class project assets, not boilerplate — the quality of the output depends almost entirely on the quality of these prompts.

## Roadmap / Planned Changes

This project is under active development. Some directions being explored:

- [ ] Support for additional LLM providers beyond Groq
- [ ] Support for languages beyond Python
- [ ] A lighter-weight web UI as an alternative to the CLI
- [ ] Tests


## Contributing

1. Fork the repository.
2. Create a feature branch (`git checkout -b feat/your-feature`).
3. Commit your changes.
4. Push the branch and open a Pull Request.

Please open an issue before starting large changes so we can align on direction first.

## Support

- **Bugs & feature requests:** open an issue on GitHub.
- **Usage questions:** use GitHub Discussions.