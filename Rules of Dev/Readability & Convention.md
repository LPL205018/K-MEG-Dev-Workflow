# 開發守則：第二部分 - 代碼可讀性與慣例 (Readability & Convention)
# 版本：v1.0 | 狀態：MANDATORY (強制執行)

## 1. 意圖導向命名 (Intent-Based Naming)
* **拒絕抽象命名：** 禁止使用 `data`, `info`, `item`, `process`, `handle` 等模糊詞彙。
    * *錯誤：* `handleData(d)`
    * *正確：* `validateUserPayload(payload)`
* **語義化布林值：** 所有布林變數與函數必須以動詞前綴開頭（如 `is`, `has`, `should`, `can`）。
    * *範例：* `isRetryAllowed`, `hasActiveSubscription`
* **常數語義化：** 嚴禁在代碼中出現「魔術數字 (Magic Numbers)」或「硬編碼字串」。所有具備業務意義的數值必須定義為具名常數。
    * *錯誤：* `if (status === 2) ...`
    * *正確：* `if (status === ORDER_STATUS.SHIPPED) ...`

## 2. 代碼自解釋結構 (Self-Documenting Structure)
* **函數原子化：** 每個函數應只做一件「標題描述的事」。如果需要寫長篇註釋來解釋一段代碼在做什麼，請直接將該段代碼重構為一個具名函數。
* **減少認知負擔：** * **提前回傳 (Early Return)：** 優先處理異常邊界條件並回傳，避免深層的 `if-else` 巢狀結構。
    * **邏輯對稱性：** 相似的邏輯在不同模組中應具備相似的結構與命名模式。

## 3. 註釋規範 (Commenting Axioms)
* **解釋「為什麼」而非「是什麼」：** 註釋應描述業務決策背景、特殊的技術權衡或非直觀的邏輯原因。
    * *禁止：* `// 增加計數 (count++)`
    * *強制：* `// 為了平衡並行競爭，此處採用原子操作以防止計數丟失`
* **TODO & FIXME 標註：** 暫時性的權宜代碼必須標註 `// TODO(姓名): 預期重構方向`，嚴禁留下無名且無截止日期的技術債。
* **文件化註釋 (JSDoc/Doxygen)：** 所有的公共接口 (Public API) 必須包含參數類型、回傳值描述及潛在的異常說明。

## 4. 格式一致性 (Syntactic Uniformity)
* **機器自動化校準：** 團隊必須統一使用相同的 Linter 配置（如 ESLint, Prettier）。
    * *原則：* 代碼庫中不應存在任何關於「空格或是 Tab」、「分號與否」的爭論，一切交由機器自動格式化。
* **文件組織：** 檔案頂部應依序為：導入語句 (Imports)、常數定義 (Constants)、核心邏輯、輔助私有函數。