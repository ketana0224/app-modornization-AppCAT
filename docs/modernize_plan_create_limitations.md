# `modernize plan create` CLI の設計上の制限

## 現象

`modernize plan create` で生成される `plan.md` に、AppCAT が検出した問題（非セキュアプロトコル・ハードコード URL 等）が含まれないことがある。

Spring WebFlow での実測例:

| AppCAT 検出（issue-only: 26件） | plan.md への反映 |
|-------------------------------|----------------|
| 非セキュアプロトコル（mandatory: 6件） | ❌ 未記載 |
| ハードコード URL（optional: 19件） | ❌ 未記載 |
| Ant ビルドツール（optional: 1件） | ❌ 未記載 |
| deprecated Spring EL API | ✅ 記載あり（AI が独自検出） |
| ロギング設定（Azure Monitor対応） | ✅ 記載あり（AI が独自検出） |

---

## 原因：CLI と VS Code 拡張の設計思想の違い

公式ドキュメント（[Microsoft Learn](https://learn.microsoft.com/en-us/azure/developer/java/migration/migrate-github-copilot-app-modernization-for-java-quickstart-assess-migrate)）が想定する正規フローは **VS Code 拡張 UI 経由**:

```
VS Code 拡張の Assessment Report を開く
  ↓
AppCAT が検出した issue を1つ選択
  ↓
「Run Task」をクリック → 選択した issue に対応する predefined task が動く
  ↓
plan.md + progress.md が issue 起点で生成される
```

一方、CLI の `modernize plan create <prompt>` は**フリーフォームのプロンプトから AI がソースを独自解析**してプランを生成する別物。

| 項目 | CLI (`modernize plan create`) | VS Code 拡張（正規フロー） |
|------|-------------------------------|--------------------------|
| 入力 | 自由文プロンプト + AI がソース解析 | AppCAT の issue 1件を選択 |
| plan の根拠 | AI の独自判断 | AppCAT の検出結果 |
| AppCAT との連携 | **なし** | **あり**（issue → task に直結） |

---

## 対処方法

### 方法 A: VS Code 拡張 UI を使う（正規フロー）

Assessment Report の issue を1件ずつ選択して「Run Task」を実行する。  
AppCAT の検出結果と plan が確実に連動する。

### 方法 B: tasks.json に手動でタスクを追加する

`modernize plan create` で雛形を生成後、AppCAT が検出した問題を `tasks.json` に手動追記する。

```json
{
  "type": "transform",
  "id": "003-transform-secure-protocol",
  "description": "非セキュアプロトコル（http://）を https:// に置換する",
  "requirements": "AppCAT ルール unsecure-network-protocol-00000 で検出された6箇所を修正。...",
  "skills": [],
  "successCriteria": {
    "passBuild": "true",
    "passUnitTests": "true"
  }
}
```

### 方法 C: プロンプトに AppCAT の問題を明示する

```powershell
modernize plan create `
  "Migrate to Azure. Fix the following AppCAT findings: 
   1. Replace http:// with https:// in source files (6 incidents, mandatory).
   2. Externalize hardcoded URLs to environment variables (19 incidents, optional)." `
  --language java --overwrite
```

ただし AI の解釈次第のため、確実性は方法 A・B より低い。

---

## 背景・ステータス

- **バグではなく設計上の制限**: CLI は VS Code 拡張の UI フローを代替するものではない
- **Public Preview 中**（2025年5月〜）のため、CLI と VS Code 拡張の機能差は今後縮まる可能性がある
- **predefined task** の概念: 公式が用意した移行シナリオ（Azure SQL + Managed Identity 等）向けのタスクが存在するが、非セキュアプロトコルやハードコード URL には該当 predefined task がない
- 参考: [GitHub Community Discussion #159076](https://github.com/orgs/community/discussions/159076)
