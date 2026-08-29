---
title: User guide
has_children: false
nav_order: 10
---

# User Guide

BridgeIT is a requirements engineering application designed to support the refinement of software requirements through AI-assisted quality analysis and explicit human validation.

The application follows a human-in-the-loop approach: Gemini can analyse a requirement and highlight potential quality issues, but it cannot approve, modify, or reject a requirement autonomously. The authoritative decision always remains with the Business Analyst.

This guide describes the web interface and the typical workflow available to an end user.

## Application navigation

The BridgeIT frontend contains six main pages:

| Page | Purpose |
|---|---|
| **Health** | Verify that the frontend can communicate with the backend |
| **Create** | Submit a new requirement |
| **Requirements** | Retrieve and inspect an existing requirement |
| **Analyse** | Request an AI-assisted quality analysis |
| **Validate** | Record the Business Analyst's decision |
| **Guide** | Read an in-application explanation of the workflow and common questions |

The same navigation bar is available throughout the application.

## Typical workflow

The normal BridgeIT workflow is:

`Create` → `Analyse` → `Validate` → `Requirements`

A requirement normally progresses through the following lifecycle:

`Submitted` → `Analyzed` → human decision

The human decision can then produce one of three outcomes:

- `Approve` → `Validated`;
- `Edit` → `Clarified`;
- `Reject` → `Rejected`.

The AI analysis itself never produces one of these authoritative final states.

If the Business Analyst chooses `Edit`, the requirement moves to `Clarified`. A requirement in `Clarified` state can be submitted for AI analysis again. After re-analysis, it returns to `Analyzed` and still requires a new Business Analyst decision.

The `Edit` → `Clarified` → `Analyse` refinement cycle can therefore be repeated before a final validation decision is recorded.

## 1. Check the backend connection

Open the **Health** page before starting a requirements engineering session.

The page contains the **Check backend health** action.

Use it to verify that the frontend can communicate with the BridgeIT backend before creating, analysing, or validating requirements.

If the health check succeeds, the application is ready to use.

If it fails, the backend may not be running or may not be reachable from the frontend. Refer to the [Deployment](../07-deployment/) section for instructions on starting the application.

## 2. Create a requirement

Open **Create**.

The page displays the **Submit a requirement** form.

1. Enter the requirement in the **Requirement text** field.
2. Write the requirement in plain language.
3. Select **Submit requirement**.

BridgeIT creates the requirement and assigns it a unique identifier.

The initial status is:

`Submitted`

Keep the generated requirement id. It is used to refer to the same requirement on the **Requirements**, **Analyse**, and **Validate** pages.

### Example

A requirement may initially be written as:

> The system should notify users quickly when an important event occurs.

BridgeIT stores the submitted text as the authoritative requirement until a Business Analyst explicitly edits it.

## 3. Inspect an existing requirement

Open **Requirements**.

The page provides the **Requirement id** field.

1. Paste the id of an existing requirement.
2. Select the lookup action.
3. Inspect the requirement text and its current status.

This page is useful throughout the workflow to verify the authoritative state stored by BridgeIT.

Typical statuses include:

- `Submitted`;
- `Analyzed`;
- `Clarified`;
- `Validated`;
- `Rejected`.

For example, after an AI analysis but before any human validation decision, the requirement remains `Analyzed`.

## 4. Request an AI-assisted analysis

Open **Analyse**.

The page displays **Request an AI-assisted analysis**.

1. Paste the requirement id into the **Requirement id** field.
2. Request the analysis.
3. Wait for the Gemini quality analysis to complete.

Gemini evaluates the requirement and returns a quality assessment.

The analysis may indicate that the requirement is ready for validation or that it still needs clarification.

When clarification is needed, the interface displays the issues identified by Gemini and explains why they matter.

For example, an ambiguous requirement may contain problems such as:

- an unspecified actor or recipient;
- subjective expressions such as "quickly";
- an undefined triggering condition.

After a successful AI analysis, the authoritative requirement status becomes:

`Analyzed`

This is an important BridgeIT invariant: **the AI analysis does not validate the requirement**.

Even when Gemini detects no blocking quality issues, a Business Analyst must still review the requirement on the **Validate** page.

## 5. Record a Business Analyst decision

Open **Validate**.

The page displays **Record a validation decision**.

Enter the requirement id and choose one of the three available decisions:

- **Approve**
- **Edit**
- **Reject**

### Approve

Choose **Approve** when the Business Analyst accepts the requirement as written.

The resulting status is:

`Validated`

Example lifecycle:

`Submitted` → `Analyzed` → `Validated`

### Edit

Choose **Edit** when the AI analysis or the Business Analyst review shows that the requirement needs clarification.

When **Edit** is selected, enter the revised wording in the **Edited text** field.

For example:

> The system shall notify registered administrators within 5 seconds when a critical system failure is detected.

After the decision is recorded:

- the revised text becomes the stored requirement text;
- the requirement status becomes `Clarified`.

Example lifecycle:

`Submitted` → `Analyzed` → `Clarified`


### Reject

Choose **Reject** when the requirement should not proceed.

The resulting status is:

`Rejected`

Example lifecycle:

`Submitted` → `Analyzed` → `Rejected`

## 6. Verify the final state

After recording a validation decision, return to **Requirements**.

Paste the same requirement id and retrieve it again.

Verify that:

- the displayed text is the expected authoritative text;
- the status reflects the Business Analyst's decision.

For example:

| Decision | Expected state |
|---|---|
| Approve | `Validated` |
| Edit | `Clarified` |
| Reject | `Rejected` |

For an edited requirement, the **Requirements** page should also display the revised text.

## 7. Use the in-application Guide

Open **Guide** for a shorter explanation directly inside BridgeIT.

The page explains the main workflow:

1. write a requirement;
2. request a Gemini quality check;
3. let a Business Analyst review the result;
4. inspect the requirement again whenever needed.

The Guide also contains frequently asked questions about requirement ids, AI clarification results, human validation, and edited requirements.

## Human-in-the-loop principle

The most important rule for using BridgeIT is that the AI is advisory.

Gemini can:

- analyse requirement quality;
- identify ambiguity or missing information;
- explain why an issue matters;
- indicate whether a requirement appears ready for human validation.

Gemini cannot:

- approve a requirement;
- reject a requirement;
- edit the authoritative requirement text by itself;
- determine the final authoritative requirement status.

Only an explicit Business Analyst action on the **Validate** page can result in `Validated`, `Clarified`, or `Rejected`.

This separation ensures that AI supports the requirements engineering process without replacing human accountability for validation decisions.

## Quick reference

| Task | Page |
|---|---|
| Check whether BridgeIT is connected to the backend | **Health** |
| Submit a new requirement | **Create** |
| Retrieve requirement text and status | **Requirements** |
| Request Gemini quality feedback | **Analyse** |
| Approve, edit, or reject a requirement | **Validate** |
| Read workflow help and FAQs | **Guide** |

For installation, runtime configuration, and deployment instructions, refer to the [Deployment](../07-deployment/) and [Developer Guide](../10-devguide/) sections.
