# Hackathon Challenge | Research Track #

**Title**

Research Compliance Intake Triage

**Overview of the issue**

Research compliance triage is a classification and routing problem. Incoming requests must be evaluated for likely triggers and routed to the right review paths quickly, even when request details are incomplete.

**Current state, why it is hard today**

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
- multi site reliance and coordination
- export control or sanctioned entity screening
- biospecimens or genomics governance
- security review for tools and vendors

**Inconsistent triage**

Different reviewers interpret the same intake differently. Teams reroute cases multiple times, causing delay and frustration.

**Documentation churn**

Administrators rewrite the same context into different systems, and researchers answer the same clarifying questions repeatedly.

**Why an agentic approach makes sense**

Agentic workflows help because they:

- normalize intake fields into a consistent case summary
- map likely compliance triggers with explicit evidence and unknowns
- produce a conservative routing recommendation and priority level
- generate targeted clarifying questions that reduce back and forth
- separate reusable utilities from domain specific compliance reasoning

**Goal**

Build an agentic workflow that ingests a research intake request and outputs a triage bundle that helps administrators route the request and identify likely compliance triggers.

- What pathways are triggered
- What routing and priority are recommended
- What information is missing, and who should provide it
- What red flags require escalation

**Primary scenario**

A research team submits a request to start or modify an activity involving one or more of:

- Human subjects data
- Multi institutional collaboration
- External data sharing
- Cloud tools or AI tools
- Potentially sensitive topics or regulated data types

The intake request is partial. Your system must still produce a useful triage packet.

**Inputs you can assume**

A structured intake CSV plus additional attachments.

- Intake file: research\_intake.csv
- Additional attachments: abstract, consent notes, data flow notes, vendor snippets, IRB reference if existing

**Expected columns in research\_intake.csv**

Your workflow should handle missing values, but the CSV may include columns like:

- project\_title
- request\_type: new, amendment, data\_access, tool\_request
- principal\_investigator
- department
- collaborators
- human\_subjects: yes, no, unknown
- data\_types: PHI, PII, de\_identified, limited\_dataset, public, genomic, biospecimen
- data\_sources
- data\_sharing: none, internal, external, cross\_border, unknown
- tools\_requested
- external\_vendors
- timeline\_need: urgent, standard
- contact\_info

**Challenge data and case packets (Blob Storage)**

All hackathon case data is hosted in Azure Blob Storage. You will use one shared assets/ container for track level files, plus one container per case.

**Storage account**

- Name: criceagenthackresearch
- URI: https://criceagenthackresearch.blob.core.windows.net/

**Structure**

- assets/\
  Shared track files such as the intake CSV, policy snippets, and templates
- proj-###/\
  One container per Research project case, containing that project’s supporting documents

**Research Track, data and case packets**

**What the data represents**\
This track simulates a research intake and compliance triage workflow for new projects, data access requests, and tooling approvals. The cases are designed to trigger decisions around human subjects determination and IRB pathway, limited dataset handling and DUAs, cross border collaboration and export control screening, biospecimen governance, and vendor security posture.

**Primary intake file**

- assets/research\_intake.csv
- Link: https://criceagenthackresearch.blob.core.windows.net/assets/research\_intake.csv

**Case containers**

- Pattern: proj-###/
- Example: https://criceagenthackresearch.blob.core.windows.net/proj-001/

**What’s inside each project container**\
A mix of abstracts, data flow notes, DUA excerpts, vendor security snippets, and policy flags for routing. Some details are intentionally ambiguous, your agent should surface gaps and recommend follow ups.

**Required output**

Generate one Markdown bundle file named:

- research\_triage\_bundle.md

**Bundle requirements**

Your bundle must include these sections.

**A. Intake normalization**

- normalized project summary derived from the CSV row
- missing required fields list
- contradictions or ambiguities
- clarifying questions, prioritized

**B. Compliance trigger map**

For each trigger:

- trigger\_name
- triggered: yes, no, unknown
- why, grounded in evidence from the CSV or additional attachments
- required next step

Example trigger areas:

- IRB or ethics review
- PHI or HIPAA related handling considerations
- limited dataset handling
- DUA requirements
- multi site reliance agreements
- export control screening
- biospecimen governance
- genomic data governance
- security review for tools and vendors

**C. Routing decision**

- recommended\_status: route\_now, needs\_more\_info, escalate, hold
- routing\_targets: list of teams or boards
- priority\_level: low, medium, high, urgent
- rationale grounded in evidence

**D. Tool and vendor triage**

- tools requested and intended use
- whether each tool touches sensitive data
- minimum evidence needed to proceed with review, based on context
- watch outs and red lines, without over claiming

**E. Triage task plan**

A checklist with owners and sequencing, for example:

- collect missing details from PI
- confirm human subjects determination
- draft IRB starter checklist
- initiate DUA or reliance discussions
- route tool request to security review
- confirm data access path and approvals

**F. Evidence and assumptions**

- evidence used from the CSV fields and any additional attachments
- assumptions made due to missing data
- what information would change routing or priority

**Starter acceptance tests**

- Includes all sections A through F
- Includes recommended\_status, routing\_targets, and priority\_level
- Produces at least 12 clarifying questions, prioritized
- Includes at least 8 triggers with yes, no, or unknown and evidence
- Assigns at least 6 tasks with owners
- Lists assumptions explicitly

