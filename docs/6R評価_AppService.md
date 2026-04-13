# 6R 評価ガイド

## 前提

- **移行志向**: クラウド移行（オンプレミスからの脱却）
- **PaaS 優先**: App Service → AKS → VM の優先順位
- **目的**: 各アプリに対して 6R（Rehost / Replatform / Refactor / Repurchase / Retire / Retain）を判断するための材料を収集する

---

## 6R とは

| R | 内容 |
|---|------|
| **Rehost** | 変更なしで Azure VM（IaaS）へ Lift & Shift |
| **Replatform** | 最小限の変更で PaaS（App Service 等）へ移行 |
| **Refactor** | アーキテクチャ・コードを再設計して移行 |
| **Repurchase** | SaaS 製品へ切り替え |
| **Retire** | 廃止 |
| **Retain** | 現状維持（移行しない） |

> **重要**: 6R のラベルはツールが自動で貼るものではない。ツールが収集した判断材料をもとに**人間が最終決定する**。

---

## 使用ツールと役割

| ツール | 役割 | 出力 |
|--------|------|------|
| **Azure Migrate Discovery** | 対象アプリの棚卸し・資産台帳の確定 | サーバー・アプリ一覧、ランタイム、依存関係マップ |
| **Azure Migrate Assessment** | 指定ターゲットへの移行可否・コスト評価 | Readiness（Ready / Ready with conditions / Not ready）、月額コスト |
| **AppCAT**（Java / .NET） | ソースコードの静的解析 | 互換性問題件数、問題の重要度（mandatory / optional） |

---

## 評価フロー

### ステップ概要

```
① Discovery（棚卸し）
        ↓
② App Service Assessment（全件）＋ VM Assessment（全件）※同時実行可
        ↓
③結果で振り分け → 必要なものだけ AppCAT 実行
        ↓
④ 6R 判定（人間）
```

---

### ① Discovery（Azure Migrate）

**目的**: 評価対象の確定

- Azure Migrate Appliance をオンプレミスに展開
- サーバー・アプリ・ランタイム・依存関係を自動収集
- **この時点では Readiness は出ない（棚卸しのみ）**
- ここで「何本を評価するか」のスコープが確定する

---

### ② Assessment（Azure Migrate）

**目的**: 移行可否・コストの評価

PaaS 優先（App Service → AKS → VM）の前提では、**App Service と VM を全件同時実行**する。

> AKS Assessment はリソースサイジング・コスト確認のみ有効。「コンテナ化できるか」は判定できないため一次評価には含めない。

---

### ③ 結果の振り分けと AppCAT 実行判断

| App Service | VM | AppCAT | 判定候補 |
|------------|-----|--------|---------|
| **Ready** | Ready | **不要** | **Replatform**（App Service） |
| **Ready with conditions** | Ready | **必要** | 問題少 → Replatform／問題多 → Refactor |
| **Not ready** | Ready | **不要** | **Rehost**（VM） |
| **Not ready** | Not ready | **不要** | **Retain** / Refactor 検討 |

> **AppCAT は「Ready with conditions」のアプリだけに絞る**ことで実行コストを最小化する。

---

### AppCAT 実行結果の判断基準

AppCAT が検出する問題は重要度で分類される。

| 重要度 | 内容 | 判断への影響 |
|--------|------|------------|
| **Mandatory** | 移行前に必ず対処が必要 | 件数が多い → Refactor 寄り |
| **Optional** | 推奨対応（移行自体は可能） | Replatform の範囲内で対応 |

「問題が多い／少ない」の閾値は公式に定義されていない。**工数見積もりと照らし合わせて人間が判断する。**

---

### ④ AKS（コンテナ化）の検討タイミング

AKS はコンテナ化が有力なアプリに対して後続で評価する。

```
App Service：Ready with conditions かつ AppCAT 問題少 → Replatform 確定
                      ↓
        「コンテナ化も検討したい」場合
                      ↓
        AppCAT の結果を確認
          ステートレス・ローカルファイル依存なし → コンテナ化可能
          ステートフル・COM 依存あり           → App Service が現実的
                      ↓
        コンテナ化可能と判断 → AKS Assessment でコスト確認
```

---

## 最終判定マトリクス（PaaS 優先）

| App Service | VM | AppCAT | 6R 判定 |
|------------|-----|--------|--------|
| Ready | Ready | — | **Replatform**（App Service） |
| Ready with conditions | Ready | 問題少 | **Replatform**（App Service） |
| Ready with conditions | Ready | 問題多 | **Refactor** |
| Not ready | Ready | — | **Rehost**（VM） |
| Not ready | Not ready | — | **Retain** または **Refactor** 検討 |
| —（使用率ゼロ） | — | — | **Retire** |
| —（SaaS 代替可能） | — | — | **Repurchase** |

> Retire・Repurchase は Discovery の使用率データや業務要件をもとに人間が判断する。ツールによる自動判定はない。

---

## ツール比較

| 項目 | Azure Migrate | AppCAT |
|------|-------------|--------|
| 判定対象 | インフラ・ランタイム・設定 | ソースコード・バイナリ |
| 判定単位 | サーバー / アプリ単位 | ファイル / 行単位 |
| Readiness 自動判定 | **あり** | なし |
| コード問題の検出 | なし | **あり** |
| 実行タイミング | 全件（Assessment） | Ready with conditions のみ |

---

## 参考

- [AppCAT CLI の使い方](AppCAT.md)
- [GitHub Copilot Modernization Agent の使い方](GHCP_Mod.md)
- [推奨モダン化ワークフロー](recommended_workflow.md)
- [modernize plan create の制限](modernize_plan_create_limitations.md)
