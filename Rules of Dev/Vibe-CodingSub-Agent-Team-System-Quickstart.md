# Vibe Coding Sub-Agent Team System（10 分鐘速讀版）
版本：v1.0  
用途：新成員 Day 1 快速上手

## 1. 你只要記住一件事
所有需求都走這條路：

`PM -> SA -> SD -> QA`

沒過 Gate，不能 merge。

## 2. 你會用到的 5 份文件
1. `Sub-Agent-Team-Charter-v1.md`
2. `Workflow-State-Machine-v1.md`
3. `Engineering-Automation/CI-CD-Pipelines/gatekeeper.yml`
4. `Workflow-Templates/PRD-template.md`
5. `Workflow-Templates/DSD-template.md`
6. `Workflow-Templates/QA-Report-template.md`

## 3. 四個角色做什麼
1. PM：寫清楚需求與驗收（PRD）。
2. SA：把需求變成可實作設計（DSD）。
3. SD：照 DSD 寫程式與測試。
4. QA：證明交付符合 AC，出 QA 報告。

## 4. 每個階段最小完成條件
### G1 PM
1. 有 `PRD-*-vX.md`。
2. 有 `acceptance_criteria/scope_in/scope_out/nfr`。
3. `open_questions` 必須是 `[]`。
4. `anchor_check: pass`、`insufficient_data: false`、`self_check phase: PM`。

### G2 SA
1. 有 `DSD-*-vX.md`。
2. 有 `api_contract/data_schema/error_code_map/consistency_strategy/security_controls`。
3. 不得出現 `TBD` 或 `TODO`。
4. `anchor_check: pass`、`insufficient_data: false`、`self_check phase: SA`。

### G3 SD
1. PR body 必須有 `REQ-ID` 與 `DSD-ID`。
2. bugfix 必須帶測試檔改動（`test/spec`）。
3. CI 要全綠（含覆蓋率門檻）。

### G4 QA
1. 有 `QA-Report-*-vX.md`。
2. `ac_pass_rate: 100%`。
3. `critical_high_defects: 0`。
4. `self_check phase: QA`。

## 5. 你每天照做的流程
1. 用模板建立或更新 PRD/DSD/QA 報告。
2. 開發前先確認自己在正確階段，不跨階段工作。
3. 提交 PR 時放入 `REQ-ID`、`DSD-ID`。
4. 看 GitHub checks：G1~G4 + CI Summary 全通過才可合併。

## 6. 失敗時怎麼做
1. 先寫 EIR（`Workflow-Templates/EIR-template.md`）。
2. 判斷錯誤來源：
- `REQ_DEFECT` 回 PM
- `DESIGN_DEFECT` 回 SA
- `IMPLEMENTATION_DEFECT` 回 SD
- `VALIDATION_DEFECT` 回 QA
3. 修正後重跑 Gate。

## 7. 最常見卡關點
1. PRD 的 `open_questions` 沒清空。
2. DSD 還有 `TBD/TODO`。
3. PR body 沒填 `REQ-ID` / `DSD-ID`。
4. QA 報告不是 `ac_pass_rate: 100%` 或 `critical_high_defects: 0`。

## 8. GitHub 上線必做（一次性）
1. 把 workflow 放到 `.github/workflows/`：
- `main-pipeline.yml`
- `pr-checks.yml`
- `gatekeeper.yml`
2. 設定 Branch Protection Required Checks：
- `Gatekeeper / G1-PM`
- `Gatekeeper / G2-SA`
- `Gatekeeper / G3-SD`
- `Gatekeeper / G4-QA`
- `CI Pipeline - Multi-Stack (Strict Mode) / CI Result Summary`
3. 建立真實 `.github/CODEOWNERS`。

## 9. 完成定義（你可以自我判斷是否合格）
1. 需求能完整走完 `PM -> SA -> SD -> QA`。
2. Gate 失敗時有 EIR 且能回到正確上游步驟。
3. 未通過 Gate 的 PR 無法 merge。
