# 6R 評価ガイド（コンテナ優先）

## 前提

- **移行志向**: クラウド移行（オンプレミスからの脱却）
- **コンテナ優先**: AKS → App Service → VM の優先順位
- **目的**: 各アプリに対して 6R（Rehost / Replatform / Refactor / Repurchase / Retire / Retain）を判断するための材料を収集する

---

## 6R とは

| R | 内容 |
|---|------|
| **Rehost** | 変更なしで Azure VM（IaaS）へ Lift & Shift |
| **Replatform** | 最小限の変更で PaaS / コンテナへ移行 |
| **Refactor** | アーキテクチャ・コードを再設計して移行 |
| **Repurchase** | SaaS 製品へ切り替え |
| **Retire** | 廃止 |
| **Retain** | 現状維持（移行しない） |

> **重要**: 6R のラベルはツールが自動で貼るものではない。ツールが収集した判断材料をもとに**人間が最終決定する**。

---

## コンテナ優先フローの特徴

App Service 優先フローとの最大の違いは**AppCAT が起点**になる点。

- Azure Migrate Assessment は「コンテナ化できるか」を判定できない
- コンテナ化の可否はコードの依存関係（ステートレスか、ローカルファイル依存があるか等）で決まる
- そのため **AppCAT を先に実行してコンテナ化可否を判断し、その後 Assessment でターゲットを確定する**

---

## 使用ツールと役割

| ツール | 役割 | 出力 |
|--------|------|------|
| **Azure Migrate Discovery** | 対象アプリの棚卸し・資産台帳の確定 | サーバー・アプリ一覧、ランタイム、依存関係マップ |
| **AppCAT**（Java / .NET） | ソースコードの静的解析 | 互換性問題件数、ステートフル依存・ファイル依存の有無 |
| **Azure Migrate Assessment** | 指定ターゲットへの移行可否・コスト評価 | Readiness、月額コスト |

---

## 評価フロー

### ステップ概要

```
① Discovery（棚卸し）
        ↓
② Discovery 結果でフィルタリング（AppCAT 実行対象を絞り込み）
        ↓
③ AppCAT（絞り込まれた対象のみ実行）
        ↓
④ AppCAT 結果でコンテナ化可否を振り分け
        ↓
⑤ Assessment（振り分け後のターゲットで実行）
        ↓
⑥ 6R 判定（人間）
```

---

### ① Discovery（Azure Migrate）

**目的**: 評価対象の確定

- Azure Migrate Appliance をオンプレミスに展開
- サーバー・アプリ・ランタイム・依存関係を自動収集
- **この時点では Readiness は出ない（棚卸しのみ）**
- ここで「何本を評価するか」のスコープが確定する

---

### ② Discovery 結果による AppCAT 実行前フィルタリング

**目的**: AppCAT の実行対象を絞り込み、実行コストを最小化する

全件に AppCAT を実行するのは非効率。Discovery で得られたランタイム・依存関係情報をフィルターとして使い、**コンテナ化が明らかに不可能なアプリを事前除外する**。

#### フィルタリング基準（Discovery 結果から判断）

| Discovery で検出された技術 | AppCAT 実行 | 理由・次のアクション |
|--------------------------|------------|------------------|
| Spring Boot / .NET Core / .NET 6+ | **実行** | コンテナ化しやすい、AppCAT で詳細確認 |
| .NET Framework 1.x / 2.x | **スキップ** | サポート外 → Retain / Refactor 確定 |
| Windows 認証・IIS 依存が明確 | **スキップ** | Linux コンテナ不可 → App Service（Windows）か VM へ |
| COM+ / DCOM / Windows サービス | **スキップ** | コンテナ化不可 → VM 確定 |
| レガシー Java EE（EJB 明記） | **スキップ** | 大規模 Refactor 確定 |

#### 依存関係マップによる絞り込み

- **依存が少ない・孤立しているアプリ** → コンテナ化しやすい → AppCAT 優先実行
- **依存が複雑・他システムと密結合** → コンテナ化困難 → 後回しまたは App Service フローへ

#### ビジネス優先度による絞り込み

- 移行優先度が高いアプリだけ AppCAT を先行実行
- 優先度低いアプリは App Service 優先フローに切り替え

---

### ③ AppCAT 実行（フィルタリング後の対象のみ）

**目的**: コンテナ化可否の判断材料を収集

AppCAT で確認するポイント：

