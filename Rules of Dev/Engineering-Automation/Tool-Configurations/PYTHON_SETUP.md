# Python 工具配置完整指南

---

## 📋 文件清單

```
Tool-Configurations/python/
├─ pyproject.toml      # 依賴、工具配置（推薦）
├─ ruff.toml           # Ruff Linter 配置
├─ pytest.ini          # Pytest 配置
├─ mypy.ini            # Mypy 配置（可選）
└─ .flake8             # Flake8 配置（可選，用 Ruff 替代）
```

---

## 1. pyproject.toml（主配置文件）

應放在項目根目錄。此文件集中管理所有 Python 工具配置。

```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "k-meg-backend"
version = "1.0.0"
description = "K-MEG Platform Backend API"
requires-python = ">=3.11"
authors = [
    {name = "K-MEG Team", email = "dev@k-meg.local"},
]
license = {text = "MIT"}
dependencies = [
    "fastapi>=0.104.0",
    "uvicorn>=0.24.0",
    "sqlalchemy>=2.0.0",
    "psycopg2-binary>=2.9.0",
    "redis>=5.0.0",
    "pydantic>=2.0.0",
    "pydantic-settings>=2.0.0",
    "alembic>=1.13.0",
]

[project.optional-dependencies]
dev = [
    # 測試框架
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "pytest-asyncio>=0.21.0",
    "pytest-mock>=3.12.0",
    # Lint & Format
    "ruff>=0.1.0",
    "black>=23.10.0",
    # 類型檢查
    "pyright>=1.1.300",
    "mypy>=1.6.0",
    # 安全掃描
    "bandit>=1.7.0",
    # 代碼複雜度
    "radon>=6.0.0",
]
docs = [
    "sphinx>=7.0.0",
    "sphinx-rtd-theme>=2.0.0",
]

[tool.setuptools]
packages = ["app"]

# ===== Black 配置 =====
[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
extend-exclude = '''
/(
  # 目錄
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
  | migrations
)/
'''

# ===== Ruff 配置 =====
[tool.ruff]
# 選擇規則集
select = [
    "E",      # pycodestyle 錯誤
    "W",      # pycodestyle 警告
    "F",      # Pyflakes
    "I",      # isort 導入排序
    "N",      # pep8-naming
    "UP",     # pyupgrade
    "B",      # flake8-bugbear
    "A",      # flake8-builtins
    "C4",     # flake8-comprehensions
    "RUF",    # Ruff 特定規則
]

# 排除規則
ignore = [
    "E501",    # 行太長（被 Black 處理）
    "W292",    # 文件末尾缺少換行符
]

line-length = 88
target-version = "py311"

# 排除目錄
exclude = [
    ".git",
    "__pycache__",
    ".venv",
    ".tox",
    "venv",
    "migrations",
]

# 強制單行導入
force-single-line = false
# 排序導入
known-first-party = ["app"]

[tool.ruff.per-file-ignores]
# __init__.py 允許未使用的導入
"__init__.py" = ["F401"]
# 測試文件允許 assert
"tests/**" = ["F401", "F841"]

# ===== Pytest 配置 =====
[tool.pytest.ini_options]
# 測試路徑
testpaths = ["tests"]
# Python 文件模式
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]

# 最小 Python 版本
minversion = "7.0"

# 覆蓋率
addopts = """
    --cov=app
    --cov-report=html
    --cov-report=term-missing
    --cov-fail-under=90
    --cov-branch
    -v
    --strict-markers
    --tb=short
"""

# 標記
markers = [
    "slow: 標記為慢速測試",
    "integration: 標記為集成測試",
    "unit: 標記為單元測試",
    "db: 標記為需要數據庫的測試",
]

# 非同步測試
asyncio_mode = "auto"

# 檔案監視
testpaths = ["tests"]

# ===== Pyright 配置 =====
[tool.pyright]
include = ["app"]
exclude = ["**/__pycache__", "**/.*", "venv"]

# 嚴格模式
typeCheckingMode = "strict"

# 報告級別
reportGeneralTypeIssues = "error"
reportOptionalMemberAccess = "error"
reportOptionalOperand = "error"
reportOptionalIterable = "error"
reportUnusedImport = "error"
reportUnusedClass = "warn"
reportUnusedFunction = "warn"
reportUnusedVariable = "warn"
reportConstantRedefinition = "error"
reportUnnecessaryIsInstance = "warn"
reportUnnecessaryCast = "warn"
reportUnnecessaryComparison = "warn"
reportIncompatibleMethodOverride = "error"
reportIncompatibleVariableOverride = "error"

# 忽略某些類型錯誤
reportPrivateUsage = "warning"

# ===== Coverage 配置 =====
[tool.coverage.run]
branch = true
source = ["app"]
omit = [
    "*/__init__.py",
    "*/migrations/*",
    "*/tests/*",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
    "class .*\\bProtocol\\):",
    "@(abc\\.)?abstractmethod",
]
precision = 2

[tool.coverage.html]
directory = "htmlcov"

# ===== Mypy 配置（可選） =====
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
check_untyped_defs = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
no_implicit_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
strict_optional = true

[tool.mypy-tests]
ignore_errors = true
```

