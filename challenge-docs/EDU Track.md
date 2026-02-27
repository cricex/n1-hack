# Hackathon Challenge | EDU Track

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
  - [Stage 2: Assemble one full case packet (for a single student)](#stage-2-assemble-one-full-case-packet-for-a-single-student)
  - [Important note: case packets vary by design](#important-note-case-packets-vary-by-design)
  - [Stage 3: Normalize into a Case Summary contract](#stage-3-normalize-into-a-case-summary-contract)
- [AUTHORITATIVE CONFIG](#authoritative-config)
  - [Storage](#storage)
  - [Containers and structure](#containers-and-structure)
  - [Primary index file (start here)](#primary-index-file-start-here)
  - [Additional reference (not required)](#additional-reference-not-required)
  - [Tools (API connections that already exist)](#tools-api-connections-that-already-exist)
    - [`blob_list_blobs_container`](#blob_list_blobs_container)
    - [`blob_get_blob_contents`](#blob_get_blob_contents)
  - [LLM guardrail](#llm-guardrail)
  - [GitHub reference](#github-reference)
- [Inputs](#inputs)
  - [Inputs you can assume](#inputs-you-can-assume)
  - [Expected fields in the case index](#expected-fields-in-the-case-index)
- [Challenge data and case packets](#challenge-data-and-case-packets)
  - [What the data represents](#what-the-data-represents)
  - [Case containers](#case-containers)
  - [What’s inside each student container](#whats-inside-each-student-container)
- [Required output](#required-output)
- [Bundle requirements](#bundle-requirements)
  - [A. Intake normalization](#a-intake-normalization)
  - [B. Risk and fairness check](#b-risk-and-fairness-check)
  - [C. Intervention routing decision](#c-intervention-routing-decision)
  - [D. Intervention options](#d-intervention-options)
  - [E. Task plan](#e-task-plan)
  - [F. Evidence and assumptions](#f-evidence-and-assumptions)
- [Starter acceptance tests](#starter-acceptance-tests)

## Student Success Intervention Orchestrator

## Start here
You are building an agentic workflow that ingests student success signals and produces a reviewable intervention coordination bundle named `student_success_bundle.md`.

Complete Stages 1–3 first, then expand into the full bundle requirements (A–F). The end state and requirements do not change.

## Problem context

### Overview of the issue
Student success intervention is a coordination and constraints problem. Institutions receive many signals that a student may need support, but acting safely requires privacy constraints, careful interpretation, and clear routing across offices.

### Current state, why it is hard today

**Noisy and incomplete signals**  
Signals are often partial and can be misleading:
- LMS inactivity does not always indicate disengagement
- missed appointments can be caused by schedule or access issues
- grade drops can be temporary or context dependent

**Constraints across offices**  
Different offices have different permissions and responsibilities:
- advising, tutoring, financial aid, accessibility services
- counseling and dean of students functions for higher urgency cases
- residence life and student affairs teams

Not every office can access every data source, and allowed use may be unclear.

**Risk of overreach**  
Poorly designed interventions can:
- violate privacy expectations
- trigger unfair or biased actions
- create student distrust
- escalate unnecessarily

**Operational load**  
Staff time is limited. Without structured triage, teams spend time searching for context, not coordinating support.

### Why an agentic approach makes sense
Agentic workflows help because they:
- summarize signals into an auditable case narrative grounded in evidence
- explicitly separate what is known from what is inferred
- recommend low risk interventions first, with constraints and required permissions
- route the case to the right office with clear next steps and owners
- incorporate fairness and data minimization checks as a first class step

## Goal and primary scenario

### Goal
Build an agentic workflow that ingests student success signals and outputs an intervention coordination bundle that helps staff decide:
- What category of need is most likely: academic, financial, wellbeing, accessibility, engagement, unknown
- What interventions are appropriate and low risk
- What constraints apply: privacy, consent, data minimization, role based access
- Which office should act next, and what should be collected before action
- What situations require escalation

### Primary scenario
A set of signals indicates a student may be at risk, examples:
- missed classes, LMS inactivity, low grades, sudden performance drop
- financial hold risk, unmet aid requirements
- advising no shows
- accommodation support needs
- wellness concerns raised by staff

The intake is partial. Your system must still produce a useful coordination packet.

## Stages

### Stage 1: Prove tool access (2 tool calls)
Goal: create an agent that can:
1) list blobs in a container
2) fetch the contents of a blob from that container

Definition of done:
- You can list blobs in `assets`
- You can fetch the case index file (`assets/edu_case_index.csv`) and see the raw contents

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
> - accountname: `criceagenthackedu`
> - containers: `assets`, and `student-###` (examples: `student-001`, `student-002`, `student-003`, `student-004`)
> - primary file to fetch as a success test: `assets/edu_case_index.csv`
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

### Stage 2: Assemble one full case packet (for a single student)
Goal: have an agent collaborate with the Stage 1 agent to retrieve the full packet for one student case (for example `student-001`). The packet should include all files in that student container.

Use `assets/edu_case_index.csv` to select the student container and to understand which files exist for that student.

Definition of done:
- You can select one `student-###` container
- You can retrieve all files for that student and hold them as a single “case packet” input for downstream steps

### Important note: case packets vary by design
Do not assume every `student-###` container has the same files, the same document types, or the same level of detail. Variability is intentional. Your agent should be nimble: use what is available, flag gaps, respect allowed data use constraints, and make a best-effort recommendation based on evidence and stated assumptions.

### Stage 3: Normalize into a Case Summary contract
Goal: given the full case packet from Stage 2, produce a consistent structured summary that makes it easy to complete the A–F bundle later.

Definition of done:
- The summary separates facts from assumptions
- It explicitly lists missing information and contradictions or ambiguities
- It stays grounded in the case packet content, and it makes gaps and follow-ups explicit

## AUTHORITATIVE CONFIG
These values are the source of truth for this track. If anything conflicts elsewhere, this section wins.

### Storage
- Storage account name: `criceagenthackedu`
- Storage account URI: `https://criceagenthackedu.blob.core.windows.net/`

### Containers and structure
- `assets` contains shared track files such as the case index, policy snippets, and reference files
- `student-###` contains one case per student with that student’s supporting documents
- EDU case containers in this hack: `student-001`, `student-002`, `student-003`, `student-004` (pattern: `student-###`)

### Primary index file (start here)
- Path: `assets/edu_case_index.csv`
- Purpose: use this to get the full list of `student_id` values and locate each student’s case container and files

### Additional reference (not required)
- `assets/student_success_intake.csv` (may be incomplete by design, do not assume it contains everything you need)

### Tools (API connections that already exist)
Use these tool names exactly. They are the only required tool calls for Stages 1–2.

#### `blob_list_blobs_container`
**What it does:** Lists blobs within a given storage account and container.

**Inputs (parameters):**
- `accountname` (string): Storage account name (for EDU: `criceagenthackedu`)
- `container` (string): Container name (for EDU: `assets` or `student-###`)

**Output:**
- A response that includes the blobs in that container (at minimum, blob names/paths).  
  You will use one of these blob names as input to `blob_get_blob_contents`.

#### `blob_get_blob_contents`
**What it does:** Retrieves the contents of a specific blob (file) from a given storage account and container.

**Inputs (parameters):**
- `accountname` (string): Storage account name (for EDU: `criceagenthackedu`)
- `container` (string): Container name (for EDU: `assets` or `student-###`)
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
A case index CSV plus student-specific supporting documents in the case container.
- Primary index file: `edu_case_index.csv`
- Supporting documents: advising notes, instructor notes, signal context, student preference notes, and policy snippets as provided

### Expected fields in the case index
Your workflow should handle missing values, but `edu_case_index.csv` is expected to include enough information to:
- enumerate `student_id` values
- map each `student_id` to a `student-###` container
- optionally indicate which supporting files exist or are most important

If the index includes multiple rows per student, your workflow should aggregate them into a single case packet.

## Challenge data and case packets

### What the data represents
This track simulates a student success intervention workflow that aggregates multiple signals into a single student case, then recommends the right next actions and routing.

The cases are designed to test academic risk detection, cross-office coordination, data use constraints, outreach channel rules, and escalation triggers that require human review.

### Case containers
- Pattern: `student-###`
- Example: `https://criceagenthackedu.blob.core.windows.net/student-001/`

### What’s inside each student container
A mix of advising notes, instructor notes, signal context, student preferences, and, where applicable, policy snippets.

Some details are intentionally incomplete or constrained by allowed data use. Your agent should surface gaps and recommend follow-ups.

## Required output
Generate one Markdown bundle file named:
- `student_success_bundle.md`

## Bundle requirements
Your bundle must include these sections.

### A. Intake normalization
- normalized summary of signals derived from the case packet and index
- missing data list
- contradictions or ambiguities
- clarifying questions, prioritized

### B. Risk and fairness check
- potential bias or confounding factors to watch for
- data minimization recommendations
- what should not be inferred from the available signals
- confidence level: low, medium, high

### C. Intervention routing decision
- recommended_status: route_now, needs_more_info, escalate, hold
- student_need_category: academic, financial, wellbeing, accessibility, engagement, unknown
- routing_targets: list of offices
- priority_level: low, medium, high, urgent
- rationale grounded in evidence

### D. Intervention options
A ranked list of interventions with:
- intervention_name
- intended outcome
- required permissions or constraints
- minimum data required to proceed
- risk_level: low, medium, high
- human_review_required: yes, no

Interventions should be framed as staff actions, not automated student messaging, unless explicitly allowed by intake constraints.

### E. Task plan
A checklist with owners and sequencing, for example:
- confirm data use permissions
- request a brief advisor note
- schedule outreach via approved channel
- refer to tutoring or financial aid consult
- document outcome in the system of record
- define a follow-up checkpoint

### F. Evidence and assumptions
- evidence used from the case index and supporting documents
- assumptions made due to missing data
- what information would change routing or escalation

## Starter acceptance tests
- Includes all sections A through F
- Includes recommended_status, student_need_category, routing_targets, and priority_level
- Produces at least 10 clarifying questions, prioritized
- Provides at least 6 intervention options with risk levels and constraints
- Includes a fairness and data minimization section
- Lists assumptions explicitly
