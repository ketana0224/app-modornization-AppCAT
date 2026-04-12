# 推奨モダン化ワークフロー

Azure への移行を検討する際、**全体把握 → 問題絞り込み → プラン作成 → コード修正** の順で進めることを推奨する。

---

## ツール選択の前提

AppCAT と GH Copilot App Modernization 拡張の連携は以下の通り：

| ツール | AppCAT 連携直接実行 | AppCAT 結果インポート | 対応言語 |
|--------|:-----------------:|:-------------------:|--------|
| **VS Code 拡張**（推奨） | ✅ | ✅ | Java, C#/.NET |
| **Visual Studio 2022/2026 拡張** | ✅ | ✅ | C#/.NET |
| **CLI**（Modernization agent） | ✅（assess のみ） | ❌ | Java, C#/.NET |
| **GitHub.com Coding Agent** | △ | △ | Java, C#/.NET |

> **「結果インポート」がある VS Code / Visual Studio 拡張を使うことを推奨する。**  
> CLI の `modernize plan create` は AppCAT の検出結果を plan に反映できない（結果インポート機能なし）。  
> 詳細は [modernize_plan_create_limitations.md](modernize_plan_create_limitations.md) を参照。

---

## 関連ドキュメント

| ドキュメント | 内容 |
|------------|------|
| [AppCAT.md](AppCAT.md) | AppCAT CLI の詳細な使い方・オプション一覧 |
| [GHCP_Mod.md](GHCP_Mod.md) | GitHub Copilot Modernization Agent (CLI) の詳細な使い方 |
| [modernize_plan_create_limitations.md](modernize_plan_create_limitations.md) | `modernize plan create` CLI の設計上の制限と背景 |

---

## ワークフロー全体像

```
Step 1: 包括的アセスメント  →  技術スタック・依存関係の全体把握（人間がレビュー）  ※省略可
         (AppCAT CLI / source-only  または  VS Code 拡張 Custom Assessment / Issues, Technologies & Dependencies)

Step 2: Issue アセスメント  →  修正すべき問題の検出・確認
         (VS Code 拡張の Assessment Dashboard で直接実行)

Step 3: プラン作成          →  AppCAT の issue を起点に AI がタスクを生成
         (VS Code / VS 拡張の Run Task ← AppCAT 結果と連動)

         ↓ここで人間が plan.md をレビュー〜

Step 4: コード修正実行      →  AI がソースコードを自動修正
         (VS Code / VS 拡張、『continue』または GitHub.com Coding Agent)
```

> **Step 1 は省略可能**  
> Step 1 は技術スタック全体把握や HTML レポートのスタンドアロン生成を目的とした手動確認用の手順である。  
> VS Code 拡張の **Custom Assessment**（Issues, Technologies & Dependencies）で同等の情報を得ることもできるため、拡張内に統一したい場合は Step 1 を省略してもよい。  
> 全体把握が不要な場合は **Step 2 ケース B**（Dashboard から直接アセスメント実行）から始めてよい。

---

## Step 1: 包括的アセスメント（技術スタック全体把握）

VS Code 拡張の **Custom Assessment** では Analysis Coverage として **Issues & Technologies** または **Issues, Technologies & Dependencies** を選択でき、技術スタック検出も拡張内で完結できる。  
AppCAT CLI（`source-only`）は、スタンドアロンの HTML レポート（`static-report/index.html`）と `report.json` をVS Codeを使わずに独立して生成したい場合に使用する。  

> **注意**: AppCAT CLI 単体は AI サマリー（`summary.md`）を自動生成しない。AI サマリーが必要な場合は`report.json` をもとに Copilot Chat で手動生成する（後述）。

### モードの選択指針

| モード | 技術スタック検出 | バイナリ解析 | 推奨シナリオ |
|--------|--------------|------------|------------|
| `source-only` | あり | なし（ソースのみ） | **ソースコードが入手可能な場合（推奨）** |
| `full` | あり | あり（JAR等も解析） | バイナリのみのサードパーティ依存が多い場合 |

> **なぜ `full` ではなく `source-only` を推奨するか**:  
> `full` はバイナリ（JAR/クラスファイル）まで解析するため時間がかかる。  
> ソースコードが入手可能なプロジェクト（自社開発・OSS）であれば `source-only` で技術スタック検出は十分にカバーできる。  
> サードパーティのバイナリのみで構成された依存が多い場合は `full` を選択する。
>
> **`source-only` が適するケース（具体例）**:
> - 自社開発の Spring Boot アプリで、全モジュールのソースコードが入手可能な場合
> - 本プロジェクト（Spring WebFlow）のように OSS でソースコードが公開されており、依存ライブラリも Maven Central から取得可能な場合
>
> **`full` が適するケース（具体例）**:
> - レガシーな EJB アプリで、一部モジュールがソースなし JAR として納品されている場合
> - ベンダー提供のバイナリ SDK を多用しており、そこに古い API が含まれている可能性がある場合

```powershell
appcat analyze `
  --input  "C:\GitHub\app_mod\spring-webflow" `
  --output "C:\GitHub\app_mod\spring-webflow\.github\modernize\assessment_full" `
  --mode   source-only `
  --overwrite
```

**確認ポイント**（`static-report\index.html` をブラウザで開く）:
- 使用フレームワーク・ライブラリ
- データベース・外部サービスの依存関係
- Java EE / Jakarta EE コンポーネントの使用状況

```powershell
Start-Process ".github\modernize\assessment_full\static-report\index.html"
```

### AIサマリーを生成したい場合

