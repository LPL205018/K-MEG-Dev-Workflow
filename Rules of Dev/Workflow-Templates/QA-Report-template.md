# QA Report Template

version: v1.0
document_type: QA_REPORT
project: <project-name>
date: YYYY-MM-DD
qa_report_id: QA-Report-<project>-<date>-v<version>
requirement_id: REQ-<id>
prd_id: PRD-<project>-<date>-v<version>
dsd_id: DSD-<project>-<date>-v<version>
pr_id: PR-<number>
owner: QA
status: fail|pass
anchor_check: pass|fail
insufficient_data: false
socratic_questions: []

## test_summary
- total_cases:
- passed_cases:
- failed_cases:

## acceptance_result
- ac_id:
  result: pass|fail
  evidence:

ac_pass_rate: 100%

## defect_summary
- defect_id:
  severity: low|medium|high|critical
  status: open|closed

critical_high_defects: 0

## regression_result
- suite:
  result: pass|fail

## security_result
- scan:
  result: pass|fail
  finding_count:

## traceability_chain
REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID

## self_check
- phase: QA
- logic_consistency: Verified
- boundary_coverage: Yes|No
- next_action: <single actionable next step>
