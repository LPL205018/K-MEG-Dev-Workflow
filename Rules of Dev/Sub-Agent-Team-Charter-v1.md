# Sub-Agent Team Charter v1 (4-Step)
版本：v1.0  
狀態：Draft-Executable  
流程範圍：`PM -> SA -> SD -> QA`  
最後更新：2026-02-13

## 1. 目的與邊界
本章程定義四個子代理在單一路徑交付中的責任、交接契約與回退規則，確保需求可追溯、交付可驗證、失敗可迭代。

適用範圍：
- 新功能開發
- 既有功能重構
- 缺陷修復（Bugfix）

不適用範圍：
- 緊急生產事故（另走 Hotfix Runbook）
- 純營運性變更（不涉及產品需求與程式碼）

## 2. 團隊結構（4 Agents）
### 2.1 PM Agent（需求蒸餾）
職責：
- 將原始需求轉為可實作 PRD（含功能、非功能、邊界、驗收）
- 消除歧義，不允許模糊需求流入 SA

輸入：
- 原始需求（Issue/訪談/會議結論）
- 既有產品限制與目標

輸出：
- `PRD-[Project]-[Date]-vX.md`
- `AC Matrix`（Given/When/Then）
- `Scope In/Out`

Exit Criteria（PM Gate）：
- 每個功能點至少 1 條 AC
- 至少包含一條非功能需求（效能/安全/可靠性）
- 邊界明確（做/不做）

### 2.2 SA Agent（系統架構）
職責：
- 將 PRD 轉成可直接開發的 DSD/STRD
- 定義資料模型、API 契約、核心流程、錯誤碼與風險策略

輸入：
- PM 輸出的 PRD + AC Matrix

輸出：
- `DSD-[Project]-[Date]-vX.md`
- API Schema（OpenAPI/contract）
- Data Schema（欄位、約束、索引）

Exit Criteria（SA Gate）：
- PRD 功能點覆蓋率 100%
- API/資料模型無 TBD 欄位
- 風險與一致性策略可執行

### 2.3 SD Agent（Software Development）
職責：
- 依 DSD 實作程式碼、測試與必要文件更新
- 僅在 DSD 覆蓋範圍內開發，禁止自行擴規

輸入：
- SA 輸出的 DSD + API/Data 契約

輸出：
- 程式碼變更（feature branch）
- 單元測試與回歸測試
- PR（含變更描述、測試證據、關聯 ID）

Exit Criteria（SD Gate）：
- CI 必過（lint/type/test/security）
- 覆蓋率達門檻：Py 90% / TS 80% / C# 85%
- 所有變更可追溯到 PRD/DSD ID

### 2.4 QA Agent（驗證與證偽）
職責：
- 驗證需求覆蓋、邊界案例與安全風險
- 拒絕無法證明正確性的交付

輸入：
- SD 的 PR、測試報告、覆蓋率與掃描結果

輸出：
- `QA-Report-[Project]-[Date]-vX.md`
- 缺陷清單（含嚴重度）
- EIR（Error Iteration Report，若失敗）

Exit Criteria（QA Gate）：
- AC 通過率 100%
- Critical/High 缺陷為 0
- 回歸測試通過，且無 blocker

## 3. 流程定義（唯一主路徑）
1. PM 完成 PRD 並通過 PM Gate  
2. SA 完成 DSD 並通過 SA Gate  
3. SD 產出程式碼與測試並通過 SD Gate  
4. QA 完成驗證並通過 QA Gate  
5. Gate 失敗時，僅允許按回退規則返回上游

## 4. RACI Matrix（四步版）
| 活動 | PM | SA | SD | QA |
|---|---|---|---|---|
| 需求蒸餾與 AC 定義 | R/A | C | I | C |
| 架構與契約設計 | C | R/A | C | C |
| 程式開發與測試撰寫 | I | C | R/A | C |
| 測試策略與驗證執行 | I | C | C | R/A |
| Gate 放行判定 | A | A | A | A |
| 失敗回退判定（依規則） | A | A(設計缺陷) | A(實作缺陷) | A(驗證缺陷) |

註記：
- `R` Responsible
- `A` Accountable
- `C` Consulted
- `I` Informed

## 5. 交接契約（Handoff Contract）
### 5.1 PM -> SA
必填欄位：
- `requirement_id`
- `problem_statement`
- `user_story[]`
- `acceptance_criteria[]`
- `scope_in[]`
- `scope_out[]`
- `nfr[]`
- `assumptions[]`
- `open_questions[]`（若不為空，禁止進 SA）

拒收條件：
- AC 缺失或不可測
- 使用主觀詞（快速、穩定、優化）但無量化

### 5.2 SA -> SD
必填欄位：
- `design_id`
- `architecture_pattern`
- `module_boundaries[]`
- `data_schema`
- `api_contract`
- `error_code_map`
- `sequence_or_pseudocode`
- `consistency_strategy`
- `security_controls[]`

拒收條件：
- API/Data 仍有 TBD
- PRD 功能點對應不完整

### 5.3 SD -> QA
必填欄位：
- `pr_link`
- `traceability_map`（PRD-ID -> DSD-ID -> Code/Tests）
- `test_report`
- `coverage_report`
- `security_scan_report`
- `known_limitations[]`

拒收條件：
- 無回歸測試
- 覆蓋率低於門檻
- 無法追溯到需求

## 6. 升級與回退政策（Escalation & Rollback）
### 6.1 失敗分類
- `REQ_DEFECT`：需求缺陷（回退 PM）
- `DESIGN_DEFECT`：設計缺陷（回退 SA）
- `IMPLEMENTATION_DEFECT`：實作缺陷（回退 SD）
- `VALIDATION_DEFECT`：驗證策略缺陷（回退 QA 重新驗證）

### 6.2 回退規則
1. QA 發現與 AC 不符，先判定缺陷類型。  
2. 若屬設計缺陷，禁止 SD 直接打補丁，必須回 SA 更新 DSD。  
3. 若屬需求缺陷，必須回 PM 更新 PRD，並同步版本號。  
4. 每次回退都必須產生 EIR，記錄根因與預防措施。

### 6.3 Stop-the-line 觸發條件
- 發現安全 Critical 漏洞
- 發現需求與實作目標衝突
- 追溯鏈中斷（無法對應 requirement_id）

## 7. 追溯與命名標準
必須維持以下鏈路：
`REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`

命名格式：
- `PRD-[Project]-[Date]-vX.md`
- `DSD-[Project]-[Date]-vX.md`
- `QA-Report-[Project]-[Date]-vX.md`
- `EIR-[Project]-[Date]-[Type]-vX.md`

## 8. 服務等級（SLA）
- PM 回應需求補充：1 個工作日內
- SA 完成設計澄清：1 個工作日內
- SD 完成缺陷修復：依嚴重度（Critical: 24h）
- QA 完成回歸驗證：1 個工作日內

## 9. Step 1 驗收（Definition of Done）
- 四角色責任無重疊衝突，且每步有唯一主責
- 三段交接契約可直接機器檢查（欄位完整性）
- 失敗可唯一分類並回退到單一上游步驟
- 可直接作為 Step 2（Workflow State Machine）輸入