AppCAT CLI は `summary.md` を生成しない。`report.json` をもとに Copilot Chat でサマリーを生成し手動保存する。

> **注意**: VS Code 拡張の Recommended Assessment は `issue-only` 相当のため技術スタック情報が少ない。Custom Assessment で **Issues, Technologies & Dependencies** を選択するか、AppCAT CLI（`source-only`）の `report.json` から生成する方が情報量が多い。

Copilot Chat での依頼例：

```
.github/modernize/assessment_full/report.json を読んで、
Azure App Service への移行アセスメントサマリーを日本語で作成してください。
技術スタック一覧・必須/任意対応の問題・推定工数・推奨事項を含めてください。
```

生成結果を `.github\modernize\assessment_full\summary.md` として保存する。

> 詳細は [AppCAT.md](AppCAT.md) を参照。

---

## Step 2: Issue アセスメント（修正すべき問題の確認）

### ケース A: Step 1 を実施した場合 → Import で結果を取り込む

**VS Code での手順**:
1. サイドバーの **GitHub Copilot modernization** ペインを開く
2. **Open Assessment Dashboard** をクリックする
3. ダッシュボード右上の **Import** をクリックする
4. Step 1 の出力先（例: `.github\modernize\assessment_full\report.json`）を指定する
5. Assessment Report で検出された issue（CRITICALITY・SOLUTION・Run Task）を確認する

> **Import 時の注意**:  
> `source-only` の report.json（801件）は技術スタック検出エントリが大半のため、  
> VS Code 拡張が絞り込むとアクション可能 issue は 3件程度になる。  
> 移行 issue の網羅性は Step 1 を省略して Dashboard から直接実行（27件）の方が上。  
> Step 1 の `report.json` は技術スタック全体把握・AI サマリー生成の用途に留めることを推奨する。

---

### ケース B: Step 1 を省略した場合 → Dashboard から直接アセスメントを実行する

**VS Code での手順**:
1. サイドバーの **GitHub Copilot modernization** ペインを開く
2. **Open Assessment Dashboard** をクリックする
3. **Custom Assessment** を選択し、Analysis Coverage で **Issues, Technologies & Dependencies** を選ぶ（技術スタック・依存関係も検出）
4. Assessment Report で検出された issue（CRITICALITY・SOLUTION・Run Task）を確認する

> **Recommended Assessment vs Custom Assessment**:  
> Recommended は Analysis Coverage が固定（issue 相当）。技術スタックや依存関係も把握したい場合は Custom Assessment で **Issues, Technologies & Dependencies** を選択する。

---

## Step 3: プラン作成

### 推奨: VS Code 拡張 / Visual Studio 拡張の Run Task を使う

AppCAT の結果インポート機能により、検出した issue がそのままタスクに反映される。

**VS Code の場合**:
1. Assessment Report で対象の issue を選択
2. **Run Task** をクリック → Copilot エージェントが起動
3. エージェントが `plan.md` を生成し、内容を表示する

> 表示された `plan.md` をレビューし、タスク内容・スコープ・影響範囲を確認する。問題がなければ Step 4 へ進む。

### Run Task がない issue（Ask Copilot のみ）の場合

Assessment Report の ACTION 列が **Ask Copilot** のみで **Run Task がない issue** は、Microsoft 提供のビルトインソリューションが存在せず、`plan.md` が自動生成されない。

> **⚠️ Ask Copilot ボタンは Ask モードで開く**  
> UI の **Ask Copilot** ボタンをクリックすると Copilot Chat が **Ask モード**（読み取り専用）で起動する。  
> Ask モードではファイルの作成・編集が行われないため、`plan.md` への追記はできない。  
> **Agent モードに切り替えてから**指示する必要がある。

#### 手順: Agent モードで plan.md に追記する

1. Assessment Report で対象の issue 行を展開し、issue の内容（タイトル・説明）をコピーする
2. VS Code のサイドバーで Copilot Chat アイコンをクリックしてチャットを開く
3. チャット入力欄左端のモード切替アイコンをクリックし、**Agent** モードを選択する  
   （"Ask" や "Edit" と表示されている場合は "Agent" に切り替える）
4. 以下のように指示する：

   ```
   plan.md に次の issue の対応計画を追記してください：
   [コピーした issue の内容を貼り付け]
   ```

5. エージェントが `plan.md` を更新したら内容をレビューする
6. 問題がなければ Step 4 へ進む

> **本プロジェクトの例**:  
> - `Use of unsecured network protocols or URI libra...`（**Mandatory**）→ Ask Copilot のみ  
> - `Avoid using hardcoded URLs (HTTP protocol) in ...`（Optional）→ Ask Copilot のみ  
>
> 上記 2件は Run Task がないため、上記手順で **Agent モード**から `plan.md` への追記を指示する。  
> Mandatory issue は必ず対応すること。

---

## Step 4: コード修正実行

**VS Code の場合**:
1. `plan.md` をレビュー後、エージェントに **`continue`** と入力 → コード修正が始まる
2. 修正後、バリデーションループ（ビルド・テスト・CVE チェック）が自動実行される

> `plan.md` の内容に修正が必要な場合は、`continue` を入力せず Step 3 に戻り、チャットで指示を修正して Run Task を再実行する。

---

## フォルダ構成まとめ

```
.github\modernize\
  assessment_full\        ← Step 1: AppCAT CLI (source-only) の出力
    static-report\
      index.html          ← 技術スタック確認用 HTML レポート
    report.json
    summary.md            ← AI サマリー（Copilot Chat で手動生成）
```
