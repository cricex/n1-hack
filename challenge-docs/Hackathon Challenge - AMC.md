# Hackathon Challenge | AMC Track

## Table of Contents
- [Start here](#start-here)
- [Problem context](#problem-context)
  - [Overview of the issue](#overview-of-the-issue)
  - [Current state, why it is hard today](#current-state-why-it-is-hard-today)
  - [Why an agentic approach makes sense](#why-an-agentic-approach-makes-sense)
- [Goal and primary scenario](#goal-and-primary-scenario)
  - [Goal](#goal)
  - [Primary scenario](#primary-scenario)
- [Stages](#stages)
  - [Stage 1: Prove tool access (2 tool calls)](#stage-1-prove-tool-access-2-tool-calls)
  - [Stage 1: Sample prompt to an LLM](#stage-1-sample-prompt-to-an-llm)
  - [Stage 2: Assemble one full case packet (for a single study)](#stage-2-assemble-one-full-case-packet-for-a-single-study)
  - [Important note: case packets vary by design](#important-note-case-packets-vary-by-design)
  - [Stage 3: Normalize into a Case Summary contract](#stage-3-normalize-into-a-case-summary-contract)
- [AUTHORITATIVE CONFIG](#authoritative-config)
  - [Storage](#storage)
  - [Containers and structure](#containers-and-structure)
  - [Primary intake file](#primary-intake-file)
  - [Tools (API connections that already exist)](#tools-api-connections-that-already-exist)
    - [`blob_list_blobs_container`](#blob_list_blobs_container)
    - [`blob_get_blob_contents`](#blob_get_blob_contents)
  - [LLM guardrail](#llm-guardrail)
  - [GitHub reference](#github-reference)
- [Inputs](#inputs)
  - [Inputs you can assume](#inputs-you-can-assume)
  - [Expected columns in `study_intake.csv`](#expected-columns-in-study_intakecsv)
- [Challenge data and case packets](#challenge-data-and-case-packets)
  - [What the data represents](#what-the-data-represents)
  - [Case containers](#case-containers)
  - [What’s inside each study container](#whats-inside-each-study-container)
- [Required output](#required-output)
- [Bundle requirements](#bundle-requirements)
  - [A. Intake normalization](#a-intake-normalization)
  - [B. Coordination readiness decision](#b-coordination-readiness-decision)
  - [C. Data handling precheck](#c-data-handling-precheck)
  - [D. Tool and vendor triage](#d-tool-and-vendor-triage)
  - [E. Study startup task plan](#e-study-startup-task-plan)
  - [F. Evidence and assumptions](#f-evidence-and-assumptions)
- [Starter acceptance tests](#starter-acceptance-tests)

## Clinical Study Startup Coordinator

## Start here
You are building an agentic workflow that ingests a clinical study startup request and produces a reviewable coordination bundle named `study_startup_bundle.md`.

Complete Stages 1–3 first, then expand into the full bundle requirements (A–F). The end state and requirements do not change.

## Problem context

### Overview of the issue
Clinical study startup is a coordination problem. Study teams must align protocol intent, data needs, tools, vendors, and review pathways early, often with incomplete information. Delays usually come from missing details and late discovery of privacy, security, and contracting requirements.

### Current state, why it is hard today

**Intake quality**  
Requests arrive as partial forms, emails, and attachments. Critical fields like data types, external sharing, and tool usage are missing or inconsistent.

**Review pathways**  
Different review groups see the request at different times:
- study operations and coordinators
- privacy and data handling reviewers
- security reviewers for tools and vendors
- contracting for third party involvement
- IRB preparation support

Without a shared packet, each team asks the same questions and reworks the same summaries.

**Risk discovery timing**  
Important issues are often discovered late:
- PHI appears in unexpected places
- vendors or tools are introduced after initial review
- cross site data sharing and DUAs surface late
- retention and logging expectations are unclear

**Coordination overhead**  
A coordinator spends time chasing stakeholders and rewriting context, not progressing the study.

### Why an agentic approach makes sense
Agentic workflows help because they:
- normalize messy intake data into a consistent, reviewable bundle
- detect contradictions and missing information early
- separate reusable utility work from domain decisioning
- produce structured outputs that multiple reviewers can scan quickly
- track evidence and assumptions explicitly to reduce confusion and rework

## Goal and primary scenario

### Goal
Build an agentic workflow that ingests a clinical study startup request and outputs a reviewable coordination bundle that helps a study startup committee decide:
- Is the request complete enough to begin review
- What the highest risk unknowns are
- Which teams should engage next
- What missing information must be collected, and from whom

### Primary scenario
A department submits a request to start a new clinical study that may involve:
- patient data, biospecimens, or imaging
- third party tools or vendors
- data movement between systems or institutions
- consent language constraints

The intake request is partial. Your system must still produce a useful coordination packet.

## Stages

### Stage 1: Prove tool access (2 tool calls)
Goal: create an agent that can:
1) list blobs in a container
2) fetch the contents of a blob from that container

Definition of done:
- You can list blobs in `assets`
- You can fetch at least one known file (for example the intake CSV) and see the raw contents

### Stage 1: Sample prompt to an LLM
Copy/paste this into your LLM of choice. It is intentionally stage-scoped, do not ask it to design the full solution yet.  
Note: This is just an example and may need to be modified by you.

> You are helping me complete **Stage 1** of an agentic hack. Do **not** design future stages.
>
> **Goal:** Create a single non-interactive agent that can make exactly two tool calls:
> 1) `blob_list_blobs_container(accountname, container)`
> 2) `blob_get_blob_contents(accountname, container, blob_name)`
>
> **AUTHORITATIVE CONFIG (copy exactly):**
> - accountname: `criceagenthackmc`
> - containers: `assets`, and `study-###` (examples: `study-001`, `study-002`, `study-003`, `study-004`)
> - primary file to fetch as a success test: `assets/study_intake.csv`
>
> **Runtime inputs to the agent (provided by upstream agents or the orchestrator):**
> - `container` (required)
> - `blob_name` (optional)
>
> **Rules:**
> - Do not ask the user questions. Assume `container` is always provided.
> - If `blob_name` is missing, first call the list tool and return the list of blob names in a structured output.
> - If `blob_name` is provided, call the get tool and return the blob contents in a structured output.
> - Copy `accountname`, container names, and tool names exactly, do not rename or substitute.
> - Keep outputs concise and machine-readable.
>
> **Deliverable:** Write a system prompt for this Stage 1 agent that includes:
> - role and goal
> - tool usage rules
> - required input fields and behavior when `blob_name` is absent
> - output schema for: (a) list result, (b) get result
> - 2 short example I/O payloads (one list, one get)

### Stage 2: Assemble one full case packet (for a single study)
Goal: have an agent collaborate with the Stage 1 agent to retrieve the full packet for one study case (for example `study-001`). The packet should include all files in that study container.

Definition of done:
- You can select one `study-###` container
- You can retrieve all files for that study and hold them as a single “case packet” input for downstream steps

### Important note: case packets vary by design
Do not assume every `study-###` container has the same files, the same document types, or the same level of detail. Variability is intentional. Your agent should be nimble: use what is available, flag gaps, ask clarifying questions, and make a best-effort recommendation based on evidence and stated assumptions.

### Stage 3: Normalize into a Case Summary contract
Goal: given the full case packet from Stage 2, produce a consistent structured summary that makes it easy to complete the A–F bundle later.

Definition of done:
- The summary separates facts from assumptions
- It explicitly lists missing information and contradictions or ambiguities
- It stays grounded in the case packet content, and it makes gaps and follow-ups explicit

## AUTHORITATIVE CONFIG
These values are the source of truth for this track. If anything conflicts elsewhere, this section wins.

### Storage
- Storage account name: `criceagenthackmc`
- Storage account URI: `https://criceagenthackmc.blob.core.windows.net/`

### Containers and structure
- `assets` contains shared track files such as the intake CSV, policy snippets, and templates
- `study-###` contains one case per study with supporting documents
- AMC case containers in this hack: `study-001`, `study-002`, `study-003`, `study-004` (pattern: `study-###`)

### Primary intake file
- Path: `assets/study_intake.csv`
- Link: `https://criceagenthackmc.blob.core.windows.net/assets/study_intake.csv`

### Tools (API connections that already exist)
Use these tool names exactly. They are the only required tool calls for Stages 1–2.

#### `blob_list_blobs_container`
**What it does:** Lists blobs within a given storage account and container.

**Inputs (parameters):**
- `accountname` (string): Storage account name (for AMC: `criceagenthackmc`)
- `container` (string): Container name (for AMC: `assets` or `study-###`)

**Output:**
- A response that includes the blobs in that container (at minimum, blob names/paths).  
  You will use one of these blob names as input to `blob_get_blob_contents`.

#### `blob_get_blob_contents`
**What it does:** Retrieves the contents of a specific blob (file) from a given storage account and container.

**Inputs (parameters):**
- `accountname` (string): Storage account name (for AMC: `criceagenthackmc`)
- `container` (string): Container name (for AMC: `assets` or `study-###`)
- `blob_name` (string): Exact blob name/path returned from `blob_list_blobs_container`

**Output:**
- The raw contents of the blob (typically text for `.csv`, `.md`, `.txt`, etc.).  
  Your downstream steps should treat this as source material for building the case packet and bundle.

### LLM guardrail
Before you ask an LLM to generate prompts or a plan, have it restate the config above verbatim. If it cannot, paste this block again and retry. When generating prompts or code, copy `accountname`, container names, and tool names exactly from this section, do not rename or substitute.

### GitHub reference
You can browse the full folder structure and example files for all tracks in `/challenge-docs` in this repo. Use it to confirm container names, file names, and expected packet structure before generating prompts or workflows.

## Inputs

### Inputs you can assume
A structured intake CSV plus additional attachments.
- Intake file: `study_intake.csv`
- Additional attachments: protocol abstract, consent notes, data flow notes, vendor snippets

### Expected columns in `study_intake.csv`
Your workflow should handle missing values, but the CSV may include columns like:
- study_title
- study_type: observational, interventional, retrospective, prospective
- principal_investigator
- sponsor_type: industry, federal, foundation, internal
- participating_sites: single_site, multi_site
- data_types: PHI, PII, de_identified, limited_dataset, public, biospecimen, imaging
- data_sources: EHR, REDCap, PACS, labs, registries, wearables, surveys
- tools_requested
- external_vendors
- data_sharing: none, internal, external, cross_border, unknown
- timeline_need: urgent, standard
- contact_info

## Challenge data and case packets

### What the data represents
This track simulates an academic medical center intake workflow for proposed studies. The cases are designed to trigger realistic triage decisions around PHI, consent posture, IRB readiness, external vendors, data sources, and governance routing.

### Case containers
- Pattern: `study-###`
- Example: `https://criceagenthackmc.blob.core.windows.net/study-001/`

### What’s inside each study container
A mix of notes and draft artifacts such as protocol snippets, data source descriptions, vendor excerpts, consent notes, and other attachments. Some details are intentionally incomplete. Your agent should surface gaps and recommend follow-ups.

## Required output
Generate one Markdown bundle file named:
- `study_startup_bundle.md`

## Bundle requirements
Your bundle must include these sections.

### A. Intake normalization
- cleaned and normalized study summary derived from the CSV row
- missing required fields list
- contradictions or ambiguities

### B. Coordination readiness decision
- recommended_status: proceed_to_review, needs_more_info, needs_privacy_review, needs_security_review, hold
- readiness_score: 0 to 100
- top reasons for status
- next team assignments

### C. Data handling precheck
- data classification summary
- where sensitive data might enter, move, or persist
- retention expectations, if provided
- de-identification or limited dataset considerations
- cross-site or cross-border considerations, if relevant

### D. Tool and vendor triage
- tools requested and intended use
- whether each tool touches sensitive data
- minimum evidence needed to proceed with review, based on context
- watch-outs and red lines, without over-claiming

### E. Study startup task plan
A checklist with owners and sequencing, for example:
- protocol and consent review routing
- IRB preparation checklist
- data access request steps
- security questionnaire steps for third party tools
- contracting touchpoints
- DUA triggers

### F. Evidence and assumptions
- evidence used from the CSV fields and any additional attachments
- assumptions made due to missing data
- what information would change recommended_status

## Starter acceptance tests
- Includes all sections A through F
- Provides recommended_status and readiness_score
- Produces at least 10 clarifying questions, prioritized
- Assigns at least 6 tasks with owners
- Lists assumptions explicitly
