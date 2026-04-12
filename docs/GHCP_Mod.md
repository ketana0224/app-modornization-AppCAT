# GitHub Copilot Modernization Agent (CLI) 手順

## 概要

`modernize` CLI は GitHub Copilot App Modernization が提供するエージェントツール。AppCAT の静的解析結果をもとに、AIがモダン化プランを作成し、コード修正まで自動実行する。

| コマンド | 役割 |
|----------|------|
| `modernize assess` | アセスメント実行（AppCAT内包）+ AIサマリー生成 |
| `modernize plan create` | アセスメント結果からモダン化プランを作成 |
| `modernize plan execute` | プランに基づいてコード修正を実行 |
| `modernize upgrade` | Java/Spring Boot バージョンアップグレード |

- **バージョン**: 0.0.252
- **デフォルトモデル**: `claude-sonnet-4.6`

---

## AppCAT CLI 単体 vs modernize assess の違い

| 項目 | AppCAT CLI 単体 | `modernize assess` |
|------|----------------|-------------------|
| 実行モード | `full`（デフォルト） | `issue-only` |
| 検出ルール数 | 17種（同ターゲットで比較） | 4種 |
| 技術スタック検出（discover系） | **あり**（フレームワーク・DB・設定ファイル等） | なし |
| 実質的な問題検出 | あり | あり |
| dockerfile 不足の検出 | なし（`--source spring` 指定時） | **あり** |
| 出力先 | `.github\modernize\assessment\` | `.github\modernize\assessment\`（同じ） |
| `report.json` | 生成される | 生成される（内容は異なる） |
| HTML レポート | `static-report\index.html` | `report.html` |
| AI サマリー | **なし** | **あり**（`summary.md` を自動生成） |
| 主な用途 | 移行前の全体把握・技術スタック調査 | モダン化プラン作成用の問題検出 |
| `plan create` との連携 | `report.json` を参照して手動実行 | シームレスに次のステップへ |

**既に AppCAT CLI で `report.json` を生成済みの場合**、`modernize assess` を再実行する必要はない。
`report.json` が `.github\modernize\assessment\` に存在する状態で `modernize plan create` を実行すると、その結果を読み込んでプランを作成する。

```powershell
# AppCAT済みなら plan create から始めればよい
modernize plan create "Migrate to Azure" --language java
```

---

## 前提条件

- `modernize` コマンドが使えること（GitHub Copilot App Modernization for Java 拡張でインストール）
- GitHub 認証済みであること（`gh auth status` で確認）
- GH_TOKEN 環境変数が設定されている場合はそのまま使用可能

```powershell
# GitHub認証確認
gh auth status

# modernizeバージョン確認
modernize --version
```

---

## ワークフロー全体像

```
[1. assess]  →  [2. plan create]  →  [3. plan execute]
 AppCAT解析       AIがプラン作成       コード自動修正
 + AIサマリー      plan.md / tasks.json
```

---

## 1. アセスメントを実行する

`modernize assess` は AppCAT CLI を内部で呼び出し、さらに AI によるサマリーレポートを生成する。

```powershell
# 基本（カレントディレクトリを対象）
modernize assess

# ソースパスを明示する場合
modernize assess --source "C:\GitHub\app_mod\spring-webflow"

# 出力先を変更する場合（デフォルト: .github\modernize\assessment）
modernize assess --output-path ".github\modernize\assessment"

# 詳細ログを表示
modernize assess --verbose
```

**出力ファイル**（`.github\modernize\assessment\`）:

| ファイル/ディレクトリ | 内容 |
|----------------------|------|
| `report.json` | AppCAT 詳細解析結果 |
| `report.html` | ブラウザで確認できる HTML レポート |
| `result.json` | サマリー結果 |
| `summary.md` | AI が生成したサマリー（Markdown） |
| `analysis.log` | AppCAT 実行ログ |

> **注意**: `modernize assess` では `report.html` が生成される。
> `appcat analyze`（AppCAT CLI 直接実行）では `report.html` は生成されず、代わりに `static-report\index.html` が出力される。

```powershell
# HTMLレポートをブラウザで開く（modernize assess の場合）
Start-Process ".github\modernize\assessment\report.html"

# AppCAT CLI 直接実行の場合
Start-Process ".github\modernize\assessment\static-report\index.html"
```

---

## 2. モダン化プランを作成する

アセスメント結果をもとに AI がモダン化プランを作成する。

```powershell
# 基本（カレントディレクトリ対象）
modernize plan create "Migrate to Azure"

# 言語を明示する場合
modernize plan create "Migrate to Azure" --language java

# プラン名を指定する場合（デフォルト: modernization-plan）
modernize plan create "Migrate to Azure" --plan-name "my-plan"

# 既存プランを上書きする場合
modernize plan create "Migrate to Azure" --overwrite

# モデルを変更する場合
modernize plan create "Migrate to Azure" --model claude-opus-4.6
```

**出力ファイル**（`.github\modernize\modernization-plan\`）:

| ファイル | 内容 |
|----------|------|
| `plan.md` | 移行概要・タスク一覧（Markdown） |
| `tasks.json` | 各タスクの詳細な修正要件（JSON） |

---

## 3. プランを実行してコードを修正する

`plan.md` / `tasks.json` をもとに AI がソースコードを自動修正する。

```powershell
# 基本（ローカル実行）
modernize plan execute

# プロンプトで修正方針を指示する場合
modernize plan execute "execute the plan"

# ローカル実行を明示する場合
modernize plan execute --delegate local

# Cloud Coding Agent（GitHub Actions）に委譲する場合
modernize plan execute --delegate cloud

# Cloud 委譲して完了まで待機する場合
modernize assess --delegate cloud --wait
```

---

## 4. バージョンアップグレード（オプション）

Java や Spring Boot のバージョンアップグレードを単独で実行できる。

```powershell
# 最新 LTS へアップグレード（自動判定）
modernize upgrade

# バージョンを指定する場合
modernize upgrade "Java 21"
modernize upgrade "Spring Boot 3.2"
modernize upgrade ".NET 10"
```

---

## 5. モデルの選択

| モデル名 | ID | コスト倍率 |
|----------|----|----------|
| Claude Sonnet 4.6（デフォルト） | `claude-sonnet-4.6` | 1x |
| Claude Haiku 4.5（軽量・高速） | `claude-haiku-4.5` | 0.33x |
| Claude Opus 4.6（高精度） | `claude-opus-4.6` | 3x |
| GPT-5.2 | `gpt-5.2` | 1x |

```powershell
# モデルを指定して実行
modernize plan create "Migrate to Azure" --model claude-haiku-4.5
```

---

## 6. プランをリセットしてやり直す

```powershell
# modernization-plan だけ削除してプランを再作成
Remove-Item ".github\modernize\modernization-plan" -Recurse -Force

# アセスメントごとリセットする場合
Remove-Item ".github\modernize\assessment" -Recurse -Force
```

---

## 7. よくあるオプション一覧

| オプション | 説明 |
|------------|------|
| `--source` | 対象プロジェクトのパス（デフォルト: `.`） |
| `--model` | 使用する LLM モデル |
| `--language` | `java` または `dotnet` |
| `--plan-name` | プラン名（デフォルト: `modernization-plan`） |
| `--overwrite` | 既存プランを上書き |
| `--delegate local` | ローカルで実行（デフォルト） |
| `--delegate cloud` | Cloud Coding Agent（GitHub Actions）に委譲 |
| `--wait` | Cloud 委譲時に完了まで待機 |
| `--verbose` | 詳細ログを表示 |
| `--no-tty` | プレーンテキスト出力（CI/CD向け） |
