## uv：クイックスタート

### 1. uv のインストール

#### macOS / Linux
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
````

#### Windows（PowerShell）

```bash
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

---

### 2. プロジェクトの初期化（Python バージョン指定）

```bash
uv init myproj --python 3.12
cd myproj
```

* `--python` で使用する Python バージョンを明示指定できます。
* 指定したバージョンが未インストールの場合、自動で取得されます。
* `.python-version` が作成され、以後のコマンドはこのバージョンを使用します。

---

### 3. 仮想環境の作成

#### 自動生成（推奨）

`uv sync` または `uv run` を実行すると、プロジェクト直下に `.venv` が自動的に作成されます。

#### 明示的に作成する場合

```bash
uv venv
```

---

### 4. パッケージの追加と実行

```bash
uv add numpy
uv run python main.py
```

* `uv add` は `pyproject.toml` と `uv.lock` を自動更新し、依存を導入します。
* `uv run` はロック・同期を自動実行し、指定スクリプトを仮想環境内で実行します。

---

### 5. 環境のエクスポートと同期・ロック

```bash
uv export --format requirements-txt > requirements.txt
uv sync
uv lock
```

* `uv export`：既存ロック内容を `requirements.txt` 形式で出力
* `uv sync`：`uv.lock` に基づいて環境を再現
* `uv lock`：依存関係を解決してロックファイルを更新
