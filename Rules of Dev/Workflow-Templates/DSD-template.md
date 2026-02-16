# DSD Template

version: v1.0
document_type: DSD
project: <project-name>
date: YYYY-MM-DD
design_id: DSD-<project>-<date>-v<version>
prd_id: PRD-<project>-<date>-v<version>
requirement_id: REQ-<id>
owner: SA
status: draft|approved
anchor_check: pass|fail
insufficient_data: false
socratic_questions: []

## architecture_pattern
- pattern:
- rationale:

## module_boundaries
- module:
  responsibility:
  dependency:

## data_schema
- entity:
  fields:
  constraints:
  indexes:

## api_contract
- endpoint:
  method:
  request_schema:
  response_schema:
  error_codes:

## error_code_map
- code:
  meaning:
  handling:

## sequence_or_pseudocode
- flow_name:
  steps:

## consistency_strategy
- model:
- conflict_resolution:

## security_controls
- control:
  threshold:

## traceability_map
- prd_feature_id:
  design_component:

## self_check
- phase: SA
- logic_consistency: Verified
- boundary_coverage: Yes|No
- next_action: <single actionable next step>
