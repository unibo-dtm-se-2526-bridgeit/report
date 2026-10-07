---
title: Usage of Generative AI
has_children: false
nav_order: 14
---

# Usage of Generative AI

This section documents how generative AI was used while building BridgeIT, and the criteria we followed when working with it.

Two different uses of AI must not be confused:

- **Gemini is a feature of the product.** BridgeIT calls the Gemini API, through the `AIGateway` port, to assist the analysis of requirements. This is documented in the [Design](../03-design/) and [Development](../04-development/) chapters and is not covered here.
- **Claude and ChatGPT are tools we used to build the product.** This section is about them.

## Our stance: collaborators, not substitutes

We used **Claude** (Anthropic) and **ChatGPT** (OpenAI) as collaborators: tools to think with, to ask questions to, and to get a second opinion from. We did not use them as a replacement for our own work. This mirrors the principle at the heart of BridgeIT itself, where the AI only produces proposals and a human takes the decision.

In practice:

- The architectural principles (Domain-Driven Design and Hexagonal Architecture) were dictated by the course requirements. Within them, the AI provided technical analysis and recommendations, while **every design decision was made by us**.
- Every change suggested by the AI was **reviewed, applied, or discarded by us**. Nothing was accepted just because it came from an AI.
- Every commit was **reviewed and made by the authors**, who take full responsibility for the content of the final report and artifact.


Generative AI was also used to support project planning, to review intermediate work, and to identify possible inconsistencies between the implementation and the documentation. These uses followed the same human-review principle described above.

## Criteria we followed

- **The human decides.** The AI proposes, we choose. This applies to design choices, code changes, and the text of the report alike.
- **Understand before applying.** We did not apply a suggestion that we could not explain ourselves.
- **No special treatment for AI-assisted work.** Anything produced with AI help goes through exactly the same automated checks as the rest of the project: Mypy type checking, Ruff linting and format checking, and the pytest suite, all run by the [CI/CD pipeline](../08-cicd/) on applicable code-changing pushes and pull requests.
- **Clear and explicit requests.** We described the task, the context, and the constraints in our requests, instead of relying on prompt tricks.
- **Responsibility stays with the authors.** The AI is not an author: the final content of the report and of the artifact is our own.
