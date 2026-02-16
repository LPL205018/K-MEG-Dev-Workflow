# 工程自動化部署與實施計劃 (Deployment & Implementation Plan)

> **目的：** 將完整的工程自動化系統從設計文檔部署到實際項目
> **預期時間：** 3-5 天（取決於 repo 現狀）
> **涉及角色：** Tech Lead, DevOps, 全體開發者

---

## 📋 部署檢查清單

### Phase 1: 準備階段（Day 1）

#### 1.1 環境檢查

- [ ] 確認團隊成員 GitHub 帳號已設置
- [ ] 確認有 admin 權限訪問項目 repo
- [ ] Docker 已在開發機器上安裝（v24+）
- [ ] Node v20+ / Python 3.11+ / .NET 8+ 已安裝
- [ ] Git 已配置 SSH 或 HTTPS 認證

**檢查命令：**

```bash
# 檢查 Git
git --version

# 檢查 Docker
docker --version
docker compose version

# 檢查語言
node --version
python3 --version
dotnet --version
```

#### 1.2 複製配置文件

```bash
# 進入項目根目錄
cd your-k-meg-project

# 複製所有工作流文件
mkdir -p .github/workflows
cp Engineering-Automation/CI-CD-Pipelines/*.yml .github/workflows/

# 複製 Code Review 配置
cp Engineering-Automation/Code-Review-Guidelines/CODEOWNERS .github/
cp Engineering-Automation/Code-Review-Guidelines/PR_TEMPLATE.md .github/pull_request_template.md

# 複製開發環境配置
cp Engineering-Automation/Development-Environments/docker-compose.yml .

# 複製工具配置
cp Engineering-Automation/Tool-Configurations/PYTHON_SETUP.md docs/

# 提交改動
git add .github .dockerfiles docker-compose.yml docs/
git commit -m "chore: Add engineering automation infrastructure"
git push origin main
```

#### 1.3 驗證 GitHub 識別配置

進入 GitHub Repository：

1. **Settings → Actions → General**
   - 確認 GitHub Actions 已啟用
   - Workflow permissions: 選擇 "Read repository contents permission" 或 "Read and write permissions"

2. **Settings → Branches → Branch protection rules**
   - 創建新規則保護 `main` 分支：
     ```
     Require status checks to pass before merging:
     ✅ backend-python
     ✅ frontend-typescript
     ✅ backend-csharp
     ✅ cross-stack-checks
     ```

3. **Settings → Secrets and variables → Actions**
   - 如需要，添加敏感環境變數（預計 M3+ 時需要）

---

### Phase 2: 本地開發環境配置（Day 1-2）

#### 2.1 Docker Compose 啟動

```bash
# 進入項目目錄
cd your-k-meg-project

# 啟動所有服務
docker compose up -d

# 檢查容器狀態（所有應為 "Up"）
docker compose ps

# 查看日誌（排查啟動問題）
docker compose logs -f
```

**預期輸出：**

```
NAME                COMMAND                  SERVICE             STATUS
k-meg-postgres      "docker-entrypoint..." postgres            Up X seconds (healthy)
k-meg-redis         "redis-server --ap..." redis               Up X seconds (healthy)
k-meg-backend-py... "bash -c 'pip insta..." backend-python      Up X seconds
k-meg-backend-c...  "dotnet run"            backend-csharp      Up X seconds
k-meg-frontend      "npm run dev"           frontend            Up X seconds
```

#### 2.2 驗證服務可訪問

```bash
# Python FastAPI
curl http://localhost:8000/health
# 期望：HTTP 200 + JSON 響應

# 前端
curl http://localhost:5173
# 期望：HTTP 200 + HTML 內容

# C# .NET
curl http://localhost:8001/health
# 期望：HTTP 200

# 數據庫
docker exec k-meg-postgres psql -U devuser -d k_meg_dev -c "SELECT 1"
# 期望：輸入密碼後返回 1

# Redis
docker exec k-meg-redis redis-cli ping
# 期望：PONG
```