| 確認項目 | コンテナ化への影響 |
|---------|-----------------|
| ステートフルな処理（セッション・インメモリ状態） | 外部化が必要 |
| ローカルファイル依存（書き込み・一時ファイル） | PVC 対応または設計変更が必要 |
| Windows 固有 API（COM・レジストリ・Win32） | Linux コンテナでは動作不可 |
| 非セキュアプロトコル・ハードコード URL | コンテナ化前に修正が必要 |
| EJB・RMI 等のレガシー依存 | コンテナ化困難、改修規模が大きい |

> **補足：動作 OS が Windows でも Linux コンテナで動く場合がある**  
> 判断基準は「動作 OS が Windows か否か」ではなく「アプリが Windows 固有 API に依存しているか否か」。  
> .NET Core / .NET 5 以降 / Java / Node 等は OS が Windows でも Linux コンテナで動作する。  
> .NET Framework（1.x〜4.x）は Linux 非対応のためコンテナ化不可。AppCAT の検出結果で判断する。

---

### ④ AppCAT 結果による振り分け

| AppCAT の結果 | 次のアクション | 判定候補 |
|-------------|-------------|---------|
| 依存なし・問題少ない | AKS Assessment → **コンテナ化（Replatform）** | AKS |
| 依存あり・修正可能 | App Service Assessment → **PaaS（Replatform）** | App Service |
| 依存が多い・修正規模大 | — | **Refactor** |
| Windows 固有 API 多数 | VM Assessment → **Rehost** | VM |

> **補足：「Windows 固有 API なし・.NET Framework のみ」の場合**  
> .NET Framework 自体が Linux 非対応のため AKS（Linux）は不可。  
> App Service の Windows プランであれば移行可能なため、App Service Assessment へ進む。  
> .NET Framework → .NET 8 への移行（Refactor）を行えば AKS も選択肢になる。

---

### ⑤ Assessment（Azure Migrate）

**目的**: 振り分け後のターゲットでの移行可否・コスト確認

| 対象 | 実行する Assessment | 確認内容 |
|------|-------------------|---------|
| コンテナ化可能グループ | AKS Assessment | リソースサイジング・コスト |
| PaaS グループ | App Service Assessment | Readiness（Ready / Ready with conditions / Not ready） |
| Rehost グループ | VM Assessment | Readiness・コスト |

App Service Assessment で **Ready with conditions** が出た場合：
- AppCAT は既に実行済みのため、問題の内容と件数を再確認
- 問題少 → **Replatform** 確定
- 問題多 → **Refactor** に変更

---

## 最終判定マトリクス（コンテナ優先）

| AppCAT 結果 | Assessment 結果 | 6R 判定 |
|-----------|---------------|--------|
| 依存なし・問題少 | AKS：Ready | **Replatform**（AKS） |
| 依存あり・修正可能 | App Service：Ready | **Replatform**（App Service） |
| 依存あり・修正可能 | App Service：Ready with conditions・問題少 | **Replatform**（App Service） |
| 依存が多い | App Service：Ready with conditions・問題多 | **Refactor** |
| Windows 固有 API 多数 | VM：Ready | **Rehost**（VM） |
| 依存が致命的 | VM：Not ready | **Retain** または **Refactor** 検討 |
| 使用率ゼロ | — | **Retire** |
| SaaS 代替可能 | — | **Repurchase** |

> Retire・Repurchase は Discovery の使用率データや業務要件をもとに人間が判断する。ツールによる自動判定はない。

---

## App Service 優先フローとの比較

| 項目 | App Service 優先 | コンテナ優先 |
|------|----------------|-----------|
| 起点 | Azure Migrate Assessment | **AppCAT** |
| AppCAT の実行対象 | Ready with conditions のみ | **全件（または絞り込み）** |
| Assessment の実行順 | App Service → VM（Not ready のみ） | AppCAT 結果でターゲットを決めてから |
| AKS Assessment | 後続のコスト確認のみ | **一次評価として実行** |
| 向いているケース | 大規模ポートフォリオ、早期スクリーニング | コンテナ化を本気で検討している場合 |

---

## 参考

- [AppCAT CLI の使い方](AppCAT.md)
- [GitHub Copilot Modernization Agent の使い方](GHCP_Mod.md)
- [推奨モダン化ワークフロー](recommended_workflow.md)
- [6R 評価ガイド（App Service 優先）](6R評価_AppService.md)
