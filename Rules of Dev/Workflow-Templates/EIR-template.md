# EIR Template

version: v1.0
document_type: EIR
project: <project-name>
date: YYYY-MM-DD
eir_id: EIR-<project>-<date>-<defect_type>-v<version>
requirement_id: REQ-<id>
prd_id: PRD-<project>-<date>-v<version>
dsd_id: DSD-<project>-<date>-v<version>
pr_id: PR-<number>
owner: PM|SA|SD|QA
defect_type: REQ_DEFECT|DESIGN_DEFECT|IMPLEMENTATION_DEFECT|VALIDATION_DEFECT
defect_source: PM|SA|SD|QA
severity: low|medium|high|critical
status: open|closed

## incident_summary
- trigger_gate: G1|G2|G3|G4
- issue_description:

## root_cause
- category:
- detail:

## impact
- scope:
- user_impact:
- technical_impact:

## rollback_decision
- rollback_to_state:
- reason:

## corrective_actions
- action:
  owner:
  due_date:

## preventive_actions
- action:
  owner:
  due_date:

## closure_criteria
- criteria:
