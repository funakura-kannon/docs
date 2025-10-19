# uv：Python開発環境の包括的管理

## 0. 概要

**uv** は Rust 製の高速な Python パッケージ／プロジェクト／Python 本体のバージョン管理ツールです。  
これまで個別に行っていた以下のツールの機能を統合的に扱うことができます。

- `pip` / `pip-tools` / `virtualenv`（パッケージ管理・環境構築）
- `pyenv`（Python バージョン管理）
- `pipx` / `poetry`（プロジェクト依存管理）
- `twine`（配布補助）

uv は「プロジェクトモード（pyproject.toml＋uv.lock）」と  
「pip互換モード（uv pip …）」の二層構成を持ちます。

---

## 1. インストール

### 推奨手順

#### macOS / Linux
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
````

#### Windows（PowerShell）

```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Homebrew や pip 経由の導入も可能です。

---

## 2. Python のバージョン管理

### 2.1 バージョンの取得・一覧・指定

```bash
uv python install 3.10 3.11 3.12
uv python install 3.12.3
uv python install '>=3.8,<3.10'
```

* 利用可能な形式例：
  `3.12`, `3.13t`（free-threaded 版）, `pypy@3.10`, `cpython-3.12.3-macos-aarch64-none`
* コマンド単位で指定可能：

  ```bash
  uv venv --python 3.11
  uv run --python 3.11 script.py
  ```

### 2.2 プロジェクト既定バージョンの固定

```bash
uv python pin 3.12
```

`.python-version` が作成され、プロジェクト全体の Python バージョンを固定します。
グローバルに固定する場合は `--global` オプションを付与します。

### 2.3 自動ダウンロードと検出

* 指定した Python が存在しない場合、uv は自動でダウンロードします。
* 自動ダウンロードを無効化する設定も可能です。
* uv 管理下の Python とシステム Python は区別されます。

### 2.4 アップグレード

```bash
uv python upgrade 3.12
uv python upgrade
```

* 前者は特定バージョンのパッチ更新、後者はすべての管理対象を更新します。

---

## 3. ライブラリのバージョン管理

### 3.1 プロジェクトモード（推奨）

#### 初期化と構成

```bash
uv init myproj
cd myproj
```

`pyproject.toml` と `uv.lock` が生成されます。
`uv run` または `uv sync` を実行すると `.venv` が自動作成されます。

構成例：

```
myproj/
  ├─ .venv/
  ├─ .python-version
  ├─ pyproject.toml
  ├─ uv.lock
  └─ main.py
```

#### 依存関係の操作

```bash
uv add requests
uv remove requests
uv sync
uv run python app.py
```

#### Lock / Sync の基本概念

* **Lock**：依存解決結果を `uv.lock` に記録
* **Sync**：`uv.lock` の内容を環境に反映

`uv run` は実行前に自動で Lock / Sync を行います。

#### Lock / Sync の補助コマンド

| 機能        | コマンド例                                    | 説明                         |
| --------- | ---------------------------------------- | -------------------------- |
| 更新要否の確認   | `uv lock --check`                        | ロック更新が必要か判定                |
| 厳格実行      | `--locked` / `--frozen` / `--no-sync`    | 更新禁止や同期省略                  |
| 同期設定      | `--inexact`, `--no-dev`, `--only-dev` など | 依存グループ制御                   |
| Extras 管理 | `--extra`, `--all-extras`                | optional dependencies を含める |

#### アップグレード

```bash
uv lock --upgrade
uv lock --upgrade-package foo
uv lock --upgrade-package foo==1.2
```

#### エクスポート

```bash
uv export --format requirements-txt > requirements.txt
```

---

### 3.2 pip互換モード

従来の pip ワークフローを高速化するための低レベルインターフェース。

代表的な操作：

```bash
uv venv
uv pip install requests
uv pip uninstall requests
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
uv pip install --system requests
```

---

## 4. 基本的な利用例

### 4.1 新規プロジェクトの作成と実行

```bash
uv init hello-world --python 3.12
cd hello-world
uv add flask
uv run -- flask run -p 3000
```

### 4.2 既存プロジェクトの再現（標準手順）

1. 既存リポジトリを取得
2. `.python-version` または `pyproject.toml` に指定されたバージョンを確認
3. 依存関係を同期

   ```bash
   uv sync
   ```
4. 実行

   ```bash
   uv run python app.py
   ```
5. CI / 本番用（開発依存除外）

   ```bash
   uv sync --frozen --no-dev
   ```

### 4.3 `.venv` を明示的に作らずに運用する仕組み

* `uv add`, `uv sync`, `uv run` などを実行すると `.venv` が自動生成されます。
* 仮想環境が存在しない場合も uv が自動的に作成・利用します。
* 手動で作成したい場合は `uv venv` を実行。
* プロジェクト直下に必ず `.venv` を作りたい場合は環境変数で固定可能：

  ```bash
  export UV_PROJECT_ENVIRONMENT=.venv
  ```

### 4.4 スクリプト単体の実行

```python
# example.py
# /// script
# dependencies = ["requests"]
# ///
```

```bash
uv run example.py
```

### 4.5 CLIツール利用

```bash
uvx ruff --version
uv tool install ruff
```

---

## 5. 仮想環境の場所に関する補足

uv は仮想環境を以下の優先順位で作成します。

1. 環境変数 `UV_PROJECT_ENVIRONMENT` が指定されている場合 → その場所
2. `.venv` ディレクトリが存在する場合 → 再利用
3. `pyproject.toml` のあるディレクトリ → `.venv` を作成
4. それ以外の場合 → キャッシュディレクトリ（例：`~/.cache/uv/envs/...`）

確認コマンド：

```bash
uv env find
uv venv list
```

---

## 6. `pyproject.toml` と `uv.lock` の違い

| ファイル名              | 目的                   | 管理内容                | 編集対象        | 更新タイミング                    |
| ------------------ | -------------------- | ------------------- | ----------- | -------------------------- |
| **pyproject.toml** | プロジェクトの仕様書（依存の希望・制約） | パッケージ名やバージョン範囲      | 開発者が手動編集    | `uv add` / `uv remove` 実行時 |
| **uv.lock**        | 依存解決結果（確定情報）         | 厳密なバージョン・ハッシュ・依存ツリー | uv が自動生成・管理 | `uv lock` / `uv sync` 実行時  |

### `pyproject.toml` の例

```toml
[project]
name = "sample-app"
version = "0.1.0"
requires-python = ">=3.11"

