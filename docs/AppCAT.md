# AppCAT CLI アセスメント手順

## AppCAT とは

**Azure Application Compatibility Toolkit (AppCAT)** は、Microsoft が提供するアプリケーション移行アセスメントツール。

- **目的**: アプリを Azure（AKS / App Service / Container Apps）へ移行する際の互換性問題・変更箇所を静的解析で特定する
- **対応言語**: Java、.NET など複数言語に対応（本手順は Java 向け AppCAT CLI を使用）
- **ベース**: オープンソースの [Konveyor/Windup](https://github.com/konveyor/windup) をベースに、Azure向けにカスタマイズ
- **対応ターゲット**: Azure専用（`azure-aks` / `azure-appservice` / `azure-container-apps`）
- **解析方式**: ソースコード・バイナリを静的解析し、移行に必要な変更箇所をレポート出力
- **出力形式**: JSON / HTML レポート

> **注意**: AppCAT は Azure への移行専用ツール。AWS・GCP などへの移行アセスメントには対応していない。

---

## 前提条件

| 項目 | 状態 |
|------|------|
| AppCAT CLI | `%USERPROFILE%\.appcat\appcat.exe` (v7.7.0.9) |
| Java | 20.0.2（インストール済み） |
| `appcat` コマンド | PATHに未登録 → フルパスで実行 or PATH追加 |

---

## 1. PATHに追加（任意・セッション内のみ）

```powershell
$env:PATH += ";$env:USERPROFILE\.appcat"
appcat version
```

永続化する場合：
```powershell
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";$env:USERPROFILE\.appcat", "User")
```

---

## 2. 利用可能な技術を確認する

```powershell
# ターゲット一覧（azure-aks / azure-appservice / azure-container-apps）
appcat analyze --list-targets

# ソース一覧（spring / springboot / java 等）
appcat analyze --list-sources
```

---

## 3. アセスメントを実行する

### ターゲット・ソースの省略について

`--target` と `--source` はどちらも省略可能。省略時はすべての技術が対象になる。

| オプション | 省略時の動作 |
|------------|-------------|
| `--target` | `azure-aks`, `azure-appservice`, `azure-container-apps` の全て |
| `--source` | 全ソース技術 |

```powershell
# ターゲット・ソースをすべて省略（全Azure サービス対象）
appcat analyze `
  --input  "C:\GitHub\app_mod\spring-webflow" `
  --output "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment" `
  --overwrite
```

### 基本コマンド

```powershell
appcat analyze `
  --input  "C:\GitHub\app_mod\spring-webflow" `
  --output "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment" `
  --source spring `
  --target azure-appservice `
  --overwrite
```

### ターゲットを複数指定する場合

```powershell
appcat analyze `
  --input  "C:\GitHub\app_mod\spring-webflow" `
  --output "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment" `
  --source spring `
  --target azure-aks,azure-appservice,azure-container-apps `
  --overwrite
```

### モードの使い分け

| モード | 解析対象 | 技術スタック検出 | 推定時間 | 用途 |
|--------|---------|----------------|--------|------|
| `issue-only` | ソースコード（問題のみ） | なし | 最速 | 修正すべき問題だけ把握したい場合。`modernize assess` と同等 |
| `source-only` | ソースコード（問題＋技術検出） | あり | 中程度 | **移行前の全体把握に推奨**。ソースが揃っている場合はこれで十分 |
| `full` | ソースコード＋バイナリ（JAR等） | あり | 最も遅い | バイナリ依存ライブラリも含めた詳細分析が必要な場合 |

#### Spring WebFlow での実測値（azure-appservice ターゲット）

| モード | ルール数 | インシデント数 | 推定工数 | 技術検出ルール |
|--------|---------|-------------|--------|-------------|
| `issue-only`（`modernize assess` 相当） | 4 | 27 | 86 SP | なし |
| `source-only` | 17 | 801 | 83 SP | あり（discover-java-files: 746件、Spring XML: 8件 等） |

> **`source-only` vs `full` の違い**:  
> `source-only` はソースコードのみを解析。`full` はさらにバイナリ（JAR/クラスファイル）も解析する。  
> ソースコードが揃っているプロジェクトでは `source-only` で技術スタックも問題も十分に検出でき、`full` との差は小さい。  
> Spring WebFlow のように OSS でソースが公開されているプロジェクトは `source-only` で十分。

> **`modernize assess` との比較**:  
> `modernize assess` は内部で `issue-only` 固定で AppCAT を実行するため、技術スタック検出（`discover-*` 系ルール）は含まれない。  
> アプリ移行前の全体把握には AppCAT CLI で `source-only` を使うことを推奨。`full` はバイナリ依存分析が必要な場合のみ。

```powershell
# 移行前の全体把握（技術スタック + 問題検出）
appcat analyze `
  --input  "C:\GitHub\app_mod\spring-webflow" `
  --output "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment_full" `
  --mode   source-only `
  --overwrite

# modernize plan create 用（issue-onlyで高速）
appcat analyze `
  --input  "C:\GitHub\app_mod\spring-webflow" `
  --output "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment" `
  --mode   issue-only `
  --overwrite
```

---

## 4. 出力ファイルの確認

```powershell
Get-ChildItem "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment"
```

| ファイル/ディレクトリ | 内容 |
|----------|------|
| `report.json` | 詳細分析結果（JSON） |
| `result.json` | サマリー結果 |
| `analysis.log` | 実行ログ |
| `static-report/` | 静的HTMLレポート一式（React製SPA） |
| `static-report/index.html` | ブラウザで見れる静的レポート |

> **注意**: `report.html` は生成されない。HTMLレポートは `static-report/index.html` を使用する。

---

## 5. レポートをブラウザで確認する

```powershell
Start-Process "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment\static-report\index.html"
```

---

## 6. よく使うオプション一覧

| オプション | 説明 |
|------------|------|
| `--input` / `-i` | 解析対象のソースコードパス |
| `--output` / `-o` | 結果の出力先ディレクトリ |
| `--source` / `-s` | 移行元技術（例: `spring`, `springboot`） |
| `--target` / `-t` | 移行先技術（例: `azure-appservice`） |
| `--mode` / `-m` | 解析モード（`issue-only` / `source-only` / `full`） |
| `--overwrite` | 既存の出力を上書き |
| `--overwrite=false` | 上書きしない（デフォルト） |
| `--rules` / `-l` | カスタムルールファイルやディレクトリを追加 |
| `--enable-default-rulesets=false` | デフォルトルールセットを無効化 |
| `--skip-static-report` | HTMLレポートの生成をスキップ（高速化） |
| `--analyze-known-libraries` | OSS依存ライブラリも解析対象にする |
| `--packages` | 解析対象パッケージを絞り込む |
| `--log-level` | ログレベル（0=ERROR〜5=DEBUG、デフォルト4） |
| `--disable-telemetry` | テレメトリを無効化 |
| `--dry-run` | フラグ検証のみ（実際には解析しない） |
