# 新手工程師完全入門指南 (Beginner's Guide)

> **給零經驗工程師的詳細手冊**
> 
> 你是第一次進入團隊開發嗎？
> 從來沒用過 Git / Docker / CI/CD？
> 沒關係！本指南會一步步教你。

---

## 🎯 本指南的目標

在完成本指南後，你將能夠：

- ✅ 理解什麼是工程自動化（為什麼我們需要它）
- ✅ 在本地啟動完整的開發環境（5 分鐘）
- ✅ 遵循規範提交代碼（不會被 CI 拒絕）
- ✅ 通過代碼審查（知道審查者會看什麼）
- ✅ 當 CI 失敗時知道怎麼修復
- ✅ 貢獻代碼到團隊項目（第一個 PR）

**預期時間：** 60-90 分鐘首次閱讀 | 之後 5-10 分鐘查閱

---

## 📚 目錄

1. [背景知識：你需要了解的概念](#背景知識)
2. [第一次啟動：本地開發環境](#第一次啟動)
3. [你的第一次代碼提交](#第一次提交)
4. [理解 GitHub Actions（CI/CD）](#理解-ci)
5. [通過代碼審查](#代碼審查)
6. [常見問題排查](#常見問題)
7. [成為高手：最佳實踐](#最佳實踐)

---

## 背景知識

### 什麼是工程自動化？

**簡單說：** 你寫代碼，機器自動檢查、測試、審查，幫你在推送前發現錯誤。

**為什麼需要它？**

想象一個場景：

```
❌ 沒有自動化時：
你寫了 100 行代碼
 ↓
手工檢查代碼格式（花 15 分鐘）
 ↓
手工運行測試（花 10 分鐘）
 ↓
等待審查者檢查（花 2-4 小時）
 ↓
發現格式問題，重新改（又花 15 分鐘）
 ↓
再等一次審查（又花 2-4 小時）

總耗時：6-9 小時！

✅ 有自動化時：
你寫了 100 行代碼
 ↓
git commit（機器自動檢查格式 + 運行測試）← 30 秒
 ↓
git push
 ↓
GitHub Actions 自動執行全部檢查 ← 5 分鐘
 ↓
自動結果顯示在 PR 上
 ↓
審查者快速審查（只需看邏輯，不需檢查格式）← 10 分鐘
 ↓
merge

總耗時：30 分鐘！
```

**關鍵收益：** 機器做枯燥的工作，人類做需要思考的工作

---

### 你會接觸的 4 個關鍵技術

#### 1. **Git** - 代碼管理工具

```
簡單類比：Google Docs 的代碼版本
   ↓
   你可以看誰改了什麼
   ↓
   可以回滾到之前的版本
   ↓
   可以多人協作，避免覆蓋

常用命令：
git clone    ← 複製項目到本地
git add      ← 標記文件為「我要提交這個」
git commit   ← 生成一個保存點（附帶說明）
git push     ← 上傳到雲端（GitHub）
git pull     ← 從雲端下載最新代碼
```

#### 2. **Docker** - 環境管理工具

```
簡單類比：虛擬機，但輕量級
   ↓
   你的代碼運行的「完整環境」被打包進去
   ↓
   包含：數據庫、緩存、後端、前端等
   ↓
   其他人執行 docker-compose up -d
   ↓
   在 3 秒內得到完全相同的環境

為什麼重要？
沒有 Docker：
  小王電腦：Python 3.10 + PostgreSQL 14
  小李電腦：Python 3.9 + PostgreSQL 12
  結果：代碼在小王的機器上跑，在小李的機器上報錯
  
有 Docker：
  所有人容器內都是：Python 3.11 + PostgreSQL 16
  結果：Everyone 的結果都一樣！
```

#### 3. **CI/CD** - 自動化檢查工具（GitHub Actions）

```
簡單類比：自動質檢機器人

你 push 代碼
  ↓
GitHub Actions 機器人：
  ✓ 檢查代碼格式（Prettier/Black）
  ✓ 檢查代碼規則（ESLint/Ruff）
  ✓ 檢查類型正確（TypeScript）
  ✓ 運行所有測試
  ✓ 檢查測試覆蓋率
  ↓
3 秒後得到報告：
  通過 ✅（可以合併）
  失敗 ❌（需要修復）
```

#### 4. **Code Review** - 人工檢查流程

```
簡單類比：同事幫你檢查書法

你的代碼 push 上去
  ↓
GitHub 自動指派 2-3 個同事來 review
  ↓
同事們檢查：
  • 邏輯是否正確？
  • 名字是否有意義？
  • 有沒有 bug 陷阱？
  ↓
同事評論：「這裡很好」或「這裡建議改成...」
  ↓
你回應評論，做出改進
  ↓
同事點擊「Approve」
  ↓
merge！
```

---

## 第一次啟動

### Step 0: 安裝必需軟件

#### Windows 用戶

1. **安裝 Docker Desktop**
   - 下載：https://www.docker.com/products/docker-desktop
   - 點擊 exe 文件，一直點「Next」
   - 重啟電腦
   - 打開 PowerShell，執行：
     ```powershell
     docker --version
     ```
   - 如果看到版本號就是成功了 ✅

2. **安裝 Git**
   - 下載：https://git-scm.com/download/win
   - 一直點「Next」（所有選項保持默認）
   - 打開 PowerShell，執行：
     ```powershell
     git --version
     ```

3. **安裝代碼編輯器（選一個）**
   - VS Code（推薦新手）：https://code.visualstudio.com
   - 或 WebStorm/PyCharm（收費但強大）

#### Mac 用戶

```bash
# 使用 Homebrew（Mac 的包管理器）

# 安裝 Homebrew（如果還沒裝）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安裝 Docker Desktop
brew install docker

# 安裝 Git
brew install git

# 安裝 VS Code
brew install visual-studio-code
```

#### Linux 用戶

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install docker.io docker-compose-plugin git

# 驗證
docker --version
git --version
```

---

### Step 1: 克隆項目

**什麼是「克隆」？**
就是把雲端的代碼複製到你的電腦。

```bash
# 打開終端 / PowerShell / 命令行

# 進入你想放項目的目錄
cd ~/Documents
# 或 cd D:\Projects （Windows）

# 克隆項目
git clone https://github.com/your-org/k-meg.git

# 進入項目目錄
cd k-meg

# 驗證（應該看到很多文件）
ls
# 或 dir （Windows）
```

---

### Step 2: 啟動開發環境（魔法時刻！）

**這是最激動人心的一步。一句命令，整個環境就起來了。**

```bash
# 確保你在項目目錄
cd k-meg

# 執行魔法命令
docker compose up -d

# 等待 1-2 分鐘...

# 檢查狀態
docker compose ps

# 你應該看到像這樣的輸出：
# NAME                   STATUS
# k-meg-postgres         Up X seconds(healthy)
# k-meg-redis            Up X seconds (healthy)
# k-meg-backend-python   Up X seconds
# k-meg-frontend         Up X seconds
```

**恭喜！** 你現在有一個完整的開發環境了。

---

### Step 3: 訪問應用

```
在瀏覽器打開：

後端 API：        http://localhost:8000
API 文檔：         http://localhost:8000/docs ← 非常有用！
前端應用：        http://localhost:5173
數據庫管理：      http://localhost:5050 （用戶名：admin, 密碼：admin123)
Redis 管理：      http://localhost:8081

都應該能訪問！ ✅
```

---

### Step 4: 當環境起不來時

**問題 A：port 已被占用**

```
症狀：Error: port 5432 already in use

解決：
1. 查找占用的進程
   lsof -i :5432  # Mac/Linux
   netstat -ano | findstr :5432  # Windows

2. 殺死進程
   kill -9 <進程號>  # Mac/Linux
   taskkill /PID <進程號> /F  # Windows

3. 重新啟動
   docker compose up -d
```

**問題 B：Docker Daemon 未啟動**

```
症狀：Cannot connect to Docker daemon

解決：
Windows: 打開 Docker Desktop
Mac: 打開 Docker.app
Linux: 執行 sudo systemctl start docker
```

**問題 C：內存不足**

```
症狀：容器不斷重啟或崩潰

解決：
Windows/Mac: 
  打開 Docker Desktop
  Settings → Resources
  增加 CPU 和內存分配（推薦 4GB+）
  重啟 Docker
```

---

## 第一次提交

### 場景：修復一個簡單的 bug

假設你被分配了一個任務：
> 「在登錄頁面把按鈕顏色從紅色改成藍色」

#### Step 1: 創建功能分支

```bash
# 確保你在項目目錄
cd k-meg

# 首先同步最新代碼
git pull origin main

# 創建一個新分支（用於你的功能）
git checkout -b fix/login-button-color

# 驗證你已經在新分支
git branch
# 輸出應該顯示 * fix/login-button-color
```

**分支命名規則：**
```
fix/xxx          ← 修復 bug
feat/xxx         ← 新功能
refactor/xxx     ← 代碼重構
docs/xxx         ← 文檔改動
```

#### Step 2: 修改代碼

```bash
# 使用編輯器打開項目
code .  # 如果安裝了 VS Code

# 或手動打開 src/frontend/src/components/LoginButton.js
# 找到按鈕顏色的代碼，改成藍色
# 保存文件 Ctrl+S / Cmd+S
```

#### Step 3: 本地測試（重要！）

在提交前，確保代碼能運行：

```bash
# 進入前端目錄
cd src/frontend

# 運行測試
npm run test

# 運行格式檢查
npm run format:check

# 運行 Lint 檢查
npm run lint

# 全檢查（推薦）
npm run check

# 所有都通過後，回到項目根目錄
cd ../..
```

✅ 如果看到「All checks passed」，就可以提交了！

---

#### Step 4: 提交代碼（正確的方式）

```bash
# 查看你改了哪些文件
git status

# 應該看到類似：
# modified:   src/frontend/src/components/LoginButton.js

# 把改動標記為「要提交」
git add src/frontend/src/components/LoginButton.js
# 或一次性添加所有
git add .

# 提交（附帶詳細說明）
git commit -m "fix(ui): change login button color to blue"
```

**提交消息規則（非常重要）：**

```
格式：<type>(<scope>): <subject>

type = fix / feat / refactor / docs / test / chore
scope = 模塊名（可選）
subject = 簡短描述

✅ 好的例子：
  fix(login): change button color from red to blue
  feat(auth): add OAuth2 support
  refactor(api): simplify user validation logic
  
❌ 不好的例子：
  update code
  fix stuff
  WIP: testing things
```

#### Step 5: 上傳到雲端

```bash
# 上傳你的分支
git push origin fix/login-button-color

# 輸出應該類似：
# Remote branch pushed successfully
```

---

### Step 6: 創建 Pull Request（PR）

這是最重要的一步。**PR = 正式請求評審你的代碼**

#### 在 GitHub 上：

1. 打開 https://github.com/your-org/k-meg
2. 你應該看到一個黃色提示：「Your branch is X commits ahead」
3. 點擊「Compare & Pull Request」按鈕
4. 填寫 PR 信息：

   ```markdown
   ## 描述
   改動登錄按鈕顏色，從紅色改為藍色以符合新的 UI 規範。
   
   ## 為什麼這樣做
   根據設計規範，主要按鈕應該是藍色，增加視覺層級。
   
   ## 相關 Issue
   Closes #123
   
   ## 測試
   - [x] 在本地運行 npm run test
   - [x] 在 Chrome 上驗證顏色正確
   - [x] 在 Safari 上驗證顏色正確
   ```

5. 點擊「Create Pull Request」

**不要擔心，PR 不會馬上 merge，它只是開啟一個討論。**

---

## 理解 CI

### GitHub Actions 會自動做什麼？

當你 push 代碼後，GitHub 自動機器人會執行（大概 5 分鐘）：

**對於前端 (TypeScript/React)：**

```
檢查 1: 代碼格式 (Prettier)
  ✓ 縮進是否正確？
  ✓ 分號是否正確？
  ✗ 不符合 → 機器人會評論你

檢查 2: 代碼規則 (ESLint)
  ✓ 有沒有未使用的變數？
  ✓ 有沒有拼寫錯誤的變數名？
  ✗ 不符合 → 機器人會評論你

檢查 3: 類型檢查 (TypeScript)
  ✓ 函數調用時參數類型對嗎？
  ✗ 類型錯誤 → 機器人會評論

檢查 4: 測試
  ✓ 運行所有單元測試
  ✓ 檢查測試覆蓋率 (要求 ≥80%)
  ✗ 測試失敗 → 機器人會評論

檢查 5: 構建
  ✓ 能否成功編譯為 JavaScript？
  ✗ 編譯失敗 → 機器人會評論
```

**對於後端 (Python)：**

```
檢查 1: 代碼格式 (Black)
  ✓ 空格是否正確？
  
檢查 2: Lint (Ruff)
  ✓ 有沒有常見錯誤？
  
檢查 3: 類型檢查 (Pyright)
  ✓ 類型正確嗎？
  
檢查 4: 測試
  ✓ 運行單元測試
  ✓ 測試覆蓋率 (要求 ≥90%)
  
檢查 5: 安全
  ✓ 有沒有常見的安全漏洞？
```

---

### 當 CI 失敗時怎麼辦？

**場景：你 push 了代碼，CI 說『Lint 失敗』**

#### PhD 方案（修心法，很麻煩）

```
1. 閱讀機器人的評論
2. 自己手工修復每個問題
3. 再次 push
4. 希望下一次通過
```

#### 聰慧方案（借助工具）

```bash
# 對於 TypeScript/JavaScript
npm run format  # 自動格式化（修復大部分問題）

# 對於 Python
black src/     # 自動格式化
ruff check --fix src/  # 自動修復 Lint 問題

# 對於 C#
dotnet format  # 自動格式化

# 然後提交
git add .
git commit -m "style: auto-format code"
git push
```

**通常一個『自動修復 + push』，CI 就通過了。**

---

### 讀懂 CI 報告

**好的報告：**

```
✅ All checks passed!

✓ Code format (Prettier) ... passed
✓ Linting (ESLint) ... passed  
✓ Type checking (TypeScript) ... passed
✓ Tests (coverage 92%) ... passed
✓ Build ... passed

Result: PR can be merged!
```

**不好的報告：**

```
❌ Some checks failed!

✓ Code format (Prettier) ... passed
✗ Linting (ESLint) ... FAILED
  - Line 45: Variable 'unused' is defined but never used
  - Line 67: Missing semicolon

✗ Tests ... FAILED
  - test/login.spec.ts:12 - Expected true but got undefined

Next step:
1. 修復代碼
2. git add . && git commit -m "fix: ..." && git push
3. CI 會自動重新運行
```

---

## 代碼審查

### 人類審查者會檢查什麼？

**自動化已經檢查過的：**
```
✓ 代碼格式（自動檢查過）
✓ Lint 規則（自動檢查過）
✓ 類型正確（自動檢查過）
✓ 測試通過（自動檢查過）
```

**人類需要檢查的：**
```
1. 邏輯正確？
2. 函數名清楚嗎？
3. 有隱藏的 bug 陷阱嗎？
4. 設計是否最優？
5. 有沒有可以更簡潔的寫法？
```

### 你會收到什麼評論？

**評論 A：『請求修改』**

```
Reviewer: 「這個函數太複雜了，建議拆成三個小函數」

你應該這樣做：
1. 理解批評（不是人身攻擊，是改進建議）
2. 修改代碼
3. 提交新 commit
4. PR 自動更新，reviewer 會重新看
5. Reviewer 滿意後點「Approve」
```

**評論 B：『讚賞』**

```
Reviewer: 「很喜歡你這個解決方案，思路清晰 👍」

你應該：
1. 感謝對方
2. 等待他點「Approve」
3. Merge your code!
```

**評論 C：『提出問題』**

```
Reviewer: 「代碼在 X 情況下會崩潰嗎？」

你應該：
1. 回答問題
2. 如果有隱患，修復代碼
3. 重新 push
```

---

### 讓 Review 更順利的方法

#### 1. 寫好 PR 描述

```markdown
❌ 不好：
    Fixed login

✅ 好：
    改進登錄流程，支持 OAuth2

    ## 為什麼改動
    用戶反饋傳統密碼登錄麻煩，我們要支持 Google/GitHub 登錄。
    
    ## 改了什麼
    - 添加 OAuth2 認證流程
    - 修改登錄 UI 加入新按鈕
    - 添加配置文件支援 OAuth tokens
    
    ## 怎麼測試
    1. npm run dev
    2. 點擊 Google 按鈕
    3. 應該跳轉到 Google 登錄
    4. 登錄後回到應用，應該看到用戶信息
    
    Related: #123
```

#### 2. 提交漂亮的 Commit

```bash
# ❌ 不好：
git commit -m "stuff"
git commit -m "fixes"
git commit -m "more fixes"

# ✅ 好：
git commit -m "feat(auth): add OAuth2 Google login"
git commit -m "refactor(login): extract validation logic"
git commit -m "test(auth): add OAuth2 flow tests"
```

#### 3. 及時回應評論

```
Reviewer 提評論 → 24 小時內回應
（不要讓 PR 堆積）
```

---

## 常見問題

### Q1: 「我改了代碼但忘了測試，直接 push 了怎麼辦？」

```
回答：
1. 不用擔心，CI 會自動幫你检测
2. 看到失敗提示
3. 本地修復代碼
4. git add . && git commit -m "fix: ..." && git push
5. CI 自動重新檢測
```

### Q2: 「CI 說代碼格式不對，但我不知道錯在哪」

```
回答：
看 CI 的詳細報告，裡面會精確指出：
  - Line 45: Missing semicolon
  - Line 67: Unexpected indentation
  
然後運行自動修復：
  npm run format  # 或 black / dotnet format
```

### Q3: 「別人改了 code，我該 pull 嗎？」

```
回答：
在創建 PR 前：
  git pull origin main
  
在開發中如果遠端更新了：
  git fetch origin
  git rebase origin/main
  # 或直接 merge（取決於團隊慣例）
```

### Q4: 「我不小心改了不該改的文件怎麼辦？」

```
回答：
1. 查看狀態
   git status

2. 撤銷某個文件的改動
   git checkout -- <filename>
   
3. 或撤銷所有改動
   git reset --hard HEAD
```

### Q5: 「測試覆蓋率要求 90%，但我的代碼只有 70%」

```
回答：
加測試例！

# 測試路徑通常是 tests/ 或 __tests__/
# 新增測試文件

# 運行測試查看覆蓋率
pytest --cov=src tests/  # Python
npm run test -- --coverage  # TypeScript

# 覆蓋率會顯示：缺哪些測試
```

### Q6: 「Docker 容器裡代碼改了，為什麼本地沒看到？」

```
回答：
容器內運行的是 mount 的本地代碼。

改本地代碼 → 立刻反映到容器

如果沒反映：
1. 檢查文件是否保存
2. 如果還不行，重啟容器
   docker compose restart <service-name>
```

### Q7: 「我想看數據庫是否真的存了數據」

```
回答：
1. 打開 http://localhost:5050 (PgAdmin)
2. 登錄（admin / admin123）
3. 連接到 k_meg_dev 數據庫
4. 查看表和數據

或用命令行：
docker exec k-meg-postgres psql -U devuser -d k_meg_dev
\dt  # 列出所有表
SELECT * FROM users;  # 查看數據
```

### Q8: 「代碼上線後，用戶報告有 bug 怎麼辦？」

```
回答：
1. 在 GitHub 建立 Issue（描述 bug）
2. 創建 fix 分支
   git checkout -b fix/user-reported-bug
3. 修復 bug
4. 創建 PR
5. 等待 merge
6. 部署到生產環境

這個流程和開發新功能一樣。
```

---

## 最佳實踐

### 做一個好隊友的 7 個習慣

#### 1️⃣ 每天 pull（同步最新代碼）

```bash
# 開工第一件事
git pull origin main

# 這樣你有最新的代碼，避免衝突
```

#### 2️⃣ 提交前本地檢查

```bash
# 千萬不要依賴 CI 來發現問題
npm run check  # 或 make check

# 格式 + Lint + 類型 + 測試 都通過後再 push
```

#### 3️⃣ 寫清楚的提交信息

```
❌ 「update」「fix」「changes」
✅ 「feat(login): add OAuth2 support」
✅ 「refactor(api): improve error handling」
✅ 「fix(ui): correct button color in dark mode」
```

#### 4️⃣ 分支用完就刪

```bash
# Merge 後，刪除分支（避免混亂）
git branch -d fix/login-button-color

# 遠端分支也刪
git push origin --delete fix/login-button-color
```

#### 5️⃣ Code Review 要認真

```
你被指派 review 別人的代碼：

✓ 詳細看邏輯
✓ 想想有沒有 edge case
✓ 鼓勵好的做法（「我喜歡這個設計」）
✓ 建議改進（「建議改成...會更清楚」）
✗ 人身攻擊（「你怎麼這麼菜」）
```

#### 6️⃣ 遇到問題先自己排查

```
「CI 失敗了」
→ 先看失敗信息
→ 再查 github 上的文檔
→ 實在搞不懂才問同事

（大多數問題 Google 能找到答案）
```

#### 7️⃣ 定期同步與隊友的進度

```
# 每天看一次 Slack / 郵件
# 避免和別人改同一個文件導致衝突

# 如果衝突了
git status  # 看衝突的文件
# 編輯文件，手工合並
git add .
git commit -m "merge: resolve conflicts with feature-x"
```

---

## 完整案例：你的第一個功能開發

### 場景：添加一個「忘記密碼」按鈕

**Day 1 上午：需求評審**

```
Product Manager: 「用戶報告說忘了密碼無法登錄，
                 我們在登錄頁面加一個『忘記密碼』按鈕」

你: 「好的，button 放在密碼框下方嗎？」

PM: 「對，點擊後跳轉到重置流程」

設計師: 「我給了設計稿，見 Figma Link」

Tech Lead: 「後端同事已经準備好 API 了」
```

**Day 1 下午：開始實現**

```bash
# Step 1: 同步最新代碼
git pull origin main

# Step 2: 創建功能分支
git checkout -b feat/forgot-password-button

# Step 3: 看設計稿（Figma）
#         看後端 API 文檔
#         了解需求

# Step 4: 編寫代碼
code src/frontend/src/pages/Login.jsx
# 添加按鈕
# 添加點擊處理器
# 測試本地運作

# Step 5: 寫測試
code src/frontend/src/pages/__tests__/Login.spec.jsx
# 測試按鈕存在
# 測試點擊後跳轉

# Step 6: 本地驗證
npm run check
# 所有都通過 ✅
```

**Day 1 傍晚：提交代碼**

```bash
# Step 1: 提交改動
git add .
git commit -m "feat(ui): add forgot password button on login page"

# Step 2: 上傳
git push origin feat/forgot-password-button

# Step 3: 創建 PR（GitHub UI）
#         - 説明功能
#         - 附上設計稿截圖
#         - 説明測試方法
```

**Day 2 上午：等待 CI**

```
自動機器人檢查：

✓ 代碼格式 ... passed
✓ Lint ... passed
✓ 類型檢查 ... passed
✓ 測試 (覆蓋 85%) ... passed

所有 CI 檢查通過！ ✅

然後通知你的 reviewer
```

**Day 2 中午：Reviewer 審查**

```
Reviewer A (前端負責人):
「UI 看起來不錯，和設計稿對齐」

Reviewer B (Tech Lead):
「建議：如果 API 超時，有沒有 error handling？」

你回應：
「会的，加了 try-catch」

Reviewer B:
「👍 Approved」
```

**Day 2 下午：Merge**

```
Tech Lead 點擊 「Merge Pull Request」

你的代碼進入 main 分支！ 🎉

其他團隊成員的 git pull 會得到你的代碼
```

**Day 2 傍晚：部署**

```
DevOps 會自動：
1. 構建 Docker 鏡像
2. 部署到 Staging 環境
3. QA 團隊測試
4. 部署到生產環境

你的功能活了！
```

---

## 快速查閱表

### 我想... → 我該執行

| 目標 | 命令 |
|------|------|
| **启动開發環境** | `docker compose up -d` |
| **查看環境狀態** | `docker compose ps` |
| **同步最新代碼** | `git pull origin main` |
| **創建新功能分支** | `git checkout -b feat/xxx` |
| **查看改了哪些文件** | `git status` |
| **運行所有檢查** | `npm run check` 或 `make check` |
| **自動修復格式** | `npm run format` 或 `black .` |
| **提交代碼** | `git commit -m "type: description"` |
| **上傳到雲端** | `git push origin <分支名>` |
| **進入容器執行命令** | `docker exec k-meg-backend-python bash` |
| **查看容器日誌** | `docker compose logs -f <service>` |
| **重啟服務** | `docker compose restart <service>` |
| **停止所有容器** | `docker compose stop` |
| **清空容器和數據** | `docker compose down -v` |

---

## 你現在已經準備好了！

✅ 你理解了工程自動化  
✅ 你能啟動開發環境  
✅ 你知道如何提交代碼  
✅ 你明白 CI 是什麼  
✅ 你準備好接受 Code Review  
✅ 你能處理常見錯誤  

**下一步：** 挑個簡單任務，完成你的第一個 PR！

---

## 進階資源

讀完本指南後，你可以進一步了解：

| 話題 | 文檔 |
|------|------|
| 詳細 Git 教程 | [Git 官方文檔](https://git-scm.com/book/zh/v2) |
| Docker 深度學習 | [Docker 官方教程](https://docs.docker.com/get-started/) |
| 代碼品質規範 | [Code Review 清單](./Code-Review-Guidelines/REVIEW_CHECKLIST.md) |
| Commit 規範詳解 | [Conventional Commits](https://www.conventionalcommits.org/zh-hans/) |
| 故障排除 | [快速啟動指南](./Development-Environments/快速啟動指南.md) |

---

## 常用資源速查

```
需要幫助？這些地方能找到答案：

內部文檔：
  - README.md (在 Engineering-Automation 目錄)
  - 快速啟動指南.md
  - Code Review 清單.md

GitHub：
  - 項目的 Wikipedia 頁面
  - Issues（看別人遇過的問題）
  - Discussions（問技術問題）

Google 搜索：
  - "git merge conflict 怎麼解決"
  - "Docker 容器怎麼進去"
  - "npm test coverage 怎麼看"

直接問隊友：
  - 同事在 Slack 上
  - 或 Code Review 時當面問
```

---

## 最後的鼓勵

**你現在對工程自動化的了解程度：** ⭐⭐⭐⭐ (4/5)

**缺少的 1 星是什麼？** 實踐經驗。

**怎麼獲得？** 做！從簡單任務開始，逐漸提升難度。

**害怕出錯？** 完全正常。軟件開發就是在安全的范圍內出錯、學習、改進。

**第一個 PR 拒絕了？** 也很正常。優秀的代碼經歷過 5+ 輪 review。

**三個月後，你將成為：**
- ✅ 能獨立完成功能開發
- ✅ 能有效進行 Code Review
- ✅ 能幫新人入門
- ✅ 能優化流程和工具

**Go ahead, make your first PR!** 🚀

---

**最後更新：** 2026-02-13  
**版本：** v1.0  
**難度級別：** 初級 (Beginner)

如有問題，先讀本指南，再看快速啟動指南，最後問隊友。祝你工程愉快！🎉
