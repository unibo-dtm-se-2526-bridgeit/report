---
title: Concept
has_children: false
nav_order: 2
---

# Concept

BridgeIT is a web-based backend service with a companion browser frontend. It is a Requirements Engineering platform featuring AI-assisted analysis and explicit human validation of natural-language requirements.

The backend is distributed as source code, cloned from its repository and run locally through Poetry:

```bash
git clone https://github.com/unibo-dtm-se-2526-bridgeit/BridgeIT-artifact.git
cd BridgeIT-artifact
poetry install
poetry run uvicorn bridgeit.adapters.api.main:app --reload
```

As an alternative, the same backend can be run inside an isolated Docker container, with a single command and no local Python or Poetry installation required.

Once running, the user opens the accompanying frontend locally in a browser, without any build step or installation of its own. The backend is built with FastAPI on Python, making it cross-platform across Windows, macOS, and Linux, and connects to Google's Gemini API for AI-assisted analysis, requiring a network connection for that specific capability.

Therefore, the users are stakeholders that can submit a requirement, request an AI-assisted analysis of it, revise its wording in response to issues the analysis surfaces, and record an explicit validation decision (approve, edit, or reject). A requirement's status changes only as a direct result of these actions: it moves from Submitted to Analyzed once an AI analysis has been produced, and to Validated (or Rejected) only once a human reviewer has recorded a decision — never automatically.

The requirement's full history is persisted in a SQLite database on the server's filesystem, surviving restarts of the application. During analysis, the AI Gateway calls the Gemini API and returns a quality indication together with a list of concrete issues found in the requirement's wording, each one explained in plain language.

The software is intended for professional use during a project's requirements-gathering and refinement phases. Users typically interact with it in the context of a specific project — capturing a stakeholder's intent, refining it with AI assistance, and validating it before it becomes authoritative — rather than for casual or recreational use. The platform also appeals to teams who want to reduce ambiguity in early-stage specifications without giving up human oversight of what an AI suggests. The frequency of use follows the pace of requirements elicitation itself: it is expected to be recurring throughout a project's early phases, whenever new requirements are captured or existing ones are revisited, rather than a one-off interaction.
