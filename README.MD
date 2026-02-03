# Agentic AI Hackathon, Challenge Repo

This repo contains the official challenge materials for a hands on hackathon focused on building agentic workflows with structured, realistic datasets. Teams will pick a track, ingest the provided case packets, then design an agent based solution that produces a clear triage output, a recommended path to action, and a short adoption plan.

## What you build

You will build a working concept that can:
- Read an intake record plus supporting documents
- Normalize messy inputs into a consistent schema
- Identify gaps, risks, and required follow ups
- Route the case to the right next step, with a rationale
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

Start here before reviewing any case assets. The official track instructions are in `challenge-docs/`.

- [`challenge-docs/Hackathon Challenges - Shared Overview.docx`](challenge-docs/Hackathon%20Challenges%20-%20Shared%20Overview.docx)  
  Shared context used across all tracks, including common assumptions, data access notes, and expectations for outputs.

- [`challenge-docs/Hackathon Challenge - AMC.docx`](challenge-docs/Hackathon%20Challenge%20-%20AMC.docx)  
  AMC track instructions, study intake scenario details, required outputs, and evaluation criteria.

- [`challenge-docs/Hackathon Challenge - EDU Track.docx`](challenge-docs/Hackathon%20Challenge%20-%20EDU%20Track.docx)  
  EDU track instructions, student success case workflow, routing rules, and escalation expectations.

- [`challenge-docs/Hackathon Challenge - Research Track.docx`](challenge-docs/Hackathon%20Challenge%20-%20Research%20Track.docx)  
  Research track instructions, compliance triage scenarios, governance and vendor review expectations.

## Suggested workflow for teams

1. Pick a track and read the matching challenge doc in `challenge-docs/`.
2. Review the shared overview and challenge doc for that track.
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