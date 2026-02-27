# Hackathon Challenge | Research Track

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
  - [Stage 2: Assemble one full case packet (for a single project)](#stage-2-assemble-one-full-case-packet-for-a-single-project)
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
  - [Expected columns in `research_intake.csv`](#expected-columns-in-research_intakecsv)
- [Challenge data and case packets](#challenge-data-and-case-packets)
  - [What the data represents](#what-the-data-represents)
  - [Case containers](#case-containers)
  - [What’s inside each project container](#whats-inside-each-project-container)
- [Required output](#required-output)
- [Bundle requirements](#bundle-requirements)
  - [A. Intake normalization](#a-intake-normalization)
  - [B. Compliance trigger map](#b-compliance-trigger-map)
  - [C. Routing decision](#c-routing-decision)
  - [D. Tool and vendor triage](#d-tool-and-vendor-triage)
  - [E. Triage task plan](#e-triage-task-plan)
  - [F. Evidence and assumptions](#f-evidence-and-assumptions)
- [Starter acceptance tests](#starter-acceptance-tests)

## Research Compliance Intake Triage

## Start here
You are building an agentic workflow that ingests a research intake request and produces a reviewable triage bundle named `research_triage_bundle.md`.

Complete Stages 1–3 first, then expand into the full bundle requirements (A–F). The end state and requirements do not change.

## Problem context

### Overview of the issue
Research compliance triage is a classification and routing problem. Incoming requests must be evaluated for likely triggers and routed to the right review paths quickly, even when request details are incomplete.

### Current state, why it is hard today

**Ambiguous request categories**  
Requests are often unclear:
- human subjects research vs quality improvement vs operations
- new study vs amendment vs data access request
- exploratory analysis vs tool request

**Many trigger pathways**  
A single project may involve multiple triggers:
- IRB or ethics review
- PHI and regulated data handling considerations
- limited dataset constraints
- DUAs and data transfer terms
- multi-site reliance and coordination
- export control or sanctioned entity screening
- biospecimens or genomics governance
- security review for tools and vendors

**Inconsistent triage**  
Different reviewers interpret the same intake differently. Teams reroute cases multiple times, causing delay and frustration.

**Documentation churn**  
Administrators rewrite the same context into different systems, and researchers answer the same clarifying questions repeatedly.

### Why an agentic approach makes sense
Agentic workflows help because they:
- normalize intake fields into a consistent case summary
- map likely compliance triggers with explicit evidence and unknowns
- produce a conservative routing recommendation and priority level
- generate targeted clarifying questions that reduce back and forth
- separate reusable utilities from domain-specific compliance reasoning

## Goal and primary scenario

### Goal
Build an agentic workflow that ingests a research intake request and outputs a triage bundle that helps administrators route the request and identify likely compliance triggers:
- What pathways are triggered
- What routing and priority are recommended
- What information is missing, and who should provide it
- What red flags require escalation

### Primary scenario
A research team submits a request to start or modify an activity involving one or more of:
- human subjects data
- multi-institutional collaboration
- external data sharing
- cloud tools or AI tools
- potentially sensitive topics or regulated data types

The intake request is partial. Your system must still produce a useful triage packet.

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
> - accountname: `criceagenthackresearch`
> - containers: `assets`, and `proj-###` (example: `proj-001`)
> - primary file to fetch as a success test: `assets/research_intake.csv`
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

### Stage 2: Assemble one full case packet (for a single project)
Goal: have an agent collaborate with the Stage 1 agent to retrieve the full packet for one project case (for example `proj-001`). The packet should include all files in that project container.

Definition of done:
- You can select one `proj-###` container
- You can retrieve all files for that project and hold them as a single “case packet” input for downstream steps

### Important note: case packets vary by design
Do not assume every `proj-###` container has the same files, the same document types, or the same level of detail. Variability is intentional. Your agent should be nimble: use what is available, flag gaps, ask clarifying questions, and make a best-effort recommendation based on evidence and stated assumptions.

### Stage 3: Normalize into a Case Summary contract
Goal: given the full case packet from Stage 2, produce a consistent structured summary that makes it easy to complete the A–F bundle later.

Definition of done:
- The summary separates facts from assumptions
- It explicitly lists missing information and contradictions or ambiguities
- It stays grounded in the case packet content, and it makes gaps and follow-ups explicit

## AUTHORITATIVE CONFIG
These values are the source of truth for this track. If anything conflicts elsewhere, this section wins.

### Storage
- Storage account name: `criceagenthackresearch`
- Storage account URI: `https://criceagenthackresearch.blob.core.windows.net/`

### Containers and structure
- `assets` contains shared track files such as the intake CSV, policy snippets, and templates
- `proj-###` contains one case per Research project case, containing that project’s supporting documents

### Primary intake file
- Path: `assets/research_intake.csv`
- Link: `https://criceagenthackresearch.blob.core.windows.net/assets/research_intake.csv`

### Tools (API connections that already exist)
Use these tool names exactly. They are the only required tool calls for Stages 1–2.

#### `blob_list_blobs_container`
**What it does:** Lists blobs within a given storage account and container.

**Inputs (parameters):**
- `accountname` (string): Storage account name (for Research: `criceagenthackresearch`)
- `container` (string): Container name (for Research: `assets` or `proj-###`)

**Output:**
- A response that includes the blobs in that container (at minimum, blob names/paths).  
  You will use one of these blob names as input to `blob_get_blob_contents`.

#### `blob_get_blob_contents`
**What it does:** Retrieves the contents of a specific blob (file) from a given storage account and container.

**Inputs (parameters):**
- `accountname` (string): Storage account name (for Research: `criceagenthackresearch`)
- `container` (string): Container name (for Research: `assets` or `proj-###`)
- `blob_name` (string): Exact blob name/path returned from `blob_list_blobs_container`

**Output:**
- The raw contents of the blob (typically text for `.csv`, `.json`, `.md`, `.txt`, etc.).  
  Your downstream steps should treat this as source material for building the case packet and bundle.

### LLM guardrail
Before you ask an LLM to generate prompts or a plan, have it restate the config above verbatim. If it cannot, paste this block again and retry. When generating prompts or code, copy `accountname`, container names, and tool names exactly from this section, do not rename or substitute.

### GitHub reference
You can browse the full folder structure and example files for all tracks in `/challenge-docs` in this repo. Use it to confirm container names, file names, and expected packet structure before generating prompts or workflows.

## Inputs

### Inputs you can assume
A structured intake CSV plus additional attachments.
- Intake file: `research_intake.csv`
- Additional attachments: abstract, consent notes, data flow notes, vendor snippets, IRB reference if existing

### Expected columns in `research_intake.csv`
Your workflow should handle missing values, but the CSV may include columns like:
- project_title
- request_type: new, amendment, data_access, tool_request
- principal_investigator
- department
- collaborators
- human_subjects: yes, no, unknown
- data_types: PHI, PII, de_identified, limited_dataset, public, genomic, biospecimen
- data_sources
- data_sharing: none, internal, external, cross_border, unknown
- tools_requested
- external_vendors
- timeline_need: urgent, standard
- contact_info

## Challenge data and case packets

### What the data represents
This track simulates a research intake and compliance triage workflow for new projects, data access requests, and tooling approvals.

The cases are designed to trigger decisions around human subjects determination and IRB pathway, limited dataset handling and DUAs, cross-border collaboration and export control screening, biospecimen governance, and vendor security posture.

### Case containers
- Pattern: `proj-###`
- Example: `https://criceagenthackresearch.blob.core.windows.net/proj-001/`

### What’s inside each project container
A mix of abstracts, data flow notes, DUA excerpts, vendor security snippets, and policy flags for routing. Some details are intentionally ambiguous. Your agent should surface gaps and recommend follow-ups.

## Required output
Generate one Markdown bundle file named:
- `research_triage_bundle.md`

## Bundle requirements
Your bundle must include these sections.

### A. Intake normalization
- normalized project summary derived from the CSV row
- missing required fields list
- contradictions or ambiguities
- clarifying questions, prioritized

### B. Compliance trigger map
For each trigger:
- trigger_name
- triggered: yes, no, unknown
- why, grounded in evidence from the CSV or additional attachments
- required next step

Example trigger areas:
- IRB or ethics review
- PHI or HIPAA related handling considerations
- limited dataset handling
- DUA requirements
- multi-site reliance agreements
- export control screening
- biospecimen governance
- genomic data governance
- security review for tools and vendors

### C. Routing decision
- recommended_status: route_now, needs_more_info, escalate, hold
- routing_targets: list of teams or boards
- priority_level: low, medium, high, urgent
- rationale grounded in evidence

### D. Tool and vendor triage
- tools requested and intended use
- whether each tool touches sensitive data
- minimum evidence needed to proceed with review, based on context
- watch-outs and red lines, without over-claiming

### E. Triage task plan
A checklist with owners and sequencing, for example:
- collect missing details from PI
- confirm human subjects determination
- draft IRB starter checklist
- initiate DUA or reliance discussions
- route tool request to security review
- confirm data access path and approvals

### F. Evidence and assumptions
- evidence used from the CSV fields and any additional attachments
- assumptions made due to missing data
- what information would change routing or priority

## Starter acceptance tests
- Includes all sections A through F
- Includes recommended_status, routing_targets, and priority_level
- Produces at least 12 clarifying questions, prioritized
- Includes at least 8 triggers with yes, no, or unknown and evidence
- Assigns at least 6 tasks with owners
- Lists assumptions explicitly
