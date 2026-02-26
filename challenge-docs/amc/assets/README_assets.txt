AMC Hackathon Assets | Clinical Study Startup Coordinator

Structure
- accountname/assets/
  - study_intake.csv
  - attachments_index.json
  - README_assets.txt
- accountname/study-00X/
  - manifest.json
  - protocol-summary-v1.txt
  - privacy-worksheet-v1.txt
  - data-management-plan-v1.txt
  - budget-worksheet-v1.txt
  - vendor-dataflow-overview-v1.txt
  - consent-draft-v1.txt
  - pi-cv-<name>.txt
  - optional: cloud certificate placeholder, depending on the study

How to use
1) Pick a row from accountname/assets/study_intake.csv.
2) Use study_id and study_container to load files for that study.
3) Load the files listed in accountname/assets/attachments_index.json or in each study's manifest.json.
4) Run your multi agent workflow to produce study_startup_bundle.md.

Notes
- These assets are synthetic and intended for hackathon evaluation only.
- Some files intentionally include unknowns to test conservative assumptions and clarifying question generation.
