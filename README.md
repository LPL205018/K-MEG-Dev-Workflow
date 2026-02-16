# K-MEG Dev

K-MEG Dev is a workflow-rules repository for AI-assisted software delivery, centered on:

- `PM -> SA -> SD -> QA` mandatory flow
- Human-in-the-loop approvals before SA and SD
- Gatekeeper checks (G1~G4) for merge blocking
- Full traceability: `REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`

## Repository Structure

- `Rules of Dev/`: Core protocol, workflow, quality and automation documents
- `Rules of Dev/Workflow-Templates/`: PRD/DSD/QA/EIR templates
- `Rules of Dev/Engineering-Automation/`: CI/CD, review and environment references

## First Release

Current baseline release: `v0.1.0`

This release packages the initial executable blueprint for:

- Workflow state machine and rollback mapping
- Antigravity node contract draft
- GitHub gate wiring strategy

## Suggested GitHub Setup

1. Create a new GitHub repository (private/public as needed).
2. Set branch protection for `main` and require:
   - PR review
   - Required status checks (`Gatekeeper G1~G4`, CI summary)
3. Add `.github/CODEOWNERS` for review routing.

