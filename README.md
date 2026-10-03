# JobSearch Agent

Upload your CV, pick a few countries, and get a ranked, downloadable report of the jobs that actually fit you — from a web console or a REST API, all in one Docker container.

<p>
  <a href="https://www.python.org/downloads/"><img src="https://img.shields.io/badge/python-3.11%2B-blue.svg" alt="Python 3.11+"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT"></a>
  <a href="https://github.com/MatMaxMatrix/JobSearch-Agent/stargazers"><img src="https://img.shields.io/github/stars/MatMaxMatrix/JobSearch-Agent?style=social" alt="Stars"></a>
  <img src="https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white" alt="FastAPI">
  <img src="https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white" alt="Docker">
</p>

It scrapes recent LinkedIn postings, parses your PDF CV with an LLM, and scores each job against your background. It can also draft tailored CVs and cover letters. Drive it from the bundled web console, the HTTP/WebSocket API, or the CLI.

## What you get

- **CV → jobs matching** — upload a PDF, choose target countries, and download a ranked Markdown/PDF report (`POST /match`).
- **Web console** — the whole flow in the browser: upload, live progress, results, and a settings panel for your model and API keys. No build step (Tailwind + Alpine via CDN), served straight from FastAPI.
- **AI documents** — tailored CV and cover-letter generation through a Google ADK agent pipeline.
- **LinkedIn scraper** — Playwright-based, with session reuse and proxy support.
- **One container** — `docker compose up` and open the console.

## Quick start

### Docker (recommended)

```bash
git clone https://github.com/MatMaxMatrix/JobSearch-Agent.git
cd JobSearch-Agent
cp .env.example .env        # fill in your keys — see Configuration
docker compose up --build
```

Open **http://localhost:8080** for the console (or `/docs` for the API). The port is bound to `127.0.0.1`, so it stays on your machine.

### Local Python

```bash
pip install -e .
playwright install --with-deps chromium
jobsearch-api               # serves the console + API on http://localhost:8000
```

The install adds two commands: `jobsearch-api` (the server) and `jobsearch-agent` (the CLI).

## Configuration

Copy `.env.example` to `.env` and fill in what you need:

| Variable | Needed for |
|---|---|
| `API_KEY` | All write endpoints + the console (sent as `X-API-Key`). |
| `ANTHROPIC_API_KEY` | AI CV / cover-letter generation + agent search on Claude. |
| `DEEPSEEK_API_KEY` | CV → jobs matching. |
| `GOOGLE_API_KEY` | AI CV / cover-letter generation + agent search on Gemini. |
| `GITHUB_API_KEY` | The `gpt_*` model aliases (GitHub Models via litellm). |
| `LINKEDIN_USERNAME` / `LINKEDIN_PASSWORD` | Scraping (use a throwaway account). |
| `TAVILY_API_KEY` | Web-search agent when running on Claude (optional for Gemini). |
| `LITELLM_MASTER_KEY` | Optional: route all model traffic through the `litellm` proxy service (must start with `sk-`). |

Set `ANTHROPIC_API_KEY` (Claude via LiteLLM) or `GOOGLE_API_KEY` (Gemini) — the
agents run on whichever is present; with both set, the `*_model` selectors in
`config/*.yaml` decide per agent. You can also set the keys at runtime from the
console's settings panel.

With `LITELLM_MASTER_KEY` set (and the compose stack running), every backend —
including Gemini — goes through the `litellm` proxy service instead of being
called directly; provider keys then only need to exist on the proxy.

## Docker image

Every push to `main` builds and publishes the image to GHCR
(`.github/workflows/docker-publish.yml`; the repo is public, so pulls are
anonymous):

```bash
# Pull the published image instead of building locally:
docker-compose pull jobsearch
docker-compose up -d

# Or force a local build from your checkout:
docker-compose up --build
```

## API at a glance

| Endpoint | Purpose |
|---|---|
| `GET /` | Web console |
| `POST /match` → `GET /match/{id}` | Match a CV against recent jobs per country |
| `POST /search` → `GET /search/{id}` | Keyword job search |
| `GET /jobs/stats`, `GET /search/history` | Database stats and recent runs |
| `GET /config` / `POST /config` | Read / set model and provider keys |
| `WS /ws` | Live progress |

Full schema at `/docs`. CLI help: `jobsearch-agent --help`.

## Project layout

```
jobsearch_agent/
├── api/            FastAPI app, Uvicorn launcher, and the bundled web console (api/ui/)
├── agents/         CV, cover-letter, parser, and search agents
├── scraper/        LinkedIn and BugMeNot scrapers
├── utils/          Pipelines, databases, file helpers
└── prompts/        Agent prompts
config/             YAML configuration
docs/               Detailed guides
tests/              Test scripts
```

## Documentation

Deeper guides live in [`docs/`](docs/): the [LinkedIn scraper](docs/LINKEDIN_SCRAPER.md), [API reference](docs/API.md), [configuration](docs/ADVANCED_CONFIGURATION.md), [deployment](docs/DEPLOYMENT.md), and [development](docs/DEVELOPMENT.md).

## Notes

LinkedIn scraping sits in a grey area of their Terms of Service and is rate-limited and prone to occasional security challenges — use a throwaway account and keep volumes low. This project is for personal and educational use; you're responsible for how you use it.

## License

MIT — see [LICENSE](LICENSE).
