# Antigravity 節點設定稿（PM -> SA -> SD -> QA）
版本：v1.0  
用途：將現有 workflow 正式映射到 Antigravity 自動流程

## 1. 設計目標
在 Antigravity 實現以下要求：
1. 強制流程：`PM -> SA -> SD -> QA`
2. 強制 HITL：在 SA、SD 前都要人工核准方向
3. 強制回退：失敗必須分類到 `REQ_DEFECT / DESIGN_DEFECT / IMPLEMENTATION_DEFECT / VALIDATION_DEFECT`
4. 強制追溯：每輪任務都帶 `REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`

## 2. 節點拓樸（建議）
1. `N0_INTAKE`（需求入口）
2. `N1_PM_GENERATE_PRD`
3. `N2_H1_HUMAN_APPROVAL_PM_TO_SA`
4. `N3_SA_GENERATE_DSD`
5. `N4_H2_HUMAN_APPROVAL_SA_TO_SD`
6. `N5_SD_IMPLEMENT_AND_PR`
7. `N6_CI_AND_GATEKEEPER_CHECK`
8. `N7_QA_VERIFY_AND_REPORT`
9. `N8_RELEASE_READY`
10. `N9_EIR_AND_ROLLBACK`（失敗共用節點）

## 3. 節點輸入/輸出契約
### N0_INTAKE
Input：
- `requirement_text`
- `requester`
- `business_context`

Output：
- `REQ-ID`
- `intent_packet`

### N1_PM_GENERATE_PRD
Input：
- `REQ-ID`
- `intent_packet`

Output：
- `PRD-ID`
- `prd_file_path`
- `pm_gate_candidate=true`

模板規範：
- 必須依 `Workflow-Templates/PRD-template.md`

### N2_H1_HUMAN_APPROVAL_PM_TO_SA
Input：
- `PRD-ID`
- `prd_file_path`

Output：
- `H1_STATUS=APPROVED|REJECTED`
- `H1_APPROVER`
- `H1_COMMENT`

規則：
- `REJECTED` -> `N9_EIR_AND_ROLLBACK`（`REQ_DEFECT`）
- `APPROVED` -> `N3_SA_GENERATE_DSD`

### N3_SA_GENERATE_DSD
Input：
- `REQ-ID`
- `PRD-ID`

Output：
- `DSD-ID`
- `dsd_file_path`
- `sa_gate_candidate=true`

模板規範：
- 必須依 `Workflow-Templates/DSD-template.md`

### N4_H2_HUMAN_APPROVAL_SA_TO_SD
Input：
- `DSD-ID`
- `dsd_file_path`

Output：
- `H2_STATUS=APPROVED|REJECTED`
- `H2_APPROVER`
- `H2_COMMENT`

規則：
- `REJECTED` -> `N9_EIR_AND_ROLLBACK`（`DESIGN_DEFECT`）
- `APPROVED` -> `N5_SD_IMPLEMENT_AND_PR`

### N5_SD_IMPLEMENT_AND_PR
Input：
- `REQ-ID`
- `PRD-ID`
- `DSD-ID`

Output：
- `PR-ID`
- `pr_url`
- `commit_refs[]`
- `traceability_ok=true|false`

規則：
- PR body 必須含 `REQ-ID`、`DSD-ID`

### N6_CI_AND_GATEKEEPER_CHECK
Input：
- `PR-ID`
- `pr_url`

Output：
- `G1_STATUS`
- `G2_STATUS`
- `G3_STATUS`
- `G4_STATUS`
- `CI_STATUS`
- `FAILED_GATE`（若失敗）

規則：
- 任一 fail -> `N9_EIR_AND_ROLLBACK`
- 全 pass -> `N7_QA_VERIFY_AND_REPORT`

### N7_QA_VERIFY_AND_REPORT
Input：
- `REQ-ID`
- `PRD-ID`
- `DSD-ID`
- `PR-ID`

Output：
- `QA-REPORT-ID`
- `qa_report_file_path`
- `qa_result=PASS|FAIL`

