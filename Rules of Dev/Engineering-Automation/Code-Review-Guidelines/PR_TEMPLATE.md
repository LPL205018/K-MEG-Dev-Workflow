# Pull Request 模板

> 此文件應放在項目根目錄 `.github/pull_request_template.md`
> 所有新 PR 都會自動使用此模板

---

## 描述 (Required)

用簡潔的語言描述此 PR 的主要改動。回答以下問題：

- **這是什麼改動？**
- **為什麼要做這個改動？**
- **有什麼影響？**

### 示例

```markdown
## 描述

修復用戶登錄頁面在行動設備上的響應式設計問題。

用戶在手機上訪問時，登錄表單被截斷，導致提交按鈕不可見。
此改動調整了 CSS media queries，確保小屏幕設備的可用性。

相關的設計在 Figma 上已批准。
```

---

## 特別關注 (Optional)

如果有特別需要審查者關注的地方，請列出。

### 示例

```markdown
## 特別關注

- 🔒 **安全性**: 此改動改變了密碼驗證邏輯，請特別檢查邊界情況
- ⚠️ **破壞性變更**: API 響應格式從 `{ data: [...] }` 改為 `[...]`，可能影響客端
- 🐛 **複雜修復**: 使用了新的並發策略，用了分佈式鎖防止競爭條件
```

---

## 改動類型 (Required)

選擇適用的類型（可多選）：

- [ ] 🐛 缺陷修復 (Bug Fix)
- [ ] ✨ 新功能 (Feature)
- [ ] 🔄 重構 (Refactoring)
- [ ] 📚 文檔 (Documentation)
- [ ] 🎨 樣式 (Styling)
- [ ] ⚡ 性能 (Performance)
- [ ] 🔒 安全 (Security)
- [ ] ♻️ 依賴更新 (Dependencies)

---

## 測試覆蓋 (Required for Strict Mode)

確認測試方面的完成情況：

- [ ] ✅ 添加了對應的單元測試
- [ ] ✅ 測試涵蓋了快樂路徑 (Happy Path)
- [ ] ✅ 測試涵蓋了邊界情況 (Edge Cases)
- [ ] ✅ 如果修復 Bug，添加了迴歸測試 (Regression Test)
- [ ] ✅ 所有測試本地運行通過

### 測試證明（建議附加）

```markdown
### 本地測試結果

```bash
$ pytest tests/auth/test_login.py -v

test_login_with_valid_credentials PASSED
test_login_with_invalid_email PASSED
test_login_with_locked_account PASSED
test_login_rate_limiting PASSED

====== 4 passed in 0.23s ======
Coverage: 92%
```
```

---

## 相關 Issue (Required)

關聯此 PR 解決的 Issue。使用以下格式：

- `Closes #123` - 完全解決
- `Fixes #456` - 修復此問題
- `Related to #789` - 相關但未完全解決

### 示例

```markdown
## 相關 Issue

Closes #542 (修復登錄響應式問題)
Related to #500 (改進登錄用戶體驗的一部分)
```

---

## 文檔更新 (If Applicable)

如果此改動需要更新文檔，請列出：

- [ ] README.md 已更新
- [ ] API 文檔 (Swagger/OpenAPI) 已更新
- [ ] 設置指南已更新
- [ ] 變更日誌 (CHANGELOG.md) 已更新
- [ ] 數據庫 Schema 文檔已更新

---

## 截圖或演示 (If Applicable)

對於 UI 改動，請提供：

- 改動前的截圖
- 改動後的截圖
- 或提供 GIF 動畫演示

### 示例

```markdown
## UI 改動預覽

### 改動前
![Before Login Page] (before.png)

### 改動後
![After Login Page] (after.png)

### 響應式測試（iPhone 12）
[Video Demo] (demo.mp4)
```

---

## 協力廠商依賴 (If Applicable)

如果添加了新的依賴，請列出：

```markdown
## 新增依賴

| 套件 | 版本 | 原因 | License |
|------|------|------|---------|
| `requests` | 2.31.0 | HTTP 客端庫 | Apache 2.0 |
| `pydantic` | 2.5.0 | 數據驗證 | MIT |
```

### 安全檢查

- [ ] 所有新依賴已通過安全掃描 (無已知漏洞)
- [ ] License 與項目兼容
- [ ] 依賴不會引入重複包

---

## 推送清單 (Before Submitting)

在提交 PR 前，確保完成以下檢查：

一般檢查：
- [ ] 代碼遵循項目的編碼規範
- [ ] 本地所有檢查都通過（使用 `make check` 或類似命令）
- [ ] 提交消息遵循 [Conventional Commits](https://www.conventionalcommits.org/)
- [ ] 相關文檔已更新
- [ ] 沒有生成不需要的文件（日誌、臨時文件）

Python 特定：
- [ ] ✅ `black --check .` 通過
- [ ] ✅ `ruff check .` 無 error
- [ ] ✅ `pytest --cov=app tests/` 達到 90% 覆蓋
- [ ] ✅ `pyright src/ --warnings` 無 error

TypeScript 特定：
- [ ] ✅ `npm run format:check` 通過
- [ ] ✅ `npm run lint` 無 error
- [ ] ✅ `npm run type-check` 無 error
- [ ] ✅ `npm run test -- --coverage` 達到 80% 覆蓋

C# 特定：
- [ ] ✅ `dotnet format --verify-no-changes` 通過
- [ ] ✅ `dotnet build /p:TreatWarningsAsErrors=true` 通過
- [ ] ✅ `dotnet test /p:CollectCoverage=true` 達到 85% 覆蓋

分支管理：
- [ ] 此分支是最新的（已從 main/develop 更新）
- [ ] 沒有未提交的更改
- [ ] 分支名遵循命名規則 (`feature/*`, `fix/*` 等)

---

## 審查者注意事項

### 此 PR 需要特別關注：

- [ ] 🔒 安全相關改動（密碼、Token、數據隱私）
- [ ] 💾 數據庫遷移（需要確認向後兼容性）
- [ ] ⚡ 性能關鍵代碼（確認沒有引入新瓶頸）
- [ ] 🚀 發佈流程改動（可能影響部署）

---

## 額外備註 (Optional)

有其他想告訴審查者的嗎？

```markdown
## 備註

- 此改動依賴於 #500，請在該 PR merged 後再 review
- 需要 DevOps 配置新的環境變數 `NEW_API_KEY`
- 此改動與團隊會議中的決定一致
```

---

## 合併後的清理

成功合併後，請確認：

- [ ] 分支已刪除
- [ ] 相關 Issue 已關閉
- [ ] Slack/Teams 已通知相關同事（如適用）
- [ ] 生產發佈（如需要）

---

## 常見 PR 模式

### 只修改文檔

```markdown
## 描述

更新 README 中的安裝步驟，改正了過時的依賴版本。

## 類型

- [x] 📚 文檔

## 相關 Issue

Closes #123
```

### 簡單 Bug 修復

```markdown
## 描述

修復登錄頁面在 Safari 上的樣式問題。
輸入框邊框在焦點時未正確顯示。

## 類型

- [x] 🐛 缺陷修復

## 相關 Issue

Closes #456

## 測試

- [x] 在 Safari 15 上驗證
- [x] 在 Chrome 最新版上驗證
```

### 大型功能

```markdown
## 描述

實現用戶兩步驗證 (2FA) 功能。

...詳細描述...

## 類型

- [x] ✨ 新功能

## 測試

- [x] 單元測試 92% 覆蓋
- [x] 集成測試通過
- [x] 在 Staging 環境手動驗證

## 文檔

- [x] README 已更新設置步驟
- [x] API 文檔已更新
- [x] 用戶指南已更新

## 相關 Issue

Closes #789, #790

## 特別關注

- 安全相關改動，請仔細審查
- 添加了新的環境變數，需 DevOps 配置
```

---

## 提示

- 💡 清晰的 PR 描述可以大幅加快審查過程
- 💡 添加截圖或視頻可以快速傳達信息
- 💡 關聯相關 Issue 幫助追蹤進度
- 💡 測試证明增加審查者信心

---

**更新於：2026-02-13**
