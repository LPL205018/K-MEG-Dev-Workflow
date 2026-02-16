# Automated Vibe Workflow 上線 Day-1 執行清單
版本：v1.0  
用途：在真實專案 repo 完成首次正式上線

## A. 上線前確認（30 分鐘）
- [ ] 已確認流程文件版本固定：
  - `Sub-Agent-Team-Charter-v1.md`
  - `Workflow-State-Machine-v1.md`
  - `Workflow-Automation-Wiring-v1.md`
  - `Vibe-CodingSub-Agent-Team-System-Guide.md`
  - `Vibe-CodingSub-Agent-Team-System-Quickstart.md`
- [ ] 已確認模板可用：
  - `Workflow-Templates/PRD-template.md`
  - `Workflow-Templates/DSD-template.md`
  - `Workflow-Templates/QA-Report-template.md`
  - `Workflow-Templates/EIR-template.md`
- [ ] 決定試點需求（建議中小型變更，非高風險核心模組）
- [ ] 指定四角色 owner（PM/SA/SD/QA）與一位流程負責人（Orchestrator）

## B. Repo 實際部署（60 分鐘）
- [ ] 建立 `.github/workflows/` 目錄（若不存在）
- [ ] 複製 workflow 到真實 repo：
  - [ ] `main-pipeline.yml`
  - [ ] `pr-checks.yml`
  - [ ] `gatekeeper.yml`
- [ ] 建立真實 `.github/CODEOWNERS`（非教學範本）
- [ ] 建立或更新 `.github/PULL_REQUEST_TEMPLATE.md`（至少包含 `REQ-ID`、`DSD-ID`、測試證據、H1/H2 欄位）
- [ ] 在 repo root 放置模板資料夾或 docs 路徑（團隊可存取）

## C. GitHub Branch Protection（20 分鐘）
目標分支：`main`、`develop`

- [ ] `Require a pull request before merging`
- [ ] `Require status checks to pass before merging`
- [ ] Required checks 加入：
  - [ ] `Gatekeeper / G1-PM`
  - [ ] `Gatekeeper / G2-SA`
  - [ ] `Gatekeeper / G3-SD`
  - [ ] `Gatekeeper / G4-QA`
  - [ ] `CI Pipeline - Multi-Stack (Strict Mode) / CI Result Summary`
- [ ] `Require at least 1 approval`
- [ ] `Dismiss stale pull request approvals when new commits are pushed`

## D. HITL（Human-in-the-loop）策略（20 分鐘）
- [ ] 定義 H1（PM -> SA 前）審批者群組（PM Lead / PO）
- [ ] 定義 H2（SA -> SD 前）審批者群組（SA Lead / Tech Lead）
- [ ] 設定規則：未完成 H1/H2，不可進入下一階段
- [ ] PR 模板增加欄位：
  - [ ] `H1 Status: Approved/Rejected/Pending`
  - [ ] `H2 Status: Approved/Rejected/Pending`
  - [ ] `Approver`
  - [ ] `Decision Date`

## E. 首次試點演練（90-150 分鐘）
試點建議：1 個 REQ，完整走完 `PM -> SA -> SD -> QA`

- [ ] 建立 `REQ-ID`
- [ ] PM 產出 PRD（用模板）
- [ ] 完成 H1 批准
- [ ] SA 產出 DSD（用模板）
- [ ] 完成 H2 批准
- [ ] SD 開發與測試，PR 填入 `REQ-ID`、`DSD-ID`
- [ ] QA 產出 QA 報告（用模板）
- [ ] Gatekeeper + CI 全通過
- [ ] 合併 PR

## F. 失敗演練（必做，30-45 分鐘）
目的：確認系統真的會阻擋與回退，不是只有文件

- [ ] 故意讓 G1 失敗（例如 PRD 缺 `open_questions` 或非 `[]`）
- [ ] 故意讓 G2 失敗（DSD 放一個 `TBD`）
- [ ] 故意讓 G4 失敗（QA 報告 `ac_pass_rate` 非 `100%`）
- [ ] 每次失敗都建立 `EIR` 並記錄回退階段

## G. Day-1 驗收（Go/No-Go）
Go 條件（全部滿足）：
- [ ] 四個 Gate 都能成功阻擋不合規 PR
- [ ] 至少 1 個試點需求成功 merge
- [ ] 至少 1 次失敗案例有 EIR 與正確回退
- [ ] H1/H2 真的能卡住流程

No-Go 條件（任一出現）：
- [ ] Required checks 可被繞過
- [ ] PR 模板無法強制追溯欄位
- [ ] 團隊角色責任不清導致流程停滯

## H. Day-2~Day-7 運營動作
- [ ] 每日 10 分鐘檢視 Gate fail 原因
- [ ] 收斂模板欄位（只增不減，保持穩定）
- [ ] 每週回顧一次 EIR，更新防呆規則
- [ ] 一週後再擴大到第二個需求類型（feature 或 bugfix）