#### 2.3 配置 Git Hooks（本地檢查）

```bash
# 進入項目根目錄
cd your-k-meg-project

# 安裝 husky 和 commitlint
npm install husky @commitlint/cli @commitlint/config-conventional --save-dev

# 初始化 husky
npx husky install

# 創建 pre-commit hook（格式檢查）
npx husky add .husky/pre-commit "npm run format:check && npm run lint && pytest"

# 創建 commit-msg hook（消息檢查）
npx husky add .husky/commit-msg 'npx commitlint --edit "$1"'

# 創建 commitlint 配置
echo "module.exports = {extends: ['@commitlint/config-conventional']};" > commitlint.config.js

# 提交這些變動
git add .husky commitlint.config.js package.json package-lock.json
git commit -m "chore: Setup Git hooks for local validation"
git push origin main
```

---

### Phase 3: GitHub Actions 驗證（Day 2）

#### 3.1 觸發首個工作流

```bash
# 創建測試 branch
git checkout -b test/ci-first-run

# 做一個小改動（或不改）
echo "# Test" >> TEST_FILE.md

# 提交
git commit -m "test: trigger first CI workflow"
git push origin test/ci-first-run

# 創建 Pull Request (GitHub UI 或 CLI)
# gh pr create --title "Test: CI First Run" --body "Testing CI workflow"
```

#### 3.2 監控工作流執行

進入 GitHub：

1. **Actions tab** → 選擇 PR
2. 查看 "CI Pipeline - Multi-Stack" workflow
3. 檢查各 job 狀態：
   - ✅ backend-python （3-5 分鐘）
   - ✅ frontend-typescript （2-4 分鐘）
   - ✅ backend-csharp （2-3 分鐘）
   - ✅ cross-stack-checks （1 分鐘）

#### 3.3 處理首次失敗

**常見失敗 1: 格式化不通過**

```bash
# 本地修復
black src/backend/python
npm run format src/frontend
dotnet format src/backend/csharp

# 提交
git commit -am "style: auto-format code"
git push

# PR 自動更新，重新運行
```

**常見失敗 2: 測試覆蓋率不足**

```bash
# 添加缺失的測試
# 提交
git commit -m "test: improve coverage"
git push

# 檢查是否通過
```

#### 3.4 成功標誌

PR 評論出現：

```
✅ All checks passed!

✓ Code format (Prettier/Black)
✓ Linting (ESLint/Ruff)
✓ Type checking (TypeScript/Pyright)
✓ Tests (coverage met)
✓ Security scan
```

---

### Phase 4: 團隊準備與培訓（Day 3）

#### 4.1 制定團隊规则

在 GitHub 中配置：

1. **Require code reviews**
   ```
   Settings → Pull requests
   ✅ Require pull request reviews before merging
   ```

2. **自動分配審查者**
   ```
   .github/CODEOWNERS 已配置
   → 編輯以符合實際團隊構成
   ```

3. **分支保護規則**
   ```
   Settings → Branches → main
   ✅ Require status checks to pass
   ✅ Require code reviews (1 approval)
   ```

#### 4.2 團隊培訓（30 分鐘）

準備以下培訓內容：

**Part 1: 日常開發流程（15 分鐘）**

```bash
# 1. 本地開發
docker compose up -d
git checkout -b feature/new-feature

# 2. 編寫代碼
# ...正常開發...

# 3. 本地驗證
npm run lint && npm run type-check && npm run test
pytest --cov=app tests/
dotnet build && dotnet test

# 或一鍵檢查
make check  # 若配置了 Makefile

# 4. 遵循 Commit 規範
git commit -m "feat(auth): add new login method"

# 5. 推送 & 創建 PR
git push origin feature/new-feature
# 在 GitHub UI 中創建 PR

# 6. CI 自動運行，監控狀態

# 7. Code Review
# - 分配的人 review 代碼
# - 解決評論
# - Merge
```

