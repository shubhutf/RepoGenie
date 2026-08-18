# Doclify

> AI‑driven tool that scans a codebase and synthesizes a professional README.md for developers.

Doclify is a command‑line application written in Python that automates the creation of high‑quality README files. It is aimed at developers and teams who want to keep documentation in sync with source code without manual effort. By leveraging the Groq large‑language‑model API, Doclify extracts source files, generates concise summaries, and assembles them into a cohesive Markdown document. The tool follows a multi‑stage pipeline (initialisation, generation, and selective update) and stores intermediate results in a hidden cache to enable fast incremental runs.

The project is in active development and provides a functional CLI for the core workflow. Configuration is driven by a `doclify.yaml` file and environment variables; no additional services are required beyond a Groq API key.

## Quick Start

```bash
# Clone the repository
git clone https://github.com/your-org/Doclify.git
cd Doclify

# Create a virtual environment and install dependencies
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Provide the Groq API key
echo "GROQ_API_KEY=your-groq-key" > .env

# Initialise the project (creates .doclify folder and config)
doclify init

# Run the full documentation generation pipeline
doclify run
```

*Optional Docker alternative:*  
```bash
docker run --rm -v "$(pwd)":/app -w /app -e GROQ_API_KEY=your-groq-key doclify:latest doclify run
```

## Environment Variables

```dotenv
# .env
GROQ_API_KEY=            # Groq API key required for LLM inference (never commit this file)
```

The `.env` file is read automatically; ensure it is excluded from version control.

## Core Features

- **Project initialisation** – Scans the repository, updates `.gitignore`, and creates a baseline `doclify.yaml` configuration.  
- **Multi‑stage generation pipeline** – Extracts source files, summarizes them with an LLM, caches results, and produces a final `README.md`.  
- **Selective update** – Refreshes documentation for a single file or the entire project without re‑processing unchanged files.  
- **Model discovery** – Lists available Groq language models and allows explicit selection.  
- **Cache management** – Persists intermediate summaries in a hidden `.doclify` directory to accelerate subsequent runs.

## Libraries & Tools

**Runtime libraries**

- `click` – CLI framework.  
- `rich` – Rich console output and tables.  
- `pydantic` – Data validation and schema enforcement.  
- `PyYAML` – YAML configuration parsing.  
- `pathspec` – Gitignore pattern matching.  
- `httpx` – HTTP client for Groq API calls.

**Development tools**

- `pytest` – Test runner (if tests are added).  
- `black` – Code formatting.  

## Tech Stack

| Layer                | Technology | Purpose                              |
|----------------------|------------|--------------------------------------|
| Language             | Python 3.9+| Core implementation                  |
| CLI framework        | Click      | Command parsing and subcommands      |
| UI/Logging           | Rich       | Colored console UI and structured logs|
| Schema validation    | Pydantic   | Typed configuration and payload models|
| Config format        | YAML (PyYAML) | Human‑readable project settings    |
| LLM integration      | Groq API (httpx) | Generate documentation text   |
| File pattern matching| pathspec   | Apply `.gitignore` rules             |
| Caching              | JSON files | Store intermediate summaries         |

## System Architecture / Workflow

- **CLI entry point** loads environment variables and reads `doclify.yaml`.  
- **Init command** scans the repository, updates `.gitignore`, and writes the initial configuration.  
- **Run command** extracts source files, chunks large files, and sends each chunk to the Groq LLM using a predefined prompt.  
- **Cache layer** stores per‑file summaries in `.doclify/cache.json` to avoid redundant LLM calls.  
- **Readme synthesis** aggregates cached summaries, feeds them to a final prompt, and writes the generated `README.md`.  
- **Update command** refreshes the cache for specified paths and optionally regenerates the README.  

All stages emit structured logs and Rich console feedback for transparency.

## Usage

```bash
# Initialise a new Doclify project
doclify init

# List available Groq models
doclify models list

# Generate documentation for the entire repository
doclify run

# Update documentation for a single file
doclify update path/to/file.py

# Update documentation for all files
doclify update .
```

Each command accepts optional flags to override the model or provider; see `doclify --help` for details.

## Project Structure

```
Doclify/
├─ doclify/
│  ├─ components/      # Core pipeline implementations (init, run, update, models)
│  ├─ pipelines/       # CLI supervisor definition
│  ├─ prompts/         # LLM prompt templates
│  ├─ schema/          # Pydantic data models
│  └─ utils/           # Helpers (extractor, cache, logger, LLM client, scanner)
├─ .gitignore
├─ doclify.yaml        # Project configuration
├─ .env                # Environment variables (GROQ_API_KEY)
├─ README.md
├─ main.py             # Entry script that forwards to the CLI
└─ requirements.txt
```

- `doclify/components/` contains the high‑level operations invoked by the CLI.  
- `doclify/utils/` provides reusable helpers for file extraction, caching, logging, and LLM interaction.  
- `doclify/prompts/` holds the text templates that drive the summarisation and final README generation.

## Contributing

1. Fork the repository.  
2. Create a feature branch (`git checkout -b feat/your-feature`).  
3. Commit changes following the **Conventional Commits** format.  
4. Push the branch and open a Pull Request.  

Please open an issue before undertaking large changes. Follow the project's `CODE_OF_CONDUCT.md` for community standards.

## Support

- **Bugs & feature requests:** Open an issue on GitHub.  
- **Usage questions:** Use GitHub Discussions.  
- **Security concerns:** Email `security@your-org.com` (see `SECURITY.md`).