# Agentic AI Hackathon, Challenge Repo

This repo contains the official challenge materials for a hands on hackathon focused on building agentic workflows with structured, realistic datasets. Teams will pick a track, ingest the provided [challenge-docs](challenge-docs/), then design an agent based solution that produces a clear triage output, a recommended path to action, and a short adoption plan.

## What you build

You will build a working concept that can:
- Read an intake record plus supporting documents
- Normalize messy inputs into a consistent schema
- Identify gaps, risks, and required follow ups
- Route the case to the right step, with a rationale
- Produce a clean summary that a real operator could act on

No coding is required for the hackathon, but teams that want to implement can do so.

## Tracks

Each track includes multiple cases and a diverse mix of documents per case. All documents are provided in non binary formats such as txt, csv, and json.

### AMC Track, Academic Medical Center intake triage
Designed for regulated clinical and research adjacent workflows.
Typical decision themes include:
- PHI involvement and minimum necessary handling
- IRB readiness and consent considerations
- External vendors and collaborators
- Data security, audit, and access controls
- Routing to compliance, privacy, security, or research operations

### EDU Track, Student success intervention orchestration
Designed for campus student support and intervention coordination.
Typical decision themes include:
- Academic risk signals and confounders
- Cross office routing, advising, tutoring, financial aid, wellbeing
- Data use constraints and role based sharing
- Outreach channel rules and opt in approaches
- Escalation triggers that require human review

### Research Track, Research compliance and intake triage
Designed for research enablement with governance guardrails.
Typical decision themes include:
- Human subjects determination and IRB pathway
- Limited dataset handling and DUA requirements
- Cross border collaboration and export control screening
- Biospecimen governance and consent scope
- Tooling requests and vendor security posture

## Challenge docs

Start here. The challenge docs and official track instructions are in [challenge-docs](challenge-docs/).

- [Shared overview](challenge-docs/)  
  Shared context across all tracks, common assumptions, data access notes, and required outputs.

- [AMC track](challenge-docs/AMC%20Track.md)  
  AMC intake triage scenarios, required outputs, and evaluation criteria.

- [EDU track](challenge-docs/EDU%20Track.md)  
  Student success routing scenarios, rules, and escalation expectations.

- [Research track](challenge-docs/Research%20Track.md)  
  Research intake and compliance triage scenarios, governance, and vendor review expectations.

## API connection test prompt

- [`test-api-prompt.md`](test-api-prompt.md)  
  A minimal system prompt used to verify the Blob Storage API connections work end to end.

Use this when:
- You just created or changed the API connections (`blob_list_blobs_container`, `blob_get_blob_contents`)
- You are troubleshooting access, permissions, or unexpected empty results
- You want to confirm the agent can list blobs and retrieve a known file (example uses `criceagenthackresearch` and `assets/research_case_index.csv`)


## Suggested workflow for teams

1. Pick a track and read the matching [challenge-docs](challenge-docs/).
2. Follow the doc’s instructions to locate the supporting assets for each case.
3. Define your target output schema and decision rules.
4. Build an agent flow, then test it against all cases.
5. Prepare a short demo plus an adoption plan aligned to outcomes.

## Output expectations

A strong submission is:
- Consistent across cases
- Explicit about assumptions and missing data
- Clear about next actions and who owns them
- Conservative with sensitive data, aligned to governance
- Easy for a human operator to trust and execute