模板規範：
- 必須依 `Workflow-Templates/QA-Report-template.md`

規則：
- `FAIL` -> `N9_EIR_AND_ROLLBACK`（`VALIDATION_DEFECT` 或對應分類）
- `PASS` -> `N8_RELEASE_READY`

### N8_RELEASE_READY
Output：
- `release_candidate=true`
- `workflow_status=DONE`

### N9_EIR_AND_ROLLBACK
Input：
- `failure_context`
- `failed_node`
- `failed_gate`

Output：
- `EIR-ID`
- `defect_type`
- `defect_source`
- `rollback_target_node`

模板規範：
- 必須依 `Workflow-Templates/EIR-template.md`

回退路由：
- `REQ_DEFECT` -> `N1_PM_GENERATE_PRD`
- `DESIGN_DEFECT` -> `N3_SA_GENERATE_DSD`
- `IMPLEMENTATION_DEFECT` -> `N5_SD_IMPLEMENT_AND_PR`
- `VALIDATION_DEFECT` -> `N7_QA_VERIFY_AND_REPORT`

## 4. 狀態機（Antigravity 版）
主狀態：
1. `INTAKE_READY`
2. `PM_READY`
3. `H1_PENDING`
4. `SA_READY`
5. `H2_PENDING`
6. `SD_READY`
7. `CI_GATE_PENDING`
8. `QA_READY`
9. `RELEASE_READY`
10. `DONE`

異常狀態：
1. `FAILED_REQUIRES_EIR`
2. `ROLLED_BACK`
3. `STOP_THE_LINE`

`STOP_THE_LINE` 觸發：
1. Security critical
2. 追溯鏈斷裂
3. 需求方向衝突（H1/H2 重複拒絕）

## 5. HITL（Human-in-the-loop）落地規則
### H1（PM -> SA）
必填：
- `方向符合需求? (Yes/No)`
- `AC 是否可測? (Yes/No)`
- `Scope In/Out 是否清楚? (Yes/No)`

阻擋條件：
- 任一項 No -> 不可進 SA

### H2（SA -> SD）
必填：
- `架構方向正確? (Yes/No)`
- `API/Data 契約可實作? (Yes/No)`
- `風險策略可執行? (Yes/No)`

阻擋條件：
- 任一項 No -> 不可進 SD

## 6. 最小事件資料結構（建議）
```json
{
  "workflow_id": "WF-2026-0001",
  "requirement_id": "REQ-123",
  "prd_id": "PRD-xxx",
  "dsd_id": "DSD-xxx",
  "pr_id": "PR-456",
  "qa_report_id": "QA-Report-xxx",
  "current_node": "N6_CI_AND_GATEKEEPER_CHECK",
  "gate_status": {
    "G1": "PASS",
    "G2": "PASS",
    "G3": "PASS",
    "G4": "PASS"
  },
  "hitl": {
    "H1_STATUS": "APPROVED",
    "H2_STATUS": "APPROVED"
  },
  "status": "DONE"
}
```

## 7. Antigravity 參數建議
1. 決策溫度：`0.1`
2. 重試策略：每節點最多 1 次自動重試，之後強制人介入
3. 逾時策略：H1/H2 超過 SLA（建議 24h）自動提醒
4. 審計日誌：所有狀態轉移都記錄 `who/when/why`

## 8. 部署步驟（實務）
1. 在 Antigravity 建立 10 個節點（N0~N9）
2. 設定路由條件（APPROVED/PASS 才可往下）
3. 連接 GitHub webhook（PR opened/sync/checks complete）
4. 將 G1~G4 與 CI 狀態映射到 N6 輸入
5. 啟用 H1/H2 人工審批 UI（必填問題）
6. 先跑 1 個試點 REQ，驗證成功與失敗路徑

## 9. 驗收標準（Go-Live）
1. 未通過 H1/H2 無法進入下一節點
2. 任一 Gate 失敗時自動進 N9 並產 EIR
3. 成功流程可完整產生追溯鏈 ID
4. 失敗流程可正確回退至指定節點
