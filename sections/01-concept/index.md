---
title: Concept
has_children: false
nav_order: 2
---

# Concept

Here you should explain:
- The type of product developed with that project, for example (non-exhaustive):
    - Application (with GUI, be it mobile, web, or desktop)
    - Command-line application (CLI could be used by humans or scripts)
    - Library
    - Web-service(s)
    - Data processing toolkit (= Library + CLI, or Jupyter Notebook)

- Use case collection
    - Where are the users?
    - When and how frequently do they interact with the system?
    - How do they interact with the system? Which devices are they using?
    - Does the system need to store user's data? Which data? Where?
    - Most likely, there will be multiple roles.

---
---
title: Concept
has_children: false
nav_order: 2
---

# Concept

## The Nature of the Platform

BridgeIT is conceived as a web service: a backend platform that exposes its functionality through a REST API (see [Architecture — API Design](./architecture.md#api-design)), rather than as a command-line tool or a library meant to be imported into other projects. Should a graphical interface be introduced later, it would take the form of a thin web client consuming this same API, not a separate product in its own right. Although the platform's core purpose involves processing natural-language text, its unit of reuse and its primary point of contact with the outside world is the service itself, reachable over HTTP.

## Users and Their Context

The people BridgeIT is built for are those involved in a software project's Requirements Engineering process: business stakeholders who originate requirements, requirements engineers and business analysts who refine and validate them, and software engineers who consume them downstream (see [Report — Stakeholders](./report.md#stakeholders)). Nothing about the platform assumes these people share a physical location — it is designed to be reached remotely over HTTP, in line with its nature as a web service, and its users are expected to work from a desktop or laptop computer rather than a mobile device, consistent with the professional, document-heavy nature of Requirements Engineering work.

## Interaction Patterns

Use of the platform follows the natural rhythm of requirements work rather than a fixed schedule: a single requirement moves through submission, AI-assisted analysis, optional clarification, and validation over a span of hours or days, not within one continuous session. BridgeIT is not designed around high-frequency or real-time interaction — there is no expectation of sub-second polling or live collaborative editing at this stage. In practice, users reach the system through direct calls to its REST API, most immediately via the interactive documentation FastAPI generates automatically, and later, potentially, through the thin web client mentioned above.

## Data and Persistence

BridgeIT stores two categories of data. The first is the Requirements Engineering domain data described in the [Domain Model](./domain-model.md): requirements, together with their lifecycle status, the AI Analyses performed on them, the Traceability Links connecting them to derived artifacts, and the Derived Artifacts themselves. The second is user management data: since the platform must now support distinct user accounts mapped to the roles described below, information about users and their assigned roles is likewise treated as persistent system data.

Both categories are stored in a **SQLite database**, accessed exclusively through the **Repository pattern**. This keeps the domain layer decoupled from the specific storage technology: the domain expresses what needs to be persisted in its own terms, while the repository implementation is the only part of the system aware that SQLite is being used underneath. Should the storage technology need to change later, this decision is confined to the repository adapters and does not ripple into the domain model.

## Roles within the System

BridgeIT distinguishes three functional roles, each interacting with the same underlying data from a different perspective: the **Business Stakeholder / Domain Expert**, who originates requirements; the **Requirements Engineer / Business Analyst**, who refines, clarifies, and validates them; and the **Software Engineer / Developer**, who consumes validated requirements and their traceability information downstream. Following the professor's guidance, these are no longer purely conceptual distinctions: the platform manages user accounts directly, and each user is associated with one of these roles as part of the system's persisted data.