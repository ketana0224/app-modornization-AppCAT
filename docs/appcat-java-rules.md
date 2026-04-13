# appcat-java ルール一覧

AppCAT Java版（`appcat` v7.7.0.9）が使用するルールの一覧。  
ルール定義は `~/.appcat/rulesets/` 配下に YAML ファイルとして格納されており、`appcat analyze` 実行時に自動的に読み込まれる。

---

## Severity（深刻度）の定義

| 値 | 意味 |
|---|---|
| `mandatory` | Azure / 移行先への対応が必須な問題 |
| `potential` | 対応が必要になる可能性がある問題 |
| `optional` | 対応を検討すべきだが必須ではない |
| `information` | 情報提供のみ |

## Effort（対応コスト）の定義

対応に必要なストーリーポイントの目安。値が大きいほど対応コストが高い。

| 値 | 目安 |
|---|---|
| `1` | 設定変更・コメント対応のみ |
| `2〜3` | コード修正が必要 |
| `5` | アーキテクチャ変更が必要 |
| `8〜13` | 大規模な設計変更が必要 |

---

## フォルダ別ルール数サマリ

| フォルダ | ルールファイル数 | 主な対象 |
|---------|--------------|---------|
| `azure/` | 36 | **Azure 移行固有ルール**（AWS/GCP/キャッシュ/DB/メッセージキュー等） |
| `cloud-readiness/` | 23 | クラウド汎用ルール（ロギング・セッション・RMI等） |
| `spring-boot/` | 8 | Spring Boot 2.x → 3.0 移行 |
| `spring-framework/` | 7 | Spring Framework 5.x → 6.0 移行 |
| `quarkus/` | 31 | Spring Boot → Quarkus 移行 |
| `jakarta-ee/` | 12 | Jakarta EE 移行（JBoss/WebLogic/WebSphere） |
| `jakarta-ee9/` | 1 | Jakarta EE 9 互換性 |
| `eap6/` | 21 | JBoss EAP 6 → 7 移行 |
| `eap7/` | 47 | JBoss EAP 7 → 8 移行 |
| `eap8/` | 28 | JBoss EAP 8 移行 |
| `openjdk11/` | 27 | OpenJDK 8 → 11 の非推奨・削除API |
| `openjdk17/` | 19 | OpenJDK 11 → 17 の非推奨・削除API |
| `openjdk21/` | 22 | OpenJDK 17 → 21 の非推奨・削除API |
| `openjdk7/` | 1 | OracleJDK → OpenJDK 7 |
| `openjdk8/` | 10 | OpenJDK 8 の非推奨API |
| `camel3/` | 32 | Apache Camel 2 → 3 移行 |
| `camel4/` | 2 | Apache Camel 3 → 4 移行 |
| `hibernate/` | 3 | Hibernate 4 移行 |
| `droolsjbpm/` | 1 | Drools / jBPM → KIE |
| `eapxp/` | 7 | EAP XP（Thorntail → EAP XP） |
| `eapxp6/` | 1 | EAP XP 5 → 6 |
| `fuse/` | 3 | Sonic ESB → Apache Fuse |
| `fuse-service-works/` | 1 | JBoss SOA Platform 5 |
| `jws6/` | 1 | JWS（Tomcat）バージョン更新 |
| `openliberty/` | 2 | WebSphere → Open Liberty |
| `os/` | 1 | Windows OS 固有の依存 |
| `rhr/` | 2 | Red Hat Runtime Spring Boot バージョン検証 |
| `technology-usage/` | 50 | 使用技術の検出・分類（スコアリングなし） |
| `00-discovery/` | 1 | ライセンスファイル検出 |
| `filemappings/` | 2 | ファイルパターンマッチ設定 |

---

## 詳細ルール一覧

### azure/（Azure 移行固有ルール）

Azure への移行に最も直接関係するルール群。`appcat analyze --target azure-appservice` 等で主に使用される。

| ルールID | 説明 | Severity | Effort |
|---------|------|----------|--------|
| azure-aws-config-credential-01000 | AWS credential configuration | mandatory | 8 |
| azure-aws-config-region-02000 | AWS region configuration | mandatory | 2 |
| azure-aws-config-s3-03000 | AWS S3 usage found | mandatory | 8 |
| azure-aws-config-s3-03001 | AWS S3 dependency usage found | mandatory | 8 |
| azure-aws-config-s3-03002 | AWS S3 operations found | mandatory | 8 |
| azure-aws-config-sqs-04000 | Amazon SQS Java 1.x SDK dependency | mandatory | 2 |
| azure-aws-config-sqs-04001 | Amazon SQS Java 1.x API | mandatory | 8 |
| azure-aws-config-sqs-04002 | Amazon SQS Java 2.x SDK dependency | mandatory | 2 |
| azure-aws-config-sqs-04003 | Amazon SQS Java 2.x API | mandatory | 8 |
| azure-aws-config-sqs-04004 | Amazon SQS configuration | mandatory | 2 |
| azure-aws-config-secret-manager-05000 | AWS Secrets Manager configuration | mandatory | 8 |
| aws-lambda-to-azure-functions-01000 | AWS Lambda usage detected | optional | 13 |
| azure-cache-redis-01000 | Redis Cache connection string found | potential | 5 |
| azure-database-config-mongodb-02000 | MongoDB connection found in configuration file | potential | 5 |
| azure-file-system-02000 | Relative path found | optional | 5 |
| azure-file-system-03000 | Home path found | optional | 5 |
| azure-java-version-01000 | Non-LTS Java version | mandatory | 5 |
| azure-java-version-02000 | Legacy Java version | mandatory | 5 |
| azure-keystore-certificates-01000 | Java KeyStore file found | potential | 5 |
| azure-keystore-certificates-02000 | The application loads certificates into a KeyStore | potential | 5 |
| azure-message-queue-config-kafka-01000 | Kafka configuration found | optional | 5 |
| azure-message-queue-config-rabbitmq-01000 | RabbitMQ connection string/username/password found | optional | 5 |
| azure-message-queue-config-artemis-01000 | ActiveMQ Artemis connection string found | optional | 5 |
| azure-message-queue-rabbitmq-01000 | Spring RabbitMQ usage found | optional | 5 |
| azure-message-queue-amqp-02000 | Spring AMQP dependency found | optional | 5 |
| azure-message-queue-spring-jms-rabbitmq-01000 | Spring JMS RabbitMQ dependency found | optional | 5 |
| azure-message-queue-activemq-01000 | ActiveMQ found | potential | 5 |
| azure-message-queue-java-ee-rabbitmq-amqp-01000 | Java EE/Jakarta EE application using RabbitMQ AMQP | optional | 5 |
| azure-message-queue-ibm-jms-01000 | IBM MQ usage detected | optional | 8 |
| amazon-sns-to-azure-servicebus-01000 | Amazon SNS usage detected | optional | 8 |
| tibco-ems-jms-to-azure-servicebus-jms-01000 | TIBCO EMS JMS usage detected | optional | 8 |
| solace-pubsubplus-to-azure-servicebus-01000 | Solace PubSub+ usage detected | optional | 8 |
| amazon-kinesis-to-azure-eventhubs-01000 | Amazon Kinesis usage detected | optional | 8 |
| apache-pulsar-to-azure-eventhubs-01000 | Apache Pulsar usage detected | optional | 8 |
| azure-password-01000 | Password found in configuration file | potential | 3 |
| azure-static-content-01000 | Static content found in the application | optional | 2 |
| azure-system-config-01000 | Environment variables/system properties | optional | 3 |
| azure-tas-binding-01000 | Tanzu Application Service service bindings | potential | 5 |
| eap-to-azure-appservice-datasource-driver-01000 | Datasource driver found in configuration file | potential | 5 |
| eap-to-azure-appservice-pom-001 | Get started with JBoss EAP on App Service | optional | 2 |
| jetty-to-azure-external-resources-01000 | External resources found in Jetty configuration file | optional | 3 |
| spring-boot-to-azure-config-server-01000 | Spring Cloud Config usage detected | optional | 8 |
| spring-boot-to-azure-eureka-01000 | Explicit eureka connection info found | potential | 2 |
| spring-boot-to-azure-eureka-02000 | Embedded framework - Eureka Client | potential | 2 |
| spring-boot-to-azure-eureka-03000 | Embedded framework - Eureka Server | potential | 5 |
| spring-boot-to-azure-port-01000 | Server port configuration found | potential | 1 |
| spring-boot-to-azure-restricted-config-01000 | Restricted configurations found | potential | 2 |
| quartz-scheduler-to-azure-functions-01000 | Quartz Scheduler usage detected | optional | 13 |
| spring-batch-to-azure-durable-functions-01000 | Spring Batch usage detected | optional | 13 |
| spring-boot-to-azure-spring-boot-version-01000 | Spring Boot open source support version | mandatory | 5 |
| spring-boot-to-azure-spring-cloud-version-01000 | Spring Cloud version compatibility | mandatory | 5 |