---

## 2. ruff.toml（可選，若文件量大）

若要詳細控制 Ruff，創建獨立配置文件：

```toml
[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "W", "F", "I", "N", "UP", "B", "A", "C4", "RUF"]
ignore = ["E501", "W292"]

# 排序導入
[tool.ruff.lint.isort]
known-first-party = ["app"]
force-single-line = false

# 命名規範
[tool.ruff.lint.pep8-naming]
classmethod-decorators = ["classmethod"]
staticmethod-decorators = ["staticmethod"]
```

---

## 3. pytest.ini（簡化版）

若只需簡單配置：

```ini
[pytest]
testpaths = tests
python_files = test_*.py *_test.py
addopts = --cov=app --cov-report=html --cov-fail-under=90 -v
markers =
    slow: 標記為慢速測試
    integration: 標記為集成測試
    unit: 標記為單元測試
```

---

## 4. .editorconfig（跨工具一致性）

放在項目根目錄，所有工具都會遵守：

```editorconfig
root = true

# 所有文件
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

# Python
[*.py]
indent_style = space
indent_size = 4

# JSON
[*.json]
indent_style = space
indent_size = 2

# YAML
[*.{yml,yaml}]
indent_style = space
indent_size = 2

# Markdown
[*.md]
trim_trailing_whitespace = false
```

---

## 使用方法

### 安裝工具

```bash
# 創建虛擬環境
python -m venv venv
source venv/bin/activate  # Mac/Linux
# 或 venv\Scripts\activate (Windows)

# 安裝依賴
pip install -e ".[dev]"

# 驗證安裝
black --version
ruff --version
pytest --version
pyright --version
```

### 運行檢查

```bash
# 格式化代碼
black src/app

# Lint 檢查
ruff check src/app

# 類型檢查
pyright src/app

# 運行測試 + 覆蓋率
pytest

# 查看覆蓋率報告
open htmlcov/index.html
```

### 一鍵執行所有檢查

```bash
# Makefile
.PHONY: check format lint type-check test

check: format lint type-check test

format:
	black src tests

lint:
	ruff check src tests

type-check:
	pyright src

test:
	pytest

# 運行：make check
```

---

## 調整嚴格度

### 降低要求（30-60 天後看是否有問題）

```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--cov-fail-under=85"  # 從 90 改成 85

[tool.ruff]
ignore = ["E501", "W292", "A001"]  # 添加更多忽略規則

[tool.pyright]
typeCheckingMode = "standard"  # 從 strict 改成 standard
```

### 提高要求（團隊熟悉後）

```toml
[tool.pytest.ini_options]
addopts = "--cov-fail-under=95"  # 從 90 改成 95

[tool.ruff.lint]
# 添加更多嚴格規則
select = ["E", "W", "F", "I", "N", "UP", "B", "A", "C4", "RUF", "S", "T"]
```

---

## 故障排除

### 錯誤：`ModuleNotFoundError: No module named 'app'`

```bash
# 確保 pyproject.toml 中的 packages 設置正確
# 或添加 src 路徑到 PYTHONPATH
export PYTHONPATH="${PYTHONPATH}:$(pwd)"
pytest
```

### 錯誤：`pyright: command not found`

```bash
# 檢查安裝
pip list | grep pyright

# 重新安裝
pip install --upgrade pyright
```

### 測試覆蓋率報告遺漏某些文件

```bash
# 檢查 coverage 設置
cat pyproject.toml | grep -A 10 "\[tool.coverage"

# 臨時禁用 omit 進行詳細分析
pytest --cov=app --cov-report=term-missing
```

---

## 最佳實踐

✅ **推薦**

- 使用 `pyproject.toml` 集中管理所有配置
- 定期運行 `pytest --cov` 檢查覆蓋率
- 在 CI/CD 中使用相同配置
- 定期更新工具版本（月度）

❌ **避免**

- 在不同地方有不同的配置（容易衝突）
- 硬編碼 magic number（使用 Constant）
- 跳過測試用例（使用 `@pytest.mark.skip`）
- 禁用類型檢查（除非有充分理由）

---

## 参考資源

- [Black 文檔](https://black.readthedocs.io/)
- [Ruff 文檔](https://docs.astral.sh/ruff/)
- [Pytest 文檔](https://docs.pytest.org/)
- [Pyright 文檔](https://github.com/microsoft/pyright)

