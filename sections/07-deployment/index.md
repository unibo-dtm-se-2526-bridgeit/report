---
title: Deployment
has_children: false
nav_order: 8
---

# Deployment

This section explains what operations are needed to make the software work on the users' machine(s).

## User installation

You need Python 3.10 or later to install and run BridgeIT from source. Alternatively, Docker lets you run it without installing Python at all.

BridgeIT is not published to PyPI (see [Release](../06-release/) for why), so there is no `pip install` path; instead, the two ways to get started are **from source** and **with Docker**.

### With Docker

If you have Docker installed, this is the fastest path, requiring no local Python or Poetry setup:

```bash
git clone https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact.git
cd BridgeIT-artifact
docker compose up --build
```

Once the container is running, the API is available at `http://127.0.0.1:8000`, with interactive documentation at `http://127.0.0.1:8000/docs`.

### From source

1. Clone the repository:

```bash
git clone https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact.git
cd BridgeIT-artifact
```

2. Install Poetry if you don't have it yet:

```bash
pip install -r requirements.txt
```

3. Install the project's dependencies (creates `.venv/` inside the project):

```bash
poetry install
```

4. Run the backend with the command:

```bash
poetry run uvicorn bridgeit.adapters.api.main:app --reload
```

This brings up the API at `http://127.0.0.1:8000`. The accompanying frontend (`web/`) needs no installation of its own: once the backend is running, its HTML pages are opened directly in a browser.

### Configuration

Either way, one environment variable must be set before requesting an AI-assisted analysis: `GEMINI_API_KEY`, holding a valid Gemini API key (obtainable for free from Google AI Studio). Its absence does not prevent the application from starting or from submitting/viewing requirements, it is only required, and only checked, at the moment an analysis is actually requested. No other configuration file is needed: the SQLite database file (`bridgeit.db`) is created automatically on first use.

## Virtual environment

All project commands run inside the `.venv/` virtual environment created by Poetry. Activate it once per terminal session, then use tools directly without any prefix:

```bash
python3 -m venv .venv

# macOS / Linux
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Windows (cmd)
.venv\Scripts\activate.bat
```

To deactivate: `deactivate`.

## Server-side installation

BridgeIT does not currently have a dedicated, hosted server deployment: the backend described above is run locally by whoever wants to use it (a team member, or the course evaluator), not deployed to a remote server or cloud provider that other users would connect to over the internet. This was a deliberate scope decision for the current milestone, see [Future work](../12-future/) for a discussion of what a hosted deployment would require.

**Further software required on the server, if hosted:** none beyond what "User installation" already lists. SQLite runs embedded within the same process as the backend — it is not a separate service to install, configure, or keep running, unlike a client-server DBMS (e.g. PostgreSQL) would be. The only genuinely external dependency is the Gemini API itself, a third-party service reached over the network, not something installed alongside BridgeIT.