dependencies = [
  "requests>=2.32,<3.0",
  "numpy>=2.3"
]
```

### `uv.lock` の例（抜粋）

```toml
[[package]]
name = "numpy"
version = "2.3.4"
hashes = ["sha256:abc123..."]

[[package]]
name = "requests"
version = "2.32.3"
dependencies = ["urllib3>=1.26.0"]
```

### 両者の関係

```
pyproject.toml  ──(依存解決)──▶  uv.lock
「希望・制約」                    「確定結果」
```

---

## 7. ベストプラクティス

1. `.python-version` を必ずリポジトリに含め、チーム全体で統一。
2. `uv run` による自動 Lock / Sync で再現性を保証。
3. CI / 本番では `--frozen --no-dev` オプションで安定化。
4. `uv sync` は既定で「exact」（余剰パッケージ削除）動作。
5. `uv pip install --system` は特殊用途（CI や Docker ビルド）に限定。
6. Python 本体のアップグレードはマイナーバージョン内で慎重に行う。

---

## 8. コマンド早見表

| 分類       | 主なコマンド                             | 説明                  |
| -------- | ---------------------------------- | ------------------- |
| Python管理 | `uv python install 3.12`           | Python の取得          |
|          | `uv python pin 3.12`               | プロジェクト固定            |
|          | `uv python upgrade`                | パッチ更新               |
| プロジェクト依存 | `uv init <name>`                   | 初期化                 |
|          | `uv add <pkg>` / `uv remove <pkg>` | 依存追加・削除             |
|          | `uv sync`                          | ロック同期               |
|          | `uv lock` / `uv lock --upgrade`    | ロック更新               |
|          | `uv run <cmd>`                     | 実行（Lock/Sync 自動）    |
| pip互換    | `uv venv`                          | 仮想環境作成              |
|          | `uv pip install <pkg>`             | パッケージ導入             |
|          | `uv pip compile` / `uv pip sync`   | requirements ワークフロー |
| ツール      | `uvx <tool>`                       | 一時実行                |
|          | `uv tool install <tool>`           | 常設インストール            |

---

## 9. まとめ

* **`pyproject.toml`**：依存の希望・仕様書
* **`uv.lock`**：依存の確定結果・再現性保証
* **`uv init`**：プロジェクト初期化（`--python` でバージョン指定可）
* **`uv sync` / `uv run`**：仮想環境自動生成と依存反映
* **`.venv`**：明示作成不要、必要に応じて自動生成
* **`uv env find`**：現在の環境パスを確認

uv は Python 環境構築・管理のあらゆるステップを統一的かつ再現性高く扱うための、
現時点で最も包括的なツールといえます。