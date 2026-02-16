# Vibe Coding Agent: 專業 PRD 生成與規範守則 (Standard PRD Protocol)
# 版本：v1.2 | 狀態：MANDATORY | 目標：消除需求歧義，確保架構對齊

## 1. PRD 核心結構規範 (Required Framework)
Agent 輸出的 PRD 必須包含以下標準章節，缺一不可：

* **1. 專案背景與願景 (Context & Vision):** 描述核心問題 (Problem Statement) 與成功指標 (Success Metrics)。
* **2. 使用者故事與場景 (User Stories & Scenarios):** 採用「身為 [角色], 我想要 [動作], 以便 [價值]」格式。
* **3. 核心功能規格 (Functional Specifications):** 詳細的功能清單、輸入輸出定義、操作流程。
* **4. 非功能性需求 (Non-Functional Requirements):** 包含性能、安全性、併發處理能力。
* **5. 數據模型與 API 定義 (Data & Interface):** 初步的 Schema 設計與核心介面定義。
* **6. 邊界條件與例外處理 (Edges & Constraints):** 錯誤處理流程與系統限制。

---

## 2. 需求描述精密化原則 (Precision Requirements)

### 2.1 消除模糊詞彙
* **禁止使用：** 「優化」、「快速」、「穩定」、「簡潔」、「高性能」等主觀描述。
* **強制使用：** 定量指標。
    * *錯誤：* 系統響應要快。
    * *正確：* 95% 的 API 請求延遲應低於 200ms。

### 2.2 邏輯完整性 (Logical Completeness)
* **State Machine 思維：** 涉及狀態轉換的功能（如訂單、審核），必須定義完整的狀態機（Status: Pending, Approved, Rejected）。
* **CRUD 完整性：** 任何實體（Entity）的定義必須考慮增刪改查的權限與限制。

---

## 3. 架構對齊與技術約束 (Architectural Alignment)

* **L0 公理對齊：** 需求必須符合物理邏輯。若涉及高頻交易，必須描述並行衝突 (Concurrency) 的處理策略。
* **技術棧預設 (Stack Constraints):** PRD 應明述技術邊界（例如：React + FastAPI + PostgreSQL），確保代碼生成時的環境一致性。
* **擴展性預留：** PRD 必須標明哪些部分是未來可能變動的，以便 Agent 在設計架構時預留 Interface。

---

## 4. 驗證與驗收標準 (Acceptance Criteria, AC)

* **每一項功能必須配對至少一條 AC：**
    * *格式：* Given [初始狀態], When [操作發生], Then [預期結果]。
* **定義「完成」的定義 (DoD):** 例如：代碼經過單元測試、文檔已更新、符合安全性掃描。

---

## 5. Agent 執行指令 (Execution Instructions for Agent)

當與使用者互動生成 PRD 時，Agent 必須遵循以下遞歸邏輯：

1.  **意圖蒸餾 (Distillation):** 若使用者輸入過於簡略，必須啟動「蘇格拉底式探詢」，主動詢問邊界條件。
2.  **虛擬原假設 (Hypothesis Testing):** 在輸出 PRD 前，內部模擬潛在的技術衝突點並在 PRD 中提出解決方案。
3.  **Fact-Check:** 確保 PRD 中的數據格式、合規性要求符合現實 L1 (法律/法規) 層級。

---

## 6. PRD 命名與文件標準 (Documentation Rules)

* **文件命名格式：** `PRD-[專案名稱]-[日期]-[版本號].md`
* **版本記錄：** 檔案頂部必須包含版本更新日誌 (Changelog)，記錄每一次需求變更的原因。