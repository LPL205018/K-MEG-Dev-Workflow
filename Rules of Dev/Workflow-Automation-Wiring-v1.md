# Workflow Automation Wiring v1
版本：v1.0  
狀態：Executable-Blueprint  
依賴文件：`Sub-Agent-Team-Charter-v1.md`, `Workflow-State-Machine-v1.md`  
最後更新：2026-02-13

## 1. 目的
將四步流程 `PM -> SA -> SD -> QA` 與四個 Gate（G1~G4）映射到 GitHub 自動化檢查，形成可阻擋 merge 的執行機制。

## 2. 實作策略
採用兩層機制：
1. `Layer A`：現有 `main-pipeline.yml` 負責 SD 技術質量（lint/type/test/security）。
2. `Layer B`：新增 `gatekeeper.yml` 負責流程門禁（PRD/DSD/QA/追溯鏈）。

Template 套件（Step 4）：
1. `Workflow-Templates/PRD-template.md`
2. `Workflow-Templates/DSD-template.md`
3. `Workflow-Templates/QA-Report-template.md`
4. `Workflow-Templates/EIR-template.md`

## 3. Gate 對應檢查點
### G1 PM Gate（需求完整性）
檢查項：
1. PR 內容包含 PRD 文件：`PRD-*-v*.md`
2. PRD 含固定欄位（模板化）：
- `requirement_id:`
- `prd_id:`
- `anchor_check: pass`
- `insufficient_data: false`
- `socratic_questions:`
- `## acceptance_criteria`
- `## scope_in`
- `## scope_out`
- `## nfr`
- `## open_questions`
3. 必須含 `## self_check` 且 `phase: PM`
4. `open_questions` 必須為空清單 `[]`

阻擋條件：
- 任一檢查失敗即 fail。

### G2 SA Gate（設計可實作性）
檢查項：
1. PR 內容包含 DSD 文件：`DSD-*-v*.md`
2. DSD 含固定欄位（模板化）：
- `design_id:`
- `prd_id:`
- `requirement_id:`
- `anchor_check: pass`
- `insufficient_data: false`
- `socratic_questions:`
- `## api_contract`
- `## data_schema`
- `## error_code_map`
- `## consistency_strategy`
- `## security_controls`
3. 必須含 `## self_check` 且 `phase: SA`
4. 禁止出現 `TBD`、`TODO` 於 DSD 核心段落

阻擋條件：
- 任一檢查失敗即 fail。

### G3 SD Gate（實作品質）
檢查項：
1. `main-pipeline.yml` 必須成功（Python/TS/C#/Cross-stack）
2. PR 描述必須出現 `REQ-ID` 與 `DSD-ID`
3. Bugfix 類型變更需包含回歸測試檔案改動（暫以檔名關鍵字 `test` 檢查）

阻擋條件：
- 任一檢查失敗即 fail。

### G4 QA Gate（驗證放行）
檢查項：
1. PR 內容包含 QA 報告：`QA-Report-*-v*.md`
2. QA 報告必須含固定欄位：
- `qa_report_id:`
- `requirement_id:`
- `prd_id:`
- `dsd_id:`
- `pr_id:`
- `anchor_check: pass`
- `insufficient_data: false`
- `socratic_questions:`
- `ac_pass_rate: 100%`
- `critical_high_defects: 0`
3. 必須含 `## self_check` 且 `phase: QA`
4. 必須含追溯鏈字串：
- `REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`

阻擋條件：
- 任一檢查失敗即 fail。

## 4. 事件與回退落地
自動化流程不直接改狀態，而是輸出標準 Gate 事件（log + artifact）：

```yaml
gate_event:
  gate_id: G1|G2|G3|G4
  status: PASS|FAIL|STOP
  defect_type: NONE|REQ_DEFECT|DESIGN_DEFECT|IMPLEMENTATION_DEFECT|VALIDATION_DEFECT
  requirement_id: REQ-XXX
  design_id: DSD-XXX
  pr_number: 123
```

失敗時由 reviewer/QA 依 `Workflow-State-Machine-v1.md` 回退矩陣執行回退。

## 5. GitHub Branch Protection 建議值
main/develop 必須啟用：
1. Require pull request before merging
2. Require status checks to pass before merging
3. Required checks:
- `Gatekeeper / G1-PM`
- `Gatekeeper / G2-SA`
- `Gatekeeper / G3-SD`
- `Gatekeeper / G4-QA`
- `CI Pipeline - Multi-Stack (Strict Mode) / CI Result Summary`
4. Require at least 1 approval
5. Dismiss stale approvals when new commits are pushed

## 6. 與現有文件的差異修正
### 6.1 `pr-checks.yml` 修正點
目前 `Check test coverage impact` 使用 `usage:`，應改為 `run:` 才會執行。

### 6.2 `CODEOWNERS`
目前內容為教學模板，落地時需建立真正 `.github/CODEOWNERS` 供自動分派審查者。

## 7. Step 3 驗收（Definition of Done）
1. 四個 Gate 均有可執行的 GitHub job
2. main/develop 可配置為「未過 Gate 無法 merge」
3. 至少 3 類失敗可被自動識別：需求缺失、設計缺失、追溯缺失
4. 與 Step 1/2 無語義衝突
