# Code Review 完整指南與自動化清單

> 幫助團隊成員高效進行代碼審查，同時通過自動化減少人工判斷

---

## 📋 目錄

1. [快速檢查清單](#快速檢查清單)
2. [詳細審查標準](#詳細審查標準)
3. [Commit 規範](#commit-規範)
4. [常見問題與模式](#常見問題與模式)
5. [自動化檢查](#自動化檢查)
6. [最佳實踐](#最佳實踐)

---

## 快速檢查清單

> **用時：5-10 分鐘** | **在提交 PR 前自檢**

### ✅ 代碼品質（自動化已檢查）

- [ ] ✅ Prettier/Black 格式檢查已通過
- [ ] ✅ Lint (ESLint/Ruff/StyleCop) 無 error
- [ ] ✅ 類型檢查 (TSC/Pyright) 100% pass
- [ ] ✅ 單元測試覆蓋率達標（Py 90%, TS 80%, C# 85%）
- [ ] ✅ 所有自動審查 Bot 的評論已解決

### ✅ 代碼邏輯（人工審查重點）

- [ ] 代碼邏輯清晰，審查者無需反覆閱讀
- [ ] 函數名/變數名準確反映其含義（無 `data`, `handle` 等模糊詞）
- [ ] 複雜邏輯有註釋解釋「為什麼」而非「做什麼」
- [ ] 沒有重複代碼（DRY 原則）
- [ ] 錯誤處理完整（無空 catch，有類型化異常）

### ✅ 安全性（Strict Mode 要求）

- [ ] ✅ 沒有硬編碼密鑰、密碼、API Token
- [ ] ✅ 用戶輸入已驗證（白名單格式化）
- [ ] ✅ 資料庫查詢使用參數化（無字串拼接）
- [ ] ✅ 敏感數據未出現在日誌中

### ✅ 測試（Strict Mode 要求）

- [ ] 新增功能配有對應的測試
- [ ] 修復 Bug 後補加測試用例（防止迴歸）
- [ ] 測試涵蓋快樂路徑 + 邊界情況
- [ ] 測試不依賴真實 DB/API（使用 Mock）

### ✅ 文檔更新

- [ ] README.md 或相關文檔已更新
- [ ] 公共 API 有 JSDoc/Doxygen 註釋
- [ ] 如涉及數據結構變更，Schema 文檔已更新
- [ ] Breaking changes 已在 CHANGELOG 記錄

### ✅ PR 本身

- [ ] PR 標題清晰明確（使用 Conventional Commits 格式）
- [ ] PR 描述解釋改動背景 & 為什麼這樣做
- [ ] 相關的 Issue 已關聯 (Closes #123)
- [ ] 沒有 Draft 狀態下的 WIP（Work In Progress）

---

## 詳細審查標準

### 【架構設計層】

#### 單一責任原則 (SRP)

```python
# ❌ 不好：函數做太多事
def process_user_order(user_id, items):
    # 驗證
    if not user_id:
        raise ValueError()
    
    # 計算
    total = sum(item.price for item in items)
    
    # 保存
    db.save_order(user_id, total)
    
    # 發送郵件
    email.send(user_id)
    
    # 記錄日誌
    logger.info(f"Order processed for {user_id}")
    return total

# ✅ 好：職責分離
def validate_user(user_id):
    if not user_id:
        raise ValueError("Invalid user ID")

def calculate_order_total(items):
    return sum(item.price for item in items)

def process_user_order(user_id, items):
    validate_user(user_id)
    total = calculate_order_total(items)
    db.save_order(user_id, total)
    return total

# 發送郵件和日誌由外層控制或事件系統處理
```

#### 依賴注入

```python
# ❌ 不好：硬編碼依賴
class UserService:
    def __init__(self):
        self.db = PostgresDatabase()  # ← 依賴在初始化時創建
        self.email = SMTPEmail()

# ✅ 好：依賴注入
class UserService:
    def __init__(self, db: Database, email: EmailProvider):
        self.db = db
        self.email = email

# 實例化時注入
service = UserService(
    db=PostgresDatabase(),
    email=SMTPEmail()
)
```

#### 介面隔離

```typescript
// ❌ 不好：接口過大
interface Repository {
  create(): Promise<void>;
  read(): Promise<any>;
  update(): Promise<void>;
  delete(): Promise<void>;
  backup(): Promise<void>;
  migrate(): Promise<void>;
  import(): Promise<void>;
  export(): Promise<void>;
  // ... 20+ 個方法
}

// ✅ 好：細粒度接口
interface CRUDRepository<T> {
  create(item: T): Promise<T>;
  read(id: string): Promise<T>;
  update(item: T): Promise<T>;
  delete(id: string): Promise<void>;
}

interface BackupRepository {
  backup(): Promise<void>;
  restore(): Promise<void>;
}

class UserRepository implements CRUDRepository<User>, BackupRepository {
  // ...
}
```

---

### 【命名規範層】

#### 清晰度檢查

審查清單：

- [ ] 類名：大寫開頭，名詞 (UserManager, OrderProcessor)
- [ ] 函數名：camelCase，動詞+賓語 (validateEmail, calculateTotal)
- [ ] 變數名：camelCase，名詞 (userData, totalPrice)
- [ ] 常量名：UPPER_SNAKE_CASE (MAX_RETRIES, DEFAULT_TIMEOUT)
- [ ] 布林變數：is/has/should/can 開頭 (isActive, hasPermission)

```java
// ❌ 不清楚
int d;  // 什麼是 d?
var handle = () => { /*...*/ };  // handle 什麼?
let info = {};  // 什麼 info?

// ✅ 清楚
int delayInMilliseconds;
var validateUserEmail = () => { /*...*/ };
let userProfileData = {};
```

---

### 【測試層】

#### 測試覆蓋率檢查

```bash
# Python: >= 90%
pytest --cov=src/app tests/ --cov-report=term-missing

# TypeScript: >= 80%
vitest --coverage

# C#: >= 85%
dotnet test /p:CollectCoverage=true
```

#### 測試質量檢查

審查清單：

- [ ] 測試名清晰描述測試內容 (test_validate_email_with_invalid_format)
- [ ] 使用 AAA 模式 (Arrange-Act-Assert)
- [ ] 不依賴外部資源 (Mock 所有 IO)
- [ ] 測試獨立（順序無關）
- [ ] 沒有 sleep() 或硬編碼延遲（使用 time-based 測試）

```python
# ❌ 不好：模糊的名稱 + 不清楚的邏輯
def test_user():
    user = User("john")
    result = validate(user)
    assert result

# ✅ 好：清晰 + AAA 模式
def test_validate_email_returns_true_for_valid_format():
    # Arrange
    valid_email = "john@example.com"
    validator = EmailValidator()
    
    # Act
    result = validator.validate(valid_email)
    
    # Assert
    assert result is True

def test_validate_email_returns_false_for_invalid_format():
    # Arrange
    invalid_emails = [
        "john",
        "john@",
        "@example.com",
        "john @example.com"
    ]
    validator = EmailValidator()
    
    # Act & Assert
    for email in invalid_emails:
        assert validator.validate(email) is False
```

---

### 【安全性層】

**Strict Mode 為安全第一**

#### SQL 注入防護

```python
# ❌ 危險：字符串拼接
user = db.query(f"SELECT * FROM users WHERE id = {user_id}")

# ✅ 安全：參數化查詢
user = db.query("SELECT * FROM users WHERE id = ?", (user_id,))

# 或使用 ORM
user = User.query.filter_by(id=user_id).first()
```

#### XSS 防護（前端）

```html
<!-- ❌ 危險：直接渲染用戶輸入 -->
<div v-html="userContent"></div>

<!-- ✅ 安全：使用 innerText 或 textContent -->
<div>{{ userContent }}</div>

<!-- 或使用轉義庫 -->
<div [innerHTML]="sanitizer.sanitize(userContent)"></div>
```

#### 敏感信息

```python
# ❌ 不要
logger.info(f"User {uid} login with password {password}")
logger.error(f"API Key: {api_key}, Error: {error}")

# ✅ 要
logger.info(f"User {uid} login successfully")
logger.error(f"API call failed: {error}")

# 如果必須日誌 secrets，用掩碼
masked_key = api_key[:8] + "****" + api_key[-4:]
logger.debug(f"Using API Key: {masked_key}")
```

---

### 【文檔與註釋層】

#### 何時使用註釋

```python
# ❌ 不要：解釋「做什麼」（代碼已經清楚）
def calculate_total(items):
    # 將每項的價格相加
    return sum(item.price for item in items)

# ✅ 要：解釋「為什麼」或非直觀的邏輯
def calculate_total_with_discount(items, user_tier):
    """
    計算總價並應用分級折扣。
    
    注意：某些商品（如數位商品）不適用折扣，
    這在 item.is_discountable 中標記。
    """
    subtotal = sum(item.price for item in items if item.is_discountable)
    
    # 分級折扣：銀牌 5%, 金牌 10%, 鑽石 15%
    discount_rates = {
        UserTier.SILVER: 0.05,
        UserTier.GOLD: 0.10,
        UserTier.DIAMOND: 0.15,
    }
    discount_rate = discount_rates.get(user_tier, 0)
    
    return subtotal * (1 - discount_rate)
```

#### JSDoc / Doxygen 標準

```python
def process_payment(order_id: str, amount: float, method: str) -> PaymentResult:
    """
    處理訂單支付。
    
    Args:
        order_id: 訂單 ID (格式: ORD-XXXXXX)
        amount: 支付金額（單位：元），範圍 0.01-999999.99
        method: 支付方式 ('credit_card', 'paypal', 'wechat')
    
    Returns:
        PaymentResult 對象，包含：
        - success (bool): 是否成功
        - transaction_id (str): 交易 ID（成功時存在）
        - error_message (str): 錯誤訊息（失敗時存在）
    
    Raises:
        InvalidAmountError: 金額不在有效範圍內
        UnsupportedPaymentMethodError: 不支援的支付方式
        OrderNotFoundError: 訂單不存在
    
    Example:
        >>> result = process_payment("ORD-123456", 99.99, "credit_card")
        >>> if result.success:
        ...     print(f"Payment successful: {result.transaction_id}")
    """
```

---

## Commit 規範

### Conventional Commits 格式

```
<type>(<scope>): <subject>
<blank line>
<body>
<blank line>
<footer>
```

#### Type（必須）

| Type | 說明 | 例子 |
|------|------|------|
| **feat** | 新功能 | `feat(auth): add OAuth2 support` |
| **fix** | 缺陷修復 | `fix(payment): handle null amount` |
| **refactor** | 代碼重構（無邏輯變更） | `refactor(api): simplify validation logic` |
| **test** | 添加/修改測試 | `test(user): add test for email validation` |
| **chore** | 依賴、構建、發佈 | `chore(deps): upgrade React to 18.2` |
| **docs** | 文檔 | `docs(README): add setup instructions` |
| **style** | 格式化、不影響代碼執行 | `style(formatting): run Prettier` |
| **perf** | 性能優化 | `perf(database): add index on user_id` |

#### Scope（可選但推薦）

```
feat(auth): ...          # auth 模塊
feat(api): ...           # API 層
feat(db): ...            # 數據庫層
feat(ui): ...            # UI 組件
```

#### Subject（必須）

- 使用命令式語氣（"add" 而非 "added" 或 "adds"）
- 不以句號結尾
- 不超過 50 字符

#### Body（可選但推薦）

```
feat(payment): add support for installment plans

The system now allows users to pay in 3, 6, or 12 monthly installments.

- Added InstallmentPlan model
- Updated PaymentProcessor to handle installment logic
- Added validation for installment eligibility
```

#### Footer（用於關聯 Issue）

```
feat(dashboard): redesign user dashboard

Closes #123
Fixes #456
Related-to #789
Breaking-change: API response format changed
```

#### 完整示例

```
feat(api): add endpoint for batch user creation

Add a new POST /api/users/batch endpoint that allows creating
multiple users in a single request for improved performance.

Benefits:
- Reduced API calls from N to 1 for bulk operations
- Better performance for data migrations
- Transaction-based consistency

Changes:
- Added BatchCreateUserRequest and BatchCreateUserResponse DTOs
- Implemented BatchUserService with transaction handling
- Added validation for duplicate emails within batch
- Added comprehensive test coverage

Closes #522
```

---

## 常見問題與模式

### 常見拒絕原因

#### ❌ PR 太大

**症狀:** >500 行改動或 >15 個文件

**審查者反應:** 「這太大了，難以全面審查」

**解決:**
```
將 PR 拆分為：
1. 基礎設施變更（一個 PR）
2. 核心邏輯（一個 PR）
3. UI 改進（一個 PR）
```

---

#### ❌ 測試不足

**症狀:** 添加 100 行代碼，但只有 10 行測試

**審查者反應:** 「需要更多測試覆蓋」

**解決:**
```
按 Strict Mode 要求：
- Python: 補加到 90% 覆蓋
- TypeScript: 補加到 80% 覆蓋
```

---

#### ❌ 混雜功能與格式改動

**症狀:** 既改業務邏輯，又改代碼風格

**審查者反應:** 「分離功能改動和樣式改動」

**解決:**
```
Commit 1: refactor(style): run Prettier on module X
Commit 2: feat(feature): implement new feature Y
```

---

#### ❌ 沒有文檔更新

**症狀:** 修改了 API，但沒更新 README 或 Swagger

**審查者反應:** 「請更新相關文檔」

**解決:**
```
在 PR 中：
- 更新 README.md
- 更新 API 文檔
- 更新數據庫 Schema 文檔
```

---

## 自動化檢查

### GitHub Bot 自動評論的意義

GitHub Actions 運行後會自動評論：

```markdown
✅ **All checks passed!**

✓ Code format (Prettier/Black)
✓ Linting (ESLint/Ruff)
✓ Type checking (TypeScript/Pyright)
✓ Tests (90%+ coverage)
✓ Security scan
```

**此時 PR 可合併，無需人工檢查這些項。**

人工代碼審查只需關注：
- 邏輯是否正確
- 是否遵循業務規則
- 是否有更優的實現方法

### 如何與自動化互動

```bash
# 推送 → GitHub Actions 運行
git push

# 如果因為 Lint 失敗
# 本地修復并推送
ruff check --fix .
git add .
git commit -m "fix: resolve lint issues"
git push

# PR 自動更新，檢查重新運行
```

---

## 最佳實踐

### 作為 PR 作者

1. **自檢**（提交前）
   ```bash
   # 運行本地檢查
   npm run lint && npm run type-check && npm run test
   pytest --cov=src tests/
   dotnet build && dotnet test
   ```

2. **寫清楚描述**
   ```markdown
   ## 變動內容
   修復用戶登錄時的快取問題
   
   ## 為什麼
   用戶登出後，快取未清理，導致客端顯示舊數據
   
   ## 如何驗證
   1. 登錄系統
   2. 記記錄當前用戶名
   3. 登出
   4. 重新登錄為不同用戶
   5. 驗證用戶名已更新
   
   Related: #456
   ```

3. **及時回應**
   ```
   評論出現 → 24h 內回應
   ```

4. **拆分 PR**
   ```
   不要：一個 PR 做 5 個功能
   要：5 個小 PR，每個獨立
   ```

### 作為代碼審查者

1. **時間安排**
   ```
   小 PR (<100 行): 15-30 分鐘
   中 PR (<300 行): 30-60 分鐘
   大 PR (>300 行): 應該被拆分
   ```

2. **審查順序**
   ```
   1. 自動檢查結果（已通過就跳過）
   2. 整體設計（架構是否合理）
   3. 具體代碼（邏輯、命名、測試）
   4. 文檔與註釋
   ```

3. **提出建議（而非指責）**
   ```
   ❌ "這個函數太長了"
   ✅ "建議將驗證邏輯提取到獨立函數，會提高可讀性"
   
   ❌ "為什麼用這個名字？"
   ✅ "考慮用 validateUserEmail 替代 checkEmail，更明確用途"
   ```

4. **鼓勵好做法**
   ```
   "好的設計，使用依賴注入讓測試變得容易 👍"
   "測試涵蓋了邊界情況，很全面 ✨"
   ```

---

## 📚 相關文檔

- [最高工作流](../Master%20Workflow.md)
- [終極開發協定](../Ultimate%20Unified%20Protocol.md)
- [架構完整性](../Structural%20Integrity.md)
- [命名與規約](../Naming%20%26%20Logic%20Rules.md)

---

**最後更新：2026-02-13 | 版本：v1.0-mvp**