**Part 2: 處理 CI 失敗（10 分鐘）**

```
常見失敗 → 解決方法表

Lint 失敗 → ruff check --fix / black
Type 失敗 → 檢查 tsconfig.json / pyright
Test 失敗 → 添加測試或修復代碼
Coverage 失敗 → 提高覆蓋率
```

**Part 3: Code Review 最佳實踐（5 分鐘）**

```
作為 Code Reviewer:
- 檢查自動化已驗證的項（✅ 可跳過）
- 專注於邏輯、設計、可讀性
- 用建議代替命令
- 鼓勵好做法

作為 PR 作者:
- 清晰描述為什麼
- 提前自檢
- 及時回應評論
- 感謝審查者
```

#### 4.3 常見問題解答（FAQ）

準備答案：

```
Q: container 報 port 已被占用怎麼辦？
A: ...

Q: 我的 commit 被 hook 拒絕了
A: ...

Q: CI 檢查為什麼這麼嚴格？
A: ...

Q: 我可以 skip CI 檢查嗎？
A: ...
```

---

### Phase 5: 生產前驗證（Day 4）

#### 5.1 示例 PR 流程

由 Tech Lead 执行一個完整示例：

```bash
# 1. 創建 branch
git checkout -b example/demo-workflow

# 2. 做一些改動（例如，添加一個函數 + 測試）
echo "def hello(): return 'world'" > hello.py
echo "def test_hello(): assert hello() == 'world'" > test_hello.py

# 3. 運行本地檢查
black . && ruff check . && pytest

# 4. Commit
git commit -m "feat(example): add demo function"

# 5. Push & PR
git push origin example/demo-workflow

# 6. 在 GitHub 創建 PR
# 描述：Demonstrating automated workflow

# 7. CI 自動運行（觀看直播）

# 8. Code review + 合併
# 展示 PR template 的使用方法
```

#### 5.2 文檔完整性檢查

確認以下文檔已準備好：

- [ ] README.md（如何貢獻）
- [ ] CONTRIBUTING.md（貢獻指南）
- [ ] .github/CODEOWNERS（審查者分配）
- [ ] docs/ 或 wiki（CI/CD 說明）
- [ ] Slack/Teams channel pin（核心文檔鏈接）

#### 5.3 回滾計劃

如果 CI 規則過於嚴格，準備回滾方案：

```
如果需要臨時放寬規則：
1. 編輯 .github/workflows/main-pipeline.yml
2. 修改相應的 --cov-fail-under 或 ignore 列表
3. 提交 PR（會自動應用新規則）
4. 合併
5. 所有後續 PR 使用新規則

不建議禁用整個檢查，而是調整閾值。
```

---

### Phase 6: 運營與維護（Day 5+）

#### 6.1 日常監控

每周檢查：

- [ ] CI 失敗率（目標 <5%）
- [ ] PR review 時間（目標 <24h）
- [ ] 覆蓋率趨勢（應穩定或上升）

```bash
# 檢查最近 10 個 PR 的 CI 狀態
gh pr list -L 10 --json statusCheckRollup,title

# 或查看 GitHub Actions 儀表板
```

#### 6.2 定期更新工具

```bash
# 月度檢查更新
pip list --outdated
npm outdated
dotnet nuget list updates

# 更新依賴
pip install --upgrade <package>
npm update
dotnet add package <package> --version <version>

# 測試更新
pytest --cov=app
npm run test
dotnet test
```

#### 6.3 改進反饋循環

**Month 1: 觀察期**
- 不做改動
- 收集團隊反饋

**Month 2: 調整期**
- 如果規則過度嚴格，降低某些閾值
- 如果發現新的 bug pattern，添加檢查

