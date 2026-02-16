# 系統開發最高行為準則：命名與規約篇 (Naming & Logic Rules)
# 版本：v1.0 | 狀態：MANDATORY (系統級別指令)

## 1. 命名空間與標示符原則 (The Naming Axioms)

### 1.1 語義化與資訊熵
* **意圖大於形式：** 命名必須回答「為什麼存在」而非「是什麼類型」。
    * *禁止：* `var list = []`, `function handle(data)`
    * *強制：* `var activeUserSubscriptions = []`, `function validateTransactionPayload(payload)`
* **上下文感知縮寫：** 除非是極其通用的縮寫（如 `id`, `err`, `i`），否則嚴禁縮寫。
    * *錯誤：* `authUsr`
    * *正確：* `authenticateUser`

### 1.2 標示符格式規範 (Casing Standards)
| 實體類型 | 格式 (Casing) | 範例 |
| :--- | :--- | :--- |
| **類別 / 介面 (Class/Interface)** | PascalCase | `UserManager`, `IDataRepository` |
| **變數 / 函數 (Variable/Function)** | camelCase | `isUserLogged`, `calculateTax()` |
| **常量 (Constants/Enums)** | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `STATUS_PENDING` |
| **文件名稱 (File Naming)** | kebab-case (建議) | `user-profile.controller.ts` |
| **私有屬性 (Private Property)** | _underscorePrefix | `_internalCache` |

### 1.3 布林值與謂詞命名
* **必須具備詢問語義：** 所有布林變數必須以 `is`, `has`, `can`, `should`, `did` 為前綴。
    * *範例：* `hasWritePermission`, `isProcessingComplete`

---

## 2. 代碼編寫硬性規則 (Hard Coding Rules)

### 2.1 函數原子化 (Atomic Functions)
* **單一職責：** 每個函數只處理一個邏輯層級的事務。若函數需要註釋來區分「第一步、第二步」，則應拆分為多個子函數。
* **純粹性：** 盡可能減少函數的副作用（Side Effects）。優先使用傳入參數與回傳值，而非修改全局狀態或外部變數。

### 2.2 控制流優化 (Control Flow Rigor)
* **衛句模式 (Guard Clauses)：** 優先處理異常邊界並提前回傳（Early Return），嚴禁深層 `if-else` 嵌套。
    ```javascript
    // 正確示範
    if (!user) return Error.NotFound();
    if (!hasPermission) return Error.Forbidden();
    return executeAction();
    ```
* **拒絕「魔術字串/數字」：** 嚴禁在邏輯判斷中直接出現數值或字串。所有分支判斷必須引用 `Enum` 或 `Constant`。

### 2.3 集合與資源處理
* **空值防禦：** 回傳集合（List/Array）時，若無數據應回傳「空集合」而非 `null`，以減少調用端的判空壓力。
* **資源自動回收：** 所有的檔案串流 (Streams)、數據庫連接 (Connections) 必須使用 `try-with-resources` 或 `finally` 確保關閉，防止內存洩漏。

---

## 3. 註釋與文檔哲學 (Documentation Axioms)

* **代碼自解釋優先：** 如果代碼需要註釋才能讀懂，優先重構代碼結構與命名。
* **註釋「動機」而非「動作」：** * *不要寫：* `// 將 i 加 1`
    * *必須寫：* `// 此處需補償索引偏差，以對齊外部 API 的起始偏移量`
* **過期註釋即毒素：** 更新代碼時，必須同步更新註釋。與代碼不符的註釋比沒有註釋更危險。

---

## 4. 錯誤處理架構 (Unified Error Handling)

* **禁止靜默失效：** 嚴禁捕捉異常後不進行任何處理（Empty Catch）。
* **強類型異常：** 拋出的錯誤應包含具備語義的錯誤代碼（Error Code），而不僅僅是字串訊息。
* **日誌可追溯性：** 報錯時必須攜帶足夠的上下文資訊（如：用戶 ID、當前操作參數），但需過濾掉敏感個人隱私。