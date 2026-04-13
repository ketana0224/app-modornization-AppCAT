# dotnet-appcat ルール一覧

AppCAT .NET版（`dotnet-appcat` v1.0.878）が使用するルールの一覧。  
ルール定義は `Microsoft.AppCAT.dll` に JSON として埋め込まれており、ファイルとして外部に存在しない。

---

## Severity（深刻度）の定義

| 値 | 意味 |
|---|---|
| `mandatory` | Azure 移行において必ず対応が必要な問題 |
| `potential` | 対応が必要になる可能性がある問題 |
| `optional` | 対応を検討すべきだが必須ではない |
| `information` | 情報提供のみ（移行への直接的な影響は小） |

## Effort（対応コスト）の定義

| 値 | 目安 |
|---|---|
| `1` | 低（設定変更・確認のみ） |
| `3` | 中（コード修正が必要） |
| `5` | 高（アーキテクチャ変更が必要） |

---

## ルール一覧

### Cache（キャッシュ）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Cache.0001 | Data caching is detected | potential | 3 |

### COM

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| COM.0001 | COM usage detected | mandatory | 5 |

### Database（データベース・ストレージ）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Connection.0001 | Connection string is detected | potential | 3 |
| Connection.0002 | Environment variables dependency detected | potential | 3 |
| Database.0001 | Database dependency detected | potential | 5 |
| Database.0002 | SQL database connection detected | potential | 3 |
| Database.0003 | System.Data.SqlClient dependency detected | optional | 3 |
| Database.0101 | Amazon S3 dependency detected | potential | 5 |
| Database.0102 | Oracle dependency detected | potential | 5 |
| Database.0103 | DB2 dependency detected | potential | 5 |
| Database.0104 | Minio dependency detected | potential | 5 |
| Database.0106 | Google Cloud Storage dependency detected | potential | 5 |

### Dependencies（クラウド間依存）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| CrossCloud.0001 | Dependency on cloud services outside of Azure is detected | potential | 3 |

### HTTP（通信）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| HTTP.0001 | Access to external resources via HTTP is detected | potential | 3 |
| HTTP.0002 | Non HTTP(s) communication detected | mandatory | 5 |

### Identity（認証）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Identity.0001 | Unsupported authentication type detected | mandatory | 3 |
| Identity.0002 | Windows authentication detected | mandatory | 3 |

### IIS（IIS 構成）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| IIS.0001 | Application Request Routing is detected | mandatory | 1 |
| IIS.0002 | ISAPI filters detected | potential | 1 |
| IIS.0003 | IIS HTTP logging settings are modified | mandatory | 1 |
| IIS.0004 | HTTPS protocol is detected | optional | 1 |
| IIS.0005 | Unsupported protocols detected | information | 1 |
| IIS.0006 | Unsupported ports detected | information | 1 |
| IIS.0007 | More than one application pool was detected for IIS site | information | 1 |
| IIS.0008 | UNC share is used as application physical path | information | 1 |
| IIS.0009 | The site content is greater than 2 GB | information | 3 |
| IIS.0010 | Custom location tags detected | information | 1 |
| IIS.0011 | Unsupported global modules detected | optional | 3 |
| IIS.0012 | Unsupported application pool identities detected | optional | 1 |
| IIS.0013 | Unsupported ISAPI filters detected | optional | 3 |
| IIS.0014 | Application pool with unsupported .NET Framework detected | mandatory | - |
| IIS.0015 | Potential non-.NET Framework dependency detected | optional | - |
| IIS.0016 | IIS Server configuration error | mandatory | - |

### Local（ローカル OS 依存）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Local.0001 | Windows local system features usage detected | mandatory | 3 |
| Local.0002 | Hardcoded local or network paths detected | mandatory | 1 |
| Local.0003 | Local or network IO operations detected | potential | 3 |
| Local.0004 | Logging to local or network paths detected | mandatory | 3 |
| Local.0005 | Windows profile APIs usage detected | mandatory | 3 |
| Local.0006 | Hardcoded URLs detected | potential | 1 |
| Local.0007 | Local OS environment access detected | potential | 1 |
| Local.0008 | User32/GDI32 system calls detected | mandatory | 3 |
| Local.0009 | Native API calls detected | optional | 3 |
| Local.0010 | Service control manager API calls detected | mandatory | 3 |
| Local.0011 | Windows Management Instrumentation (WMI) API calls detected | mandatory | 3 |
| Local.0012 | Local process start detected | optional | 5 |
| Local.0013 | Shared job storage provider detected | potential | 3 |
| Local.0014 | Writing or modifying diagnostics in Windows Event Log detected | optional | 3 |
| Local.0015 | Writing to Event Tracing for Windows (ETW) detected | optional | 3 |
| Local.0016 | Crystal Reports dependency detected | optional | 5 |
| Local.0018 | Assembly references from GAC are detected | potential | 1 |
| Local.0019 | Directory services (LDAP) access is detected | mandatory | 3 |
| Local.0020 | Named pipes communication is detected | potential | 3 |
| Local.0021 | Detected loading of dynamic assemblies | potential | 3 |

### Perf（パフォーマンス）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Perf.0001 | Synchronous API usage detected | optional | 1 |

### Queue（メッセージキュー）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Queue.0001 | RabbitMQ usage is detected | mandatory | 3 |
| Queue.0002 | Service Bus dependency is detected | potential | 3 |
| Queue.0003 | MSMQ usage is detected | mandatory | 3 |
| Queue.0101 | Local kafka dependency detected | potential | 3 |
| Queue.0102 | Amazon Simple Queue Service (SQS) dependency detected | potential | 5 |

### Runtime（.NET フレームワーク）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Runtime.0001 | Out of support .NET target framework is detected | mandatory | 3 |
| Runtime.0002 | Upgrade to newer target framework to get better cloud experience | optional | 3 |
| Runtime.0003 | Old .NET Framework dependency detected | potential | 3 |

### Scale（スケーラビリティ）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Scale.0001 | Static content detected | optional | 3 |
| Scale.0002 | Session state stored in-proc or in local process is detected | potential | 3 |
| Scale.0003 | MachineKey dependency is detected | optional | 3 |
| Scale.0004 | Potential duplex communication is detected | optional | 3 |

### Security（セキュリティ）

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| Security.0001 | MD5/SHA1 usage detected | optional | 1 |
| Security.0002 | Connection strings without configuration builders detected | optional | 3 |
| Security.0003 | Hardcoded sensitive data detected | optional | 3 |
| Security.0004 | Explicit password-based credentials usage is detected | potential | 3 |
| Security.0102 | AWS Secrets Manager dependency detected | potential | 5 |
| Security.0103 | Certificate management dependency detected | potential | 5 |

### SMTP

| ルールID | ルール名 | Severity | Effort |
|---------|---------|----------|--------|
| SMTP.0001 | SMTP connections detected | potential | 3 |

---

## カテゴリ別サマリ

| カテゴリ | ルール数 | mandatory | potential | optional | information |
|---------|---------|-----------|-----------|----------|-------------|
| Cache | 1 | 0 | 1 | 0 | 0 |
| COM | 1 | 1 | 0 | 0 | 0 |
| Database | 10 | 0 | 9 | 1 | 0 |
| Dependencies | 1 | 0 | 1 | 0 | 0 |
| HTTP | 2 | 1 | 1 | 0 | 0 |
| Identity | 2 | 2 | 0 | 0 | 0 |
| IIS | 16 | 5 | 1 | 4 | 6 |
| Local | 20 | 7 | 7 | 6 | 0 |
| Perf | 1 | 0 | 0 | 1 | 0 |
| Queue | 5 | 2 | 3 | 0 | 0 |
| Runtime | 3 | 1 | 1 | 1 | 0 |
| Scale | 4 | 0 | 1 | 3 | 0 |
| Security | 6 | 0 | 3 | 3 | 0 |
| SMTP | 1 | 0 | 1 | 0 | 0 |
| **合計** | **73** | **19** | **29** | **19** | **6** |

---

## 備考

- ルールは `Microsoft.AppCAT.dll` 内に埋め込まれており、外部ファイルとして参照・編集はできない
- カスタムルールは `--config` オプションで JSON ファイルを指定して追加する
- ルールのローカライズ（日本語を含む11言語）も同 DLL に含まれる
- `--target` オプション（`AppService.Windows` / `AppService.Linux` / `AKS` 等）によって、適用されるルールのサブセットや重み付けが変わる

## 参考リンク

- [AppCAT .NET 公式ドキュメント](https://learn.microsoft.com/ja-jp/dotnet/azure/migration/appcat/app-code-assessment-toolkit)
- [AppCAT カスタムルール設定](https://learn.microsoft.com/ja-jp/dotnet/azure/migration/appcat/custom-rules)
