# Antigravity 完整作業流指引手冊

版本：v1.0  
適用：Antigravity Agent（Gemini / Claude）  
目標：讓 Agent 自動完成 K-MEG Dev Workflow 的平台側設定與驗收

---

## 1. 目標成果（Definition of Done）

Agent 完成後，必須同時滿足：

1. Antigravity 流程已建立 `N0 -> N9` 節點並正確連線。
2. H1/H2 人工審批已啟用且必填欄位完整。
3. GitHub 事件可回寫到 `N6_CI_AND_GATEKEEPER_CHECK`。
4. 失敗可分類為四種 defect，並自動導向 `N9_EIR_AND_ROLLBACK`。
5. 可用一個測試需求完成從 INTAKE 到 RELEASE_READY 的演練。

---

## 2. 前置條件

在開始前，確認：

1. Repo：`https://github.com/LPL205018/K-MEG-Dev-Workflow`
2. 分支：`main`、`develop` 已存在
3. Branch protection 已啟用（含 G1~G4 + CI Result Summary）
4. GitHub token 已可供 Antigravity Integration 使用
5. 參考文件：
   - `Rules of Dev/Antigravity-Workflow-Node-Config-v1.md`
   - `Rules of Dev/Workflow-Automation-Wiring-v1.md`
   - `docs/GITHUB_BRANCH_PROTECTION_SETUP.md`

---

## 3. 給 Antigravity Agent 的系統指令（可直接貼）

```text
你是 Workflow Setup Agent。你的任務是把 K-MEG Dev 的流程規範落地到 Antigravity。
執行原則：
1) 嚴格依照 Rules of Dev/Antigravity-Workflow-Node-Config-v1.md 建立 N0~N9。
2) 禁止跳過 H1/H2，任何必填欄位缺失都視為阻擋。
3) 所有 GitHub Gate/CI 結果必須映射到 N6，失敗導向 N9。
4) N9 必須依 defect_type 回退到正確節點：
   REQ_DEFECT->N1, DESIGN_DEFECT->N3, IMPLEMENTATION_DEFECT->N5, VALIDATION_DEFECT->N7。
5) 產出最終設定報告，列出每個節點、觸發條件、輸入/輸出契約、回退路由。
```

---

## 4. 執行步驟（Agent 任務清單）

### Step A：建立流程骨架

1. 新增 workflow：`K-MEG-Dev-Workflow-v1`
2. 建立節點：
   - `N0_INTAKE`
   - `N1_PM_GENERATE_PRD`
   - `N2_H1_HUMAN_APPROVAL_PM_TO_SA`
   - `N3_SA_GENERATE_DSD`
   - `N4_H2_HUMAN_APPROVAL_SA_TO_SD`
   - `N5_SD_IMPLEMENT_AND_PR`
   - `N6_CI_AND_GATEKEEPER_CHECK`
   - `N7_QA_VERIFY_AND_REPORT`
   - `N8_RELEASE_READY`
   - `N9_EIR_AND_ROLLBACK`
3. 連線：
   - 主線：`N0 -> N1 -> N2 -> N3 -> N4 -> N5 -> N6 -> N7 -> N8`
   - 失敗線：`N2/N4/N6/N7 -> N9`

### Step B：設定節點契約（I/O）

依 `Rules of Dev/Antigravity-Workflow-Node-Config-v1.md` 寫入每個節點輸入輸出欄位。  
至少包含：

1. 追溯鏈欄位：`REQ-ID`, `PRD-ID`, `DSD-ID`, `PR-ID`, `QA-REPORT-ID`
2. 狀態欄位：`current_node`, `status`, `failed_gate`, `defect_type`
3. 針對 `N6`：`G1_STATUS`, `G2_STATUS`, `G3_STATUS`, `G4_STATUS`, `CI_STATUS`

### Step C：設定 H1 / H2（HITL）

1. 在 `N2` 建立 H1 表單（皆必填）：
   - `方向符合需求?`
   - `AC 是否可測?`
   - `Scope In/Out 是否清楚?`
2. 規則：任一 `No` -> 路由到 `N9` 並標記 `REQ_DEFECT`
3. 在 `N4` 建立 H2 表單（皆必填）：
   - `架構方向正確?`
   - `API/Data 契約可實作?`
   - `風險策略可執行?`
4. 規則：任一 `No` -> 路由到 `N9` 並標記 `DESIGN_DEFECT`

### Step D：綁定 GitHub 事件

1. 綁定 GitHub repo：`LPL205018/K-MEG-Dev-Workflow`
2. 監聽事件：
   - `pull_request`
   - `check_run`
   - `workflow_run`
3. 事件映射到 `N6`：
   - `G1-PM` -> `G1_STATUS`
   - `G2-SA` -> `G2_STATUS`
   - `G3-SD` -> `G3_STATUS`
   - `G4-QA` -> `G4_STATUS`
   - `CI Result Summary` -> `CI_STATUS`
4. 判斷規則：
   - 任一 FAIL -> `N9`
   - 全 PASS -> `N7`

### Step E：設定 N9 回退引擎

`N9_EIR_AND_ROLLBACK` 必做：

1. 產出 `EIR-ID`
2. 填入 `defect_type`, `failed_node`, `failed_gate`
3. 路由：
   - `REQ_DEFECT` -> `N1_PM_GENERATE_PRD`
   - `DESIGN_DEFECT` -> `N3_SA_GENERATE_DSD`
   - `IMPLEMENTATION_DEFECT` -> `N5_SD_IMPLEMENT_AND_PR`
   - `VALIDATION_DEFECT` -> `N7_QA_VERIFY_AND_REPORT`

### Step F：啟用 STOP_THE_LINE

若發生以下任一條件，直接停線並通知維運/負責人：

1. Security critical
2. 追溯鏈斷裂
3. H1/H2 重複拒絕導致需求方向衝突

---

## 5. Agent 驗收腳本（可直接貼給 Gemini/Claude）

```text
請對 workflow「K-MEG-Dev-Workflow-v1」做驗收：
1) 驗證 N0~N9 節點存在且主線/失敗線正確。
2) 驗證 N2/N4 具備必填 HITL 表單與 No->N9 規則。
3) 驗證 N6 已綁定 G1/G2/G3/G4/CI 五種檢查結果。
4) 驗證 N9 可輸出 EIR-ID 並按 defect_type 回退。
5) 用測試案例 REQ-DEMO-001 跑一次 dry-run，輸出每節點輸入輸出紀錄。
若任一項不通過，請列出 FAIL 點、根因、修正建議與重測結果。
```

---

## 6. 建議測試案例（REQ-DEMO-001）

1. 輸入需求到 N0（簡單 CRUD 功能即可）
2. H1/H2 都選 Yes，確認可到 N8
3. 模擬 G2 失敗，確認進 N9 並回退 N3
4. 模擬 QA Fail，確認進 N9 並回退 N7
5. 檢查追溯鏈是否完整：
   - `REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`

---

## 7. 交付物清單（Agent 最終回報格式）

Agent 最終回報必須包含：

1. Workflow 名稱與版本
2. 節點拓樸截圖/匯出
3. 每節點 I/O 契約摘要
4. H1/H2 表單欄位與阻擋條件
5. GitHub 事件映射表
6. N9 回退矩陣
7. Dry-run 結果（成功/失敗路徑各至少一次）

---

## 8. 常見失敗與修正

1. 問題：看不到 G1~G4 checks  
   修正：確認 workflow 在 `.github/workflows/` 並且至少跑過一次。

2. 問題：PR 可直接 merge  
   修正：重查 `main/develop` branch protection 是否啟用 required checks。

3. 問題：N6 無法判定 gate 結果  
   修正：檢查 check name 是否與 GitHub 實際 job name 一致。

4. 問題：回退路由錯誤  
   修正：校正 `defect_type -> target_node` 對照表。

