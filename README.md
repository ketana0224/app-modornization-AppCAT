# app-modernization-AppCAT

Azure へのアプリケーション移行（モダン化）を支援するツールチェーンの使い方・ノウハウをまとめたドキュメントリポジトリ。

## 概要

Java / .NET アプリを Azure（AKS / App Service / Container Apps）へ移行する際に必要な、以下のツールの操作手順・ベストプラクティス・設計上の制限事項を記載している。

| ツール | 役割 |
|--------|------|
| **AppCAT CLI** (Azure Application Compatibility Toolkit) | ソースコードを静的解析し、Azure 移行時の互換性問題を検出する |
| **GitHub Copilot App Modernization** (`modernize` CLI / VS Code 拡張) | AppCAT の解析結果をもとに AI がモダン化プランを生成し、コードを自動修正する |

## 推奨ワークフロー

```
Step 1: 包括的アセスメント  →  技術スタック・依存関係の全体把握          ※省略可
         (AppCAT CLI source-only  または  VS Code 拡張 Custom Assessment)

Step 2: Issue アセスメント  →  修正すべき問題の検出
         (VS Code 拡張の Assessment Dashboard)

Step 3: プラン作成          →  AppCAT の issue を起点に AI がタスクを生成
         (VS Code / VS 拡張の Run Task)

Step 4: コード修正実行      →  AI がソースコードを自動修正
         (VS Code / VS 拡張、または GitHub.com Coding Agent)
```

> AppCAT の検出結果と plan を確実に連動させるには、**VS Code 拡張 UI 経由のフロー（Step 2 → Run Task）を推奨する**。  
> CLI の `modernize plan create` は AppCAT 結果を plan に反映できない制限がある（詳細後述）。

## ドキュメント一覧

| ドキュメント | 内容 |
|------------|------|
| [docs/AppCAT.md](docs/AppCAT.md) | AppCAT CLI のセットアップ・オプション・実行手順 |
| [docs/GHCP_Mod.md](docs/GHCP_Mod.md) | GitHub Copilot Modernization Agent（`modernize` CLI）の詳細な使い方 |
| [docs/recommended_workflow.md](docs/recommended_workflow.md) | ツール選択の指針と推奨モダン化ワークフローの全体像 |
| [docs/modernize_plan_create_limitations.md](docs/modernize_plan_create_limitations.md) | `modernize plan create` CLI の設計上の制限・原因・対処方法 |

## 主なポイント

- **AppCAT CLI 単体** はスタンドアロンの HTML レポートと `report.json` を生成し、技術スタック検出も可能（`source-only` モード推奨）。ただし AI サマリーは生成されない。
- **`modernize assess`** は AppCAT を内包し、さらに AI が `summary.md` を自動生成する。`modernize plan create` へシームレスに連携できる。
- **CLI の `modernize plan create`** は AppCAT の検出結果を plan に反映できない設計上の制限がある。VS Code 拡張の Run Task（issue 起点）が正規フロー。
- **`report.json` が既に生成済みの場合**、`modernize assess` を再実行せずに `modernize plan create` から始めることができる。

## ライセンス

[MIT License](LICENSE)
