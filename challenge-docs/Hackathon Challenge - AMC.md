# Hackathon Challenge | AMC Track #

**Title**

Clinical Study Startup Coordinator

**Overview of the issue**

Clinical study startup is a coordination problem. Study teams need to align protocol intent, data needs, tools, vendors, and review pathways early, often with incomplete information. Delays usually come from missing details and late discovery of privacy, security, and contracting requirements.

**Current state, why it is hard today**

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

**Why an agentic approach makes sense**

Agentic workflows help because they:

- normalize messy intake data into a consistent, reviewable bundle
- detect contradictions and missing information early
- separate reusable utility work from domain decisioning
- produce structured outputs that multiple reviewers can scan quickly
- track evidence and assumptions explicitly, reducing confusion and rework

**Goal**

Build an agentic workflow that ingests a clinical study startup request and outputs a reviewable coordination bundle that helps a study startup committee decide.

- Is the request complete enough to begin review
- What the highest risk unknowns are
- Which teams should engage next
- What missing information must be collected, and from whom

**Primary scenario**

A department submits a request to start a new clinical study that may involve:

- Patient data, biospecimens, or imaging
- Third party tools or vendors
- Data movement between systems or institutions
- Consent language constraints

The intake request is partial. Your system must still produce a useful coordination packet.

**Inputs you can assume**

A structured intake CSV plus additional attachments.

- Intake file: study\_intake.csv
- Additional attachments: protocol abstract, consent notes, data flow notes, vendor snippets

**Expected columns in study\_intake.csv**

Your workflow should handle missing values, but the CSV may include columns like:

- study\_title
- study\_type: observational, interventional, retrospective, prospective
- principal\_investigator
- sponsor\_type: industry, federal, foundation, internal
- participating\_sites: single\_site, multi\_site
- data\_types: PHI, PII, de\_identified, limited\_dataset, public, biospecimen, imaging
- data\_sources: EHR, REDCap, PACS, labs, registries, wearables, surveys
- tools\_requested
- external\_vendors
- data\_sharing: none, internal, external, cross\_border, unknown
- timeline\_need: urgent, standard
- contact\_info

**Challenge data and case packets (Blob Storage)**

All hackathon case data is hosted in Azure Blob Storage. You will use one shared assets/ container for track level files, plus one container per case.

**Storage account**

- Name: criceagenthackmc
- URI: https://criceagenthackmc.blob.core.windows.net/

**Structure**

- assets/\
  Shared track files such as the intake CSV, policy snippets, and templates
- study-###/\
  One container per AMC study, containing that study’s supporting documents

**AMC Track, data and case packets**

**What the data represents**\
This track simulates an academic medical center intake workflow for proposed studies. The cases are designed to trigger realistic triage decisions around PHI, consent posture, IRB readiness, external vendors, data sources, and governance routing.

**Primary intake file**

- assets/study\_intake.csv
- Link: https://criceagenthackmc.blob.core.windows.net/assets/study\_intake.csv

**Case containers**

- Pattern: study-###/
- Example: https://criceagenthackmc.blob.core.windows.net/study-001/

**What’s inside each study container**\
A mix of notes and draft artifacts such as protocol snippets, data source descriptions, vendor excerpts, consent notes, and other attachments. Some details are intentionally incomplete, your agent should surface gaps and recommend follow ups.

**Required output**

Generate one Markdown bundle file named:

- study\_startup\_bundle.md

**Bundle requirements**

Your bundle must include these sections.

**A. Intake normalization**

- cleaned and normalized study summary derived from the CSV row
- missing required fields list
- contradictions or ambiguities

**B. Coordination readiness decision**

- recommended\_status: proceed\_to\_review, needs\_more\_info, needs\_privacy\_review, needs\_security\_review, hold
- readiness\_score: 0 to 100
- top reasons for status
- next team assignments

**C. Data handling precheck**

- data classification summary
- where sensitive data might enter, move, or persist
- retention expectations, if provided
- de identification or limited dataset considerations
- cross site or cross border considerations, if relevant

**D. Tool and vendor triage**

- tools requested and intended use
- whether each tool touches sensitive data
- minimum evidence needed to proceed with review, based on context
- watch outs and red lines, without over claiming

**E. Study startup task plan**

A checklist with owners and sequencing, for example:

- protocol and consent review routing
- IRB preparation checklist
- data access request steps
- security questionnaire steps for third party tools
- contracting touchpoints
- DUA triggers

**F. Evidence and assumptions**

- evidence used from the CSV fields and any additional attachments
- assumptions made due to missing data
- what information would change recommended\_status

**Starter acceptance tests**

- Includes all sections A through F
- Provides recommended\_status and readiness\_score
- Produces at least 10 clarifying questions, prioritized
- Assigns at least 6 tasks with owners
- Lists assumptions explicitly

