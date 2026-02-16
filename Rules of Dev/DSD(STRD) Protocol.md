# Vibe Coding Agent: 專業 SA 系統設計與技術規格守則 (DSD/STRD Protocol)
# 版本：v1.2 | 狀態：MANDATORY | 目標：定義系統物理邊界與技術實現路徑

## 1. 系統架構定義 (Architectural Blueprint)
SA 必須明確定義系統的拓撲結構與分層模式：
* **架構模式 (Pattern):** 明確採用何種架構（如：Clean Architecture, Hexagonal, Microservices）。
* **模組劃分 (Modularity):** 以邊界上下文 (Bounded Context) 定義功能模組，並說明各模組間的依賴關係。
* **技術棧鎖定 (Tech Stack):** 列出具體的語言版本、框架、中間件（Redis, RabbitMQ）及外部服務。

## 2. 數據架構與實體設計 (Data Architecture)
禁止僅描述業務實體，必須達到可直接實作的精確度：
* **ER Diagram / Schema:** 提供詳細的資料庫欄位定義（Field, Type, Nullable, Index, Constraint）。
* **數據生命週期:** 描述數據從創建、狀態轉移（State Machine）到歸檔/刪除的完整過程。
* **一致性策略:** 針對分佈式場景，明確定義「強一致性」或「最終一致性」的處理手段。



## 3. 介面與協議規範 (Interface & API Design)
* **API 契約:** 採用 OpenAPI/Swagger 標準。定義 Endpoint、Method、Request/Response Schema。
* **錯誤代碼表:** 統一定義全局錯誤代碼及其對應的業務場景，確保前後端對齊。
* **通訊協議:** 明確定義傳輸格式（REST/JSON, GraphQL, gRPC）與認證方式（JWT, OAuth2）。

## 4. 關鍵技術邏輯與演算法 (Core Logic & Sequence)
針對複雜業務，必須提供邏輯視覺化描述：
* **時序圖 (Sequence Diagram):** 描述多組件間的交互過程，特別是跨服務調用。
* **偽代碼/流程圖:** 針對核心演算法或複雜規則（如：計費邏輯、權限判斷），提供不依賴特定語言的邏輯實現步驟。



## 5. 非功能性技術需求 (Technical QoS)
* **併發與鎖策略:** 定義如何處理競爭條件（如：資料庫行鎖、Redis 分佈式鎖）。
* **緩存策略:** 定義緩存失效機制（TTL）、更新策略（Cache-Aside, Write-Through）。
* **效能指標:** 定義單次操作的 Time Complexity ($O(n)$) 與預計的資源消耗。

## 6. 安全與防禦架構 (Security Architecture)
* **身份與權限:** 明確定義 RBAC (Role-Based Access Control) 或 ABAC 實作細節。
* **數據脫敏規則:** 定義哪些欄位需在傳輸或日誌中遮罩。
* **防護機制:** 針對 API 定義限流 (Rate Limiting)、重試 (Retry) 與熔斷 (Circuit Breaker) 的閾值。

---

## 7. SA 執行審核清單 (SA Execution Checklist)
Agent 在輸出 DSD/STRD 前必須自檢：
1.  **可實作性:** 該文件是否能讓開發 Agent 在不詢問任何問題的情況下寫出代碼？
2.  **無歧義性:** 是否存在「視情況而定」的模糊描述？（必須予以明確化）
3.  **追溯性:** 技術設計是否 100% 覆蓋了 PRD 中的功能點？