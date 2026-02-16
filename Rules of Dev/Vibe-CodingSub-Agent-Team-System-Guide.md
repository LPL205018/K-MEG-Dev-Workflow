# Vibe Coding Sub-Agent Team System 使用說明（完整版本）
版本：v1.0  
狀態：Operational Guide  
最後更新：2026-02-15

## 1. 系統目標
本系統用於將需求以固定流程落地為可驗證交付，核心流程為：

`PM -> SA -> SD -> QA`

設計目標：
1. 可追溯：每次交付可對應 `REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`
2. 可阻擋：未通過 Gate 不可合併
3. 可迭代：失敗必須進入 EIR 與回退機制

## 2. 核心文件地圖
1. 角色與責任：`Sub-Agent-Team-Charter-v1.md`
2. 狀態機與四 Gate：`Workflow-State-Machine-v1.md`
3. 自動化接線：`Workflow-Automation-Wiring-v1.md`
4. 模板：`Workflow-Templates/PRD-template.md`
5. 模板：`Workflow-Templates/DSD-template.md`
6. 模板：`Workflow-Templates/QA-Report-template.md`
7. 模板：`Workflow-Templates/EIR-template.md`
8. Gate 工作流：`Engineering-Automation/CI-CD-Pipelines/gatekeeper.yml`
9. CI 工作流：`Engineering-Automation/CI-CD-Pipelines/main-pipeline.yml`
10. PR 輔助工作流：`Engineering-Automation/CI-CD-Pipelines/pr-checks.yml`

## 3. 角色定義（四代理）
1. PM Agent  
任務：產出 PRD、清除歧義、定義 AC 與範圍邊界。
2. SA Agent  
任務：產出 DSD、鎖定資料模型/API/一致性與安全策略。
3. SD Agent  
任務：依 DSD 實作程式與測試，提交可追溯 PR。
4. QA Agent  
任務：執行證偽、驗收 AC、輸出 QA 報告與缺陷結論。

## 4. 一次完整開發的標準操作
### Step A：PM 階段（G1）
1. 由 `Workflow-Templates/PRD-template.md` 建立新 PRD。
2. 填寫必要欄位：`requirement_id`、`prd_id`、`acceptance_criteria`、`scope_in`、`scope_out`、`nfr`。
3. 填寫 OAL/OAF 錨定欄位：
`anchor_check: pass`、`insufficient_data: false`、`socratic_questions:`。
4. `open_questions` 必須為 `[]` 才可進入 SA。
5. 填寫 `self_check`，phase 必須為 `PM`。

### Step B：SA 階段（G2）
1. 由 `Workflow-Templates/DSD-template.md` 建立 DSD。
2. 完成必要章節：`api_contract`、`data_schema`、`error_code_map`、`consistency_strategy`、`security_controls`。
3. 禁止 `TBD`、`TODO`。
4. 填寫 OAL/OAF 錨定欄位：
`anchor_check: pass`、`insufficient_data: false`、`socratic_questions:`。
5. 填寫 `self_check`，phase 必須為 `SA`。

### Step C：SD 階段（G3）
1. 依 DSD 開發，提交 PR。
2. PR 內容必須包含 `REQ-ID` 與 `DSD-ID`。
3. 若為 bugfix，需包含回歸測試變更（檔名含 `test` 或 `spec`）。
4. CI 必須全綠，覆蓋率門檻：
Py>=90%，TS>=80%，C#>=85%。

### Step D：QA 階段（G4）
1. 由 `Workflow-Templates/QA-Report-template.md` 建立 QA 報告。
2. 必填：`qa_report_id`、`requirement_id`、`prd_id`、`dsd_id`、`pr_id`。
3. 必須達成：`ac_pass_rate: 100%`、`critical_high_defects: 0`。
4. 必須包含追溯鏈：
`REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`。
5. 填寫 `self_check`，phase 必須為 `QA`。

## 5. 四 Gate 判定規則（摘要）
1. G1 PM Gate  
檢查 PRD 存在與欄位完整；`open_questions` 非空即 fail。
2. G2 SA Gate  
檢查 DSD 存在、核心章節完整、無 `TBD/TODO`。
3. G3 SD Gate  
檢查 PR body 追溯 ID 與 bugfix 測試要求；CI 結果由主 pipeline 控制。
4. G4 QA Gate  
檢查 QA 報告完整與驗收結果達標。

## 6. GitHub 協作與上線方式
### 6.1 必做配置
1. 將下列檔案放到 `.github/workflows/`：
- `main-pipeline.yml`
- `pr-checks.yml`
- `gatekeeper.yml`
2. 設定 Branch Protection（main/develop）：
- Require pull request before merging
- Require status checks to pass before merging
- Require at least 1 approval
- Dismiss stale approvals when new commits are pushed
3. Required checks 建議至少包含：
- `Gatekeeper / G1-PM`
- `Gatekeeper / G2-SA`
- `Gatekeeper / G3-SD`
- `Gatekeeper / G4-QA`
- `CI Pipeline - Multi-Stack (Strict Mode) / CI Result Summary`

### 6.2 CODEOWNERS
1. 建立真實 `.github/CODEOWNERS`（不可使用教學範本文字版）。
2. 依技術域指定 reviewer，避免 PR 無法自動分派。

## 7. 錯誤處理與回退
### 7.1 失敗分類
1. `REQ_DEFECT`：回退 PM。
2. `DESIGN_DEFECT`：回退 SA。
3. `IMPLEMENTATION_DEFECT`：回退 SD。
4. `VALIDATION_DEFECT`：QA 修正驗證策略後重驗。

### 7.2 EIR 強制規範
1. 失敗後先建 `EIR`，使用 `Workflow-Templates/EIR-template.md`。
2. 必填 `defect_type` 與 `defect_source`。
3. 補齊 `root_cause`、`rollback_decision`、`corrective_actions`。

### 7.3 Stop-the-line 觸發
1. 安全 Critical 漏洞。
2. 需求與實作目標衝突。
3. 追溯鏈中斷。

## 8. 關鍵操作（Quick Ops）
### 8.1 建立本次交付文件
1. 複製 PRD 模板並命名：`PRD-<project>-<date>-v1.md`
2. 複製 DSD 模板並命名：`DSD-<project>-<date>-v1.md`
3. 複製 QA 模板並命名：`QA-Report-<project>-<date>-v1.md`
4. 失敗時建立：`EIR-<project>-<date>-<type>-v1.md`

### 8.2 本地提交前檢查
1. 確認 PRD/DSD/QA 文件欄位完整。
2. 確認 PR body 有 `REQ-ID`、`DSD-ID`。
3. 確認 bugfix 包含測試檔變更。
4. 確認 CI 應跑項目可在本機先驗證。

### 8.3 常見阻擋原因
1. PRD 有 `open_questions` 未清空。
2. DSD 內含 `TBD` 或缺 `api_contract`。
3. QA 報告未達 `ac_pass_rate: 100%`。
4. PR 缺 `REQ-ID` 或 `DSD-ID`。

## 9. 推薦團隊執行節奏
1. 每個需求都以一組 PRD/DSD/QA 報告閉環。
2. 每次 Gate fail 都要有 EIR，不做口頭補救。
3. 每週檢視 Gate fail 統計，調整模板與規則而不是放寬標準。

## 10. 定義完成（DoD）
符合以下條件，視為本系統成功落地：
1. 團隊可依 `PM -> SA -> SD -> QA` 完成至少 1 個真實需求。
2. GitHub 分支保護可阻擋未通過 Gate 的 PR。
3. 每次失敗都可追溯到 `EIR + 回退節點`。
4. 模板、流程、Gate 規則三者一致，無文件漂移。