### cloud-readiness/（クラウド汎用ルール）

クラウド環境全般での問題を検出するルール群。

| 主な検出内容 | ルール数 | Severity |
|------------|---------|----------|
| ロギングの設定（ファイル出力→クラウドストリーミングへ） | 複数 | mandatory |
| HTTP セッション管理（スティッキーセッション依存） | 複数 | mandatory |
| RMI / JNDI 使用 | 複数 | mandatory |
| CORBA 使用 | 複数 | mandatory |
| Java EE 組み込みキャッシュ | 複数 | potential |
| ローカルファイル IO | 複数 | mandatory |
| 共有メモリ・シグナル | 複数 | mandatory |

### spring-boot/（Spring Boot 移行）

Spring Boot 2.x → 3.0 移行ルール。

| ルールID | 説明 | Severity | Effort |
|---------|------|----------|--------|
| spring-boot-2.x-to-3.0-batch-00000 | @EnableBatchProcessing not needed anymore | optional | 1 |
| spring-boot-2.x-to-3.0-core-changes-00001 | Image banner support removed | mandatory | 1 |
| spring-boot-2.x-to-3.0-datasource-00000 | Spring data properties | potential | 1 |
| spring-boot-2.x-to-3.0-dependencies-00001 | Spring Boot 3.0 must use at least spring-cloud-bus 4.0.x | mandatory | 5 |
| spring-boot-2.x-to-3.0-removals-00010 | AbstractDataSourceInitializer has been removed | mandatory | 3 |
| spring-boot-2.x-to-3.0-security-00000 | ReactiveUserDetailsService is no longer auto-configured | mandatory | 3 |
| spring-boot-2.x-to-3.0-session-00000 | Store type explicit config no longer supported | mandatory | 1 |
| spring-boot-2.x-to-3.0-webapp-changes-00000 | server.max-http-header-size has been deprecated | optional | 1 |

### spring-framework/（Spring Framework 移行）

Spring Framework 5.x → 6.0 移行ルール。

| ルールID | 説明 | Severity | Effort |
|---------|------|----------|--------|
| spring-framework-5.x-to-6.0-baseline-00001 | Spring Framework 6.0 must use at least Java 17 | mandatory | 1 |
| spring-framework-5.x-to-6.0-core-container-00001 | Spring Framework 6.0 does not use Introspector anymore | potential | 1 |
| spring-framework-5.x-to-6.0-data-access-00001 | Spring 6 must use at least Hibernate 5.6.x | mandatory | 5 |
| spring-framework-5.x-to-6.0-removed-apis-00001 | Spring Framework 6.0 has dropped support for EhCache 2.x | mandatory | 5 |
| spring-framework-5.x-to-6.0-security-deprecations-00000 | Use the new setClockSkew methods | mandatory | 1 |
| spring-framework-5.x-to-6.0-security-00001 | WebSecurityConfigurerAdapter has been removed in Spring Security 6.0 | mandatory | 3 |
| spring-framework-5.x-to-6.0-web-applications-00001 | Trailing slash matching has changed in Spring 6.0 | potential | 1 |

### jakarta-ee/（Jakarta EE 移行）

JBoss EAP / WebLogic / WebSphere → Azure のマイグレーションルール。

| 主なルールID | 説明 | Severity | Effort |
|------------|------|----------|--------|
| jboss-eap-to-azure-app-service | Migrate JBoss EAP → JBoss EAP on Azure App Service | mandatory | 5 |
| jboss-eap-to-aks | Migrate JBoss EAP → Azure Kubernetes Service | mandatory | 8 |
| jboss-eap-to-azure-container-apps | Migrate JBoss EAP → Azure Container Apps | mandatory | 8 |
| weblogic-to-azure-app-service | Migrate WebLogic → JBoss EAP on Azure App Service | mandatory | 5 |
| weblogic-to-aks | Migrate WebLogic → Azure Kubernetes Service | mandatory | 8 |
| websphere-to-azure-app-service | Migrate WebSphere → JBoss EAP on Azure App Service | mandatory | 5 |
| websphere-to-aks | Migrate WebSphere → Azure Kubernetes Service | mandatory | 8 |
| jboss-dependencies-00006 | Legacy javax.* spec artifacts detected | mandatory | 5 |
| seam-java-00010〜00080 | Seam API 移行（6ルール） | mandatory | 3〜5 |

### openjdk11/（OpenJDK 8 → 11）

OpenJDK 8 から 11 へのアップグレードで問題となる API の検出（27ルール）。

| 主な検出内容 | 例 |
|------------|---|
| Java EE モジュール削除 | java.activation, java.annotation の削除 |
| CORBA モジュール削除 | JEP 320 |
| sun.misc.Unsafe 廃止API | defineClass の削除 |
| SecurityManager 廃止API | checkMemberAccess 等 |
| JavaFX Builder クラス削除 | |
| Thread.destroy / Thread.stop(Throwable) 削除 | |
| java 9〜11 の deprecate/removal | 計27ルール（全て mandatory, effort 1〜5） |

### openjdk17/（OpenJDK 11 → 17）

OpenJDK 11 から 17 へのアップグレードで問題となる API の検出（19ルール）。

| 主な検出内容 | 例 |
|------------|---|
| Security Manager 廃止 | java.lang.SecurityManager deprecated |
| Pack200 ツール削除 | JEP 367 |
| Applet API 廃止 | JEP 398 |
| RMI Activation 削除 | JEP 407 |
| Nashorn JavaScript エンジン削除 | JEP 372 |
| Thread Suspend/Resume/Stop の廃止 || 

### openjdk21/（OpenJDK 17 → 21）

OpenJDK 17 から 21 へのアップグレードで問題となる API の検出（22ルール）。

| 主な検出内容 | 例 |
|------------|---|
| Thread.stop → UnsupportedOperationException | mandatory |
| ThreadGroup の機能縮小 | mandatory |
| java.net.URL コンストラクタ廃止 | mandatory |
| Finalization の廃止 | mandatory |
| UTF-8 デフォルト化 | java.io / java.util / URLEncoder 等 |

### quarkus/（Spring Boot → Quarkus）

Spring Boot アプリを Quarkus へ移行するためのルール（31ルール）。Azure Container 環境への最適化を目的とする。

| 主なルールID | 説明 | Severity | Effort |
|------------|------|----------|--------|
| springboot-parent-pom-to-quarkus-00000 | Replace Spring Parent POM with Quarkus BOM | mandatory | 1 |
| springboot-di-to-quarkus-00000 | Replace Spring DI artifact | potential | 1 |
| springboot-jpa-to-quarkus-00000 | Replace Spring Data JPA artifact | mandatory | 1 |
| springboot-security-to-quarkus-00000 | Replace Spring Security artifact | mandatory | 1 |
| springboot-webmvc-to-quarkus-00000 | Spring MVC is not supported by Quarkus | mandatory | 13 |
| springboot-jmx-to-quarkus-00000 | Spring JMX is not supported (GraalVM Native) | mandatory | 13 |
| jms-to-reactive-quarkus-00000 | JMS is not supported in Quarkus | mandatory | 5 |

### eap8/（JBoss EAP 8 移行）

Java EE → Jakarta EE 名前空間変更が中心（28ルール）。`javax.*` から `jakarta.*` への変更。

| 主な検出内容 | ルール数 |
|------------|---------|
| javax.* → jakarta.* 名前空間変換 | 約20ルール（全て mandatory, effort 1） |
| jboss 依存関係の更新 | 約7ルール |
| Log4j v1 削除 | 5ルール |

### eap7/（JBoss EAP 7 移行、47ルール）

EAP 6 → EAP 7 の大規模移行ルール。MDB、EJB、JAX-RS、JNDI 等。

### eap6/（JBoss EAP 6 移行、21ルール）

Seam 2 UI コンポーネントの移行が中心。EJB 3.0、JBoss固有APIの検出。

### camel3/（Apache Camel 2 → 3、32ルール）

pom.xml の依存関係変更・API変更を検出。ComponentBindingRegistry、CamelContext等。

### openliberty/（WebSphere → Open Liberty）

IBM WebSphere 固有API の Open Liberty での利用可否チェック（50以上のルール）。

| 主な検出内容 | Severity |
|------------|----------|
| WebSphere 独自スケジューラ API | mandatory |
| WebSphere セキュリティ API | mandatory |
| WebSphere 管理 API | mandatory |
| Entity EJB | mandatory |
| OSGI リモートサービス | mandatory |

### os/（Windows OS 依存）

| ルールID | 説明 | Severity | Effort |
|---------|------|----------|--------|
| os-specific-00001 | Windows file system path detected | mandatory | 3 |
| os-specific-00002 | Dynamic-Link Library (DLL) usage detected | mandatory | 5 |

### technology-usage/（技術スタック検出）

Severity・Effort なし（スコアリング対象外）。アプリが使用している技術スタックの把握目的。

| 検出カテゴリ | 例 |
|------------|---|
| フレームワーク | Spring Boot, Spring, Apache Camel, Apache Axis |
| ロギング | Log4j, Spring Boot Actuator |
| テスト | EasyMock |
| データベース | JDBC datasources, HSQLDB |
| メッセージング | JTA, EJB Timer |
| UI | JSF, Apache Wicket |
| 構成管理 | Spring Cloud Config |
| APM | Application Insights |
| セキュリティ | Spring Security |
| クラスタリング | Clustering Web Session |

---

## --target オプションとルールの適用範囲

`appcat analyze --target <TARGET>` で指定するターゲットに応じて、適用されるルールが変わる。

| ターゲット | 主な用途 |
|-----------|---------|
| `azure-appservice` | Azure App Service (Linux/Windows) への移行 |
| `azure-aks` | Azure Kubernetes Service への移行 |
| `azure-container-apps` | Azure Container Apps への移行 |
| `quarkus` | Quarkus への移行 |
| `openjdk11` / `openjdk17` / `openjdk21` | OpenJDK バージョンアップ |
| `eap7` / `eap8` | JBoss EAP バージョンアップ |

> ターゲットを省略した場合、すべてのルールが適用される。Azure 移行評価では `--target azure-appservice` または `--target azure-aks` の指定を推奨。

---

## .NET版との比較

| | Java版 (`appcat`) | .NET版 (`dotnet-appcat`) |
|--|------|------|
| ルール形式 | YAML ファイル（`~/.appcat/rulesets/`） | DLL 埋め込み JSON |
| 総ルール数 | 400件超 | 約73件 |
| Azure 移行ルール | `azure/` 36ファイル（50+ルール） | Security, IIS, Local 等に分散 |
| カスタマイズ | `--rules` でYAMLを追加 | `--config` でJSONを追加 |
| 対象フレームワーク | Spring, Jakarta EE, Camel, Quarkus等 | .NET Framework, ASP.NET |

---

## 参考リンク

- [AppCAT Java CLI GitHub (Konveyor)](https://github.com/konveyor/kantra)
- [AppCAT ルールセット (analyzer-lsp YAML 形式)](https://github.com/konveyor/rulesets)
- [Azure Migrate AppCAT ドキュメント](https://learn.microsoft.com/ja-jp/azure/migrate/appcat/java)
