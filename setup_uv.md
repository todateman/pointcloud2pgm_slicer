# uv を用いた環境セットアップと実行方法

Pythonパッケージマネージャ`uv`を使って本プロジェクトの開発環境を構築・実行する方法です。

## 前提

- OS: Linux (Ubuntu 22.04)
- シェル: bash
- Python: 3.12 を推奨（`open3d` が 3.13 に未対応のため）
- GUI 実行環境（PyQt5 / pyvistaqt によるレンダリングが必要）。  
  GUIなしで動かす場合は `xvfb-run` 等の仮想ディスプレイを利用してください。

## 1. uv のインストール確認

```bash
command -v uv >/dev/null 2>&1 && echo "uv installed" || echo "uv not found"
```

`uv not found` の場合は、公式手順に従ってインストールしてください。

## 2. Python 3.12 の用意（必要時）

`open3d` が Python 3.12 までをサポートしているため、3.12 を用意しておくと確実です。

```bash
uv python install 3.12 --quiet
```

## 3. 仮想環境の作成

プロジェクト直下に `.venv` を作成します。

```bash
uv venv -p 3.12 .venv
```

作成後、仮想環境を有効化します。

```bash
source .venv/bin/activate
python -V
```

## 4. 依存関係のインストール（同期）

`requirements.txt` をもとに依存パッケージをインストールします。

```bash
uv pip sync requirements.txt
```

## 5. パッケージの開発インストール（任意）

プロジェクトを編集可能モードでインストールします（ソース変更が即反映されます）。

```bash
uv pip install -e .
```

## 6. プログラムの実行方法

### 6.1 仮想環境を有効化して実行（推奨）

```bash
source .venv/bin/activate
python -m pointcloud2pgm_slicer.main <入力点群.pcd> <出力ディレクトリ>
```

### 6.2 uv から直接実行（仮想環境の有効化不要）

```bash
uv run -p 3.12 python -m pointcloud2pgm_slicer.main <入力点群.pcd> <出力ディレクトリ>
```

### 6.3 GUI環境がない場合の実行（仮想ディスプレイ）

CI環境やリモートでディスプレイがない場合は、`xvfb-run` を使います。

```bash
# 事前にインストール（必要なら）
sudo apt-get update && sudo apt-get install -y xvfb

# 仮想ディスプレイで実行
xvfb-run -a uv run -p 3.12 python -m pointcloud2pgm_slicer.main <入力点群.pcd> <出力ディレクトリ>
```

## よくある質問（FAQ）

- Python 3.13 は使えますか？
  - 現状 `open3d` が 3.13 未対応のため、3.12 を推奨します。
- 出力ディレクトリが存在しなくても大丈夫？
  - はい。プログラム側で自動作成します（`model.convert_to_pgm` 内で `os.makedirs(..., exist_ok=True)` を実行）。
- CLI で GUI なしの変換だけしたい
  - 現在は GUI 前提のメインエントリです。必要なら、GUIなしで `PointCloudModel.convert_to_pgm` を呼ぶ専用 CLI を追加可能です。

## トラブルシューティング

- `ModuleNotFoundError` が出る
  - `python -m pointcloud2pgm_slicer.main` で実行しているか確認してください。パッケージ相対インポートに修正済みです。
- 画面が固まる／レンダリングでクラッシュする
  - ディスプレイ環境を用意するか、`xvfb-run -a` を利用してください。

---

再現性の高いセットアップ手順として、次の 4 ステップが最小構成です：

```bash
uv python install 3.12 --quiet
uv venv -p 3.12 .venv
uv pip sync requirements.txt
uv pip install -e .
```

その後、以下で実行できます：

```bash
uv run -p 3.12 python -m pointcloud2pgm_slicer.main <入力点群> <出力ディレクトリ>
```

