# Workflow State Machine v1 (4 Gates)
版本：v1.0  
狀態：Draft-Executable  
對應章程：`Sub-Agent-Team-Charter-v1.md`  
最後更新：2026-02-13

## 1. 目的
將 `PM -> SA -> SD -> QA` 流程轉成可判定、可回退、可自動化的狀態機，作為後續 CI/CD Gate 控制基準。

## 2. 狀態定義
主狀態：
1. `S0_INTENT_RECEIVED`
2. `S1_PMDRAFT_READY`
3. `S2_SADESIGN_READY`
4. `S3_SDIMPLEMENTED_READY`
5. `S4_QAVERIFIED_PASS`
6. `S5_DONE`

異常狀態：
1. `E_REQ_DEFECT`
2. `E_DESIGN_DEFECT`
3. `E_IMPLEMENTATION_DEFECT`
4. `E_VALIDATION_DEFECT`
5. `E_STOP_THE_LINE`

## 3. Gate 與狀態轉移
### 3.1 PM Gate（G1）
From: `S0_INTENT_RECEIVED`  
To: `S1_PMDRAFT_READY`

通過條件（全部滿足）：
1. PRD 文件存在且命名正確：`PRD-[Project]-[Date]-vX.md`
2. 每個功能點至少一條 `Given/When/Then` AC
3. `scope_in` 與 `scope_out` 皆非空
4. `nfr` 至少一條可量化指標
5. `open_questions[]` 為空

失敗處理：
1. 進入 `E_REQ_DEFECT`
2. 產生 `EIR-[Project]-[Date]-REQ_DEFECT-vX.md`
3. 回退至 `S0_INTENT_RECEIVED`

### 3.2 SA Gate（G2）
From: `S1_PMDRAFT_READY`  
To: `S2_SADESIGN_READY`

通過條件（全部滿足）：
1. DSD 文件存在且命名正確：`DSD-[Project]-[Date]-vX.md`
2. PRD 功能點覆蓋率 100%（有 mapping）
3. `api_contract`、`data_schema` 無 TBD/空欄
4. `error_code_map` 存在
5. `consistency_strategy` 與 `security_controls[]` 存在

失敗處理：
1. 進入 `E_DESIGN_DEFECT`
2. 產生 `EIR-[Project]-[Date]-DESIGN_DEFECT-vX.md`
3. 回退至 `S1_PMDRAFT_READY` 或 `S0_INTENT_RECEIVED`（若判定為需求缺陷）

### 3.3 SD Gate（G3）
From: `S2_SADESIGN_READY`  
To: `S3_SDIMPLEMENTED_READY`

通過條件（全部滿足）：
1. PR 已建立，且關聯 `PRD-ID` + `DSD-ID`
2. CI 全綠（lint/type/test/security）
3. 覆蓋率達標：Py>=90%、TS>=80%、C#>=85%
4. 有回歸測試（Bugfix 必需）
5. `traceability_map` 完整（需求到代碼/測試）

失敗處理：
1. 進入 `E_IMPLEMENTATION_DEFECT`
2. 產生 `EIR-[Project]-[Date]-IMPLEMENTATION_DEFECT-vX.md`
3. 回退至 `S2_SADESIGN_READY`（若設計不足可再回 S1）

### 3.4 QA Gate（G4）
From: `S3_SDIMPLEMENTED_READY`  
To: `S4_QAVERIFIED_PASS`

通過條件（全部滿足）：
1. AC 通過率 100%
2. Critical/High 缺陷數 = 0
3. 回歸測試與邊界測試皆通過
4. QA 報告存在且命名正確：`QA-Report-[Project]-[Date]-vX.md`

失敗處理：
1. 進入 `E_VALIDATION_DEFECT` 或對應實際缺陷類型
2. 產生 `EIR-[Project]-[Date]-[Type]-vX.md`
3. 依缺陷類型回退：REQ -> S0、DESIGN -> S1、IMPLEMENTATION -> S2

### 3.5 完成條件
From: `S4_QAVERIFIED_PASS`  
To: `S5_DONE`

條件：
1. 四個 Gate 全部通過
2. 追溯鏈完整：`REQ-ID -> PRD-ID -> DSD-ID -> PR-ID -> QA-REPORT-ID`

## 4. Stop-the-line 規則
任一階段觸發下列條件，直接進入 `E_STOP_THE_LINE`：
1. 安全 Critical 漏洞
2. 需求與實作目標衝突
3. 追溯鏈中斷（requirement_id 缺失）

處置：
1. 暫停 merge/release
2. 指派責任 agent 24 小時內提交 EIR 與修復計畫
3. Orchestrator（現階段由流程 owner 代行）確認後恢復流程

## 5. 自動化映射（對應 GitHub 工作流）
### 5.1 可機器檢查（Hard Gate）
1. 文件命名規則檢查（PRD/DSD/QA/EIR）
2. PR 關聯 ID 檢查（regex）
3. CI 結果檢查（jobs 全成功）
4. 覆蓋率閾值檢查
5. 安全掃描結果閾值檢查（Critical=0）

### 5.2 人工判定（Soft Gate）
1. AC 品質（是否可測）
2. 設計合理性（架構權衡）
3. 邊界案例完整性

策略：
1. Hard Gate 未過：直接阻擋
2. Soft Gate 未過：需 reviewer/QA 明確回覆與回退分類

## 6. Gate 事件格式（標準化）
每次 Gate 判定必須產生一筆事件：

```yaml
gate_event:
  gate_id: G1|G2|G3|G4
  requirement_id: REQ-XXX
  design_id: DSD-XXX
  pr_id: PR-XXX
  status: PASS|FAIL|STOP
  defect_type: NONE|REQ_DEFECT|DESIGN_DEFECT|IMPLEMENTATION_DEFECT|VALIDATION_DEFECT
  reason: "short reason"
  decided_by: PM|SA|SD|QA
  timestamp: "2026-02-13T00:00:00Z"
```

## 7. 回退矩陣
| 失敗類型 | 來源 Gate | 必回退狀態 | 必產物 |
|---|---|---|---|
| `REQ_DEFECT` | G1/G2/G4 | `S0_INTENT_RECEIVED` | EIR + PRD version bump |
| `DESIGN_DEFECT` | G2/G3/G4 | `S1_PMDRAFT_READY` | EIR + DSD version bump |
| `IMPLEMENTATION_DEFECT` | G3/G4 | `S2_SADESIGN_READY` | EIR + 修復 PR |
| `VALIDATION_DEFECT` | G4 | `S3_SDIMPLEMENTED_READY` | EIR + QA 測試策略修訂 |

## 8. Step 2 驗收（Definition of Done）
1. 四個 Gate 皆有明確進入條件/放行條件/失敗去向
2. 每一種失敗類型都可唯一回退到上游狀態
3. 至少 5 個 Hard Gate 可直接由 CI 自動判定
4. 與 Charter v1 無衝突，且能作為 Step 3（Automation Wiring）輸入