**Month 3+: 穩定運營**
- 定期季度評估
- H1/H2 計劃升級工具版本

---

## ⚠️ 常見陷阱與規避

### ❌ 陷阱 1：一開始就啟用所有檢查

**症狀：** CI 失敗率超過 50%，團隊抱怨

**規避方案：** 逐步啟用
```
Week 1: 格式化 + import 排序 (最簡單)
Week 2: 添加 Lint (中等難度)
Week 3: 添加類型檢查 (高難度)
Week 4: 添加測試覆蓋要求 (需要投入時間)
```

### ❌ 陷阱 2：忽視本地 Hook

**症狀：** 開發者經常提交不符合規則的代碼，浪費 CI 時間

**規避方案：** 強制安裝 Git Hooks
```bash
# 在 README 中明確說明
make setup-hooks
```

### ❌ 陷阱 3：規則衝突

**症狀：** 兩個 linter 要求相反的格式

**規避方案：** 讓 Prettier 掌控格式化，其他工具只檢查邏輯
```yaml
# Prettier 控制：縮進、換行、分號等
# Linter 控制：命名、import 順序、複雜度等
```

### ❌ 陷阱 4：測試覆蓋率遊戲

**症狀：** 開發者寫無意義的測試只為達到覆蓋率

**規避方案：** 定期檢查測試質量
```
不只看覆蓋率百分比，看：
- 測試是否有意義?
- 是否測試邊界情況?
- 是否實際會發現 bugs?
```

---

## 📊 預期時間表

```
Day 1 (4h)
├─ 環境檢查與準備
├─ 複製配置文件
└─ GitHub 設置 (branch protection)

Day 2 (6h)
├─ Docker Compose 啟動 & 驗證 (2h)
├─ Git Hooks 配置 (1h)
├─ 首個 workflow 運行 & 調試 (2h)
└─ PR template 驗證 (1h)

Day 3 (4h)
├─ 團隊培訓 (1.5h)
├─ FAQ & 故障排除 (1h)
├─ CODEOWNERS 配置 (1h)
└─ 文檔準備 (0.5h)

Day 4 (3h)
├─ 示例 PR flow (1h)
├─ 完整性檢查 (1h)
└─ 回滾計劃 (1h)

Day 5+ (1h/week)
├─ 監控 & 維護
├─ 工具更新
└─ 反饋改進
```

**總計：17-20 小時（小團隊）**

---

## 成功標誌

✅ 部署成功的指標：

- [ ] 所有服務能 `docker compose up -d` 啟動
- [ ] 首個 CI workflow 成功運行
- [ ] 團隊成員理解日常開發流程
- [ ] 至少 1 個完整示例 PR 合併
- [ ] 沒有遺漏的 error 日誌
- [ ] Code Review 流程正常運作

---

## 接下來？(Next Steps)

部署完成後（Day 5+）：

### M3-M6 計劃

```
Week 3-4 (M3):
└─ 部署自動化到 Staging 環境 (可選)

Week 5-8 (M4):
├─ 評估工具效果
├─ 收集團隊反饋
└─ 調整規則

Week 9-12 (M5):
├─ 先進檢查 (SonarQube, security scanning)
├─ 性能監控
└─ 自動化部署到生產

Week 13+ (M6):
├─ Kubernetes 部署 (如需要)
├─ 多區域部署
└─ 高級監控與告警
```

### 文檔維護

```
每月：
- [ ] 检查 README 是否還準確
- [ ] 更新常見問題

每季度：
- [ ] 評估規則是否還合適
- [ ] 檢查工具版本更新
- [ ] 與團隊進行回顧會議
```

---

## 📞 支援

遇到問題？

1. **檢查本指南的故障排除部分**
2. **查看相應工具的官方文檔**
3. **在 GitHub Discussions 提問**
4. **聯繫 Tech Lead / DevOps**

**最後更新：2026-02-13 | 版本：v1.0-mvp**
