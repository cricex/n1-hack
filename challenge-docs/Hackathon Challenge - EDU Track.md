# Hackathon Challenge | EDU Track #

**Title**

Student Success Intervention Orchestrator

**Overview of the issue**

Student success intervention is a coordination and constraints problem. Institutions receive many signals that a student may need support, but acting safely requires privacy constraints, careful interpretation, and clear routing across offices.

**Current state, why it is hard today**

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

**Why an agentic approach makes sense**

Agentic workflows help because they:

- summarize signals into an auditable case narrative grounded in evidence
- explicitly separate what is known from what is inferred
- recommend low risk interventions first, with constraints and required permissions
- route the case to the right office with clear next steps and owners
- incorporate fairness and data minimization checks as a first class step

**Goal**

Build an agentic workflow that ingests student success signals and outputs an intervention coordination bundle that helps staff decide.

- What category of need is most likely: academic, financial, wellbeing, accessibility, engagement, unknown
- What interventions are appropriate and low risk
- What constraints apply: privacy, consent, data minimization, role based access
- Which office should act next, and what should be collected before action
- What situations require escalation

**Primary scenario**

A set of signals indicates a student may be at risk, examples:

- missed classes, LMS inactivity, low grades, sudden performance drop
- financial hold risk, unmet aid requirements
- advising no shows
- accommodation support needs
- wellness concerns raised by staff

The intake is partial. Your system must still produce a useful coordination packet.

**Inputs you can assume**

A structured intake CSV plus additional attachments.

- Intake file: student\_success\_intake.csv
- Additional attachments: advising notes, instructor notes, student preferences if provided, policy snippets if provided

**Expected columns in student\_success\_intake.csv**

Your workflow should handle missing values, but the CSV may include columns like:

- student\_identifier: pseudonymous is fine
- term
- signals: a structured representation of signals, or multiple signal columns
- signal\_sources
- allowed\_data\_use: what the requestor says is allowed, may be unknown
- requesting\_office
- urgency\_hint: none, medium, high, unknown
- contact\_info

If signals are represented as multiple rows per student, your workflow should aggregate them into a single case bundle.

**Challenge data and case packets (Blob Storage)**

All hackathon case data is hosted in Azure Blob Storage. You will use one shared assets/ container for track level files, plus one container per case.

**Storage account**

- Name: criceagenthackedu
- URI: https://criceagenthackedu.blob.core.windows.net/

**Structure**

- assets/\
  Shared track files such as the intake CSV, policy snippets, and templates
- student-###/\
  One container per EDU student case, containing that student’s supporting documents

**EDU Track, data and case packets**

**What the data represents**\
This track simulates a student success intervention workflow that aggregates multiple signals into a single student case, then recommends the right next actions and routing. The cases are designed to test academic risk detection, cross office coordination, data use constraints, outreach channel rules, and escalation triggers that require human review.

**Primary intake file**

- assets/student_success_intake.csv
- Link: [https://criceagenthackedu.blob.core.windows.net/assets/student\_success\_intake.csv](https://criceagenthackedu.blob.core.windows.net/assets/student_success_intake.csv)

**Case containers**

- Pattern: student-###/
- Example: https://criceagenthackedu.blob.core.windows.net/student-001

**What’s inside each student container**\
A mix of advising notes, instructor notes, student preferences, LMS activity summaries, and, where applicable, wellbeing or accessibility related notes. Some details are intentionally incomplete or constrained by allowed data use, your agent should surface gaps and recommend follow ups.

**Required output**

Generate one Markdown bundle file named:

- student\_success\_bundle.md

**Bundle requirements**

Your bundle must include these sections.

**A. Intake normalization**

- normalized summary of signals derived from the CSV row(s)
- missing data list
- contradictions or ambiguities
- clarifying questions, prioritized

**B. Risk and fairness check**

- potential bias or confounding factors to watch for
- data minimization recommendations
- what should not be inferred from the available signals
- confidence level: low, medium, high

**C. Intervention routing decision**

- recommended\_status: route\_now, needs\_more\_info, escalate, hold
- student\_need\_category: academic, financial, wellbeing, accessibility, engagement, unknown
- routing\_targets: list of offices
- priority\_level: low, medium, high, urgent
- rationale grounded in evidence

**D. Intervention options**

A ranked list of interventions with:

- intervention\_name
- intended outcome
- required permissions or constraints
- minimum data required to proceed
- risk\_level: low, medium, high
- human\_review\_required: yes, no

Interventions should be framed as staff actions, not automated student messaging, unless explicitly allowed by intake constraints.

**E. Task plan**

A checklist with owners and sequencing, for example:

- confirm data use permissions
- request a brief advisor note
- schedule outreach via approved channel
- refer to tutoring or financial aid consult
- document outcome in the system of record
- define a follow up checkpoint

**F. Evidence and assumptions**

- evidence used from the CSV fields and any additional attachments
- assumptions made due to missing data
- what information would change routing or escalation

**Starter acceptance tests**

- Includes all sections A through F
- Includes recommended\_status, student\_need\_category, routing\_targets, and priority\_level
- Produces at least 10 clarifying questions, prioritized
- Provides at least 6 intervention options with risk levels and constraints
- Includes a fairness and data minimization section
- Lists assumptions explicitly
