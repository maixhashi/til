# About aws_rds_cluster
<!-- このファイルはaws_rds_clusterに関する包括的な知識をまとめたものです -->
<!-- daily-TILから重要な内容を抽出・整理し、体系的にまとめています -->

> [!NOTE]
> このドキュメントはaws_rds_clusterに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [About aws_rds_cluster](#about-aws_rds_cluster)
  - [目次](#目次)
  - [概要](#概要)
    - [キーポイント](#キーポイント)
  - [What - aws_rds_clusterとは何か](#what---aws_rds_clusterとは何か)
    - [基本概念](#基本概念)
      - [定義](#定義)
      - [構成要素](#構成要素)
    - [主要な特徴](#主要な特徴)
    - [アーキテクチャ](#アーキテクチャ)
      - [レイヤー構成](#レイヤー構成)
      - [データフロー](#データフロー)
  - [Why - なぜaws_rds_clusterが必要なのか](#why---なぜaws_rds_clusterが必要なのか)
    - [解決する課題](#解決する課題)
      - [従来の問題点](#従来の問題点)
      - [aws_rds_clusterによる解決策](#aws_rds_clusterによる解決策)
    - [メリット](#メリット)
      - [ビジネス面のメリット](#ビジネス面のメリット)
      - [技術面のメリット](#技術面のメリット)
    - [デメリット](#デメリット)
    - [他の選択肢との比較](#他の選択肢との比較)
  - [How - aws_rds_clusterの実装方法](#how---aws_rds_clusterの実装方法)
    - [基本的な使い方](#基本的な使い方)
      - [セットアップ](#セットアップ)
      - [基本的な実装](#基本的な実装)
      - [実行例](#実行例)
    - [ベストプラクティス](#ベストプラクティス)
      - [1. セキュリティの強化](#1-セキュリティの強化)
      - [2. 高可用性の確保](#2-高可用性の確保)
      - [3. モニタリングの設定](#3-モニタリングの設定)
    - [よくある実装パターン](#よくある実装パターン)
      - [パターン1: 標準的な本番環境構成](#パターン1-標準的な本番環境構成)
      - [パターン2: 開発環境用最小構成](#パターン2-開発環境用最小構成)
      - [パターン3: サーバーレス構成](#パターン3-サーバーレス構成)
    - [トラブルシューティング](#トラブルシューティング)
      - [エラー1: InvalidDBClusterStateFault](#エラー1-invaliddbclusterstatefault)
      - [エラー2: DBSubnetGroupNotFoundFault](#エラー2-dbsubnetgroupnotfoundfault)
      - [エラー3: InsufficientDBClusterCapacityFault](#エラー3-insufficientdbclustercapacityfault)
  - [参照：daily-TIL](#参照daily-til)
    - [What関連](#what関連)
    - [Why関連](#why関連)
    - [How関連](#how関連)
  - [バージョン履歴](#バージョン履歴)

</details>

---

## 概要

aws_rds_clusterはTerraformでAmazon Aurora DBクラスターを作成・管理するためのリソースです。高可用性と自動スケーリングを備えたマネージドデータベースクラスターを構築し、MySQLやPostgreSQLと互換性のある高性能なデータベースサービスを提供します。

### キーポイント

- Amazon Auroraクラスターの宣言的な構成管理
- 自動フェイルオーバーと3AZレプリケーションによる高可用性
- ストレージの自動スケーリングとバックアップ管理

---

## What - aws_rds_clusterとは何か

### 基本概念

<details>
<summary>基本概念の詳細</summary>

aws_rds_clusterリソースは、Amazon Aurora DBクラスターをTerraformで管理するためのリソースタイプです。複数のDBインスタンスを管理するクラスターレベルのリソースとして、データベースエンジン、認証情報、バックアップ設定などを統合的に定義します。

#### 定義

aws_rds_clusterは、Amazon Auroraデータベースクラスターの設定と管理を行うTerraformリソースです。MySQLまたはPostgreSQLと互換性のあるフルマネージド型リレーショナルデータベースサービスを提供し、エンタープライズレベルの性能と可用性を実現します。

#### 構成要素

1. **クラスター**
   - データベースの論理的なグループ単位

2. **クラスターインスタンス**
   - 実際のコンピューティングリソース（別リソースで定義）

3. **ストレージレイヤー**
   - 自動スケーリングする共有ストレージ

</details>

### 主要な特徴

<details>
<summary>特徴の詳細</summary>

1. **高性能**
   - 標準的なMySQLの最大5倍、PostgreSQLの最大3倍のスループット
   - 利点: 大規模なワークロードへの対応

2. **高可用性**
   - 3つのAZに6つのデータコピーを自動複製
   - 利点: 30秒以内の自動フェイルオーバー

3. **自動管理機能**
   - ストレージの自動拡張（10GB〜128TB）
   - 利点: 運用負荷の大幅な削減

</details>

### アーキテクチャ

<details>
<summary>アーキテクチャ図と説明</summary>

```mermaid
graph TB
    subgraph "Aurora Cluster Architecture"
        subgraph "Cluster Level"
            Cluster[RDS Cluster]
            ClusterEndpoint[Cluster Endpoint<br/>(Read/Write)]
            ReaderEndpoint[Reader Endpoint<br/>(Read Only)]
        end
        
        subgraph "Instance Level"
            Primary[Primary Instance<br/>(Writer)]
            Replica1[Replica 1<br/>(Reader)]
            Replica2[Replica 2<br/>(Reader)]
        end
        
        subgraph "Storage Layer"
            Storage[Aurora Storage<br/>Auto-scaling: 10GB-128TB]
            
            subgraph "Multi-AZ Replication"
                AZ1[AZ-1<br/>2 copies]
                AZ2[AZ-2<br/>2 copies]
                AZ3[AZ-3<br/>2 copies]
            end
        end
        
        subgraph "Network Layer"
            SubnetGroup[DB Subnet Group]
            SG[Security Group]
        end
    end
    
    Client[Application] --> ClusterEndpoint
    Client --> ReaderEndpoint
    
    ClusterEndpoint --> Primary
    ReaderEndpoint --> Replica1
    ReaderEndpoint --> Replica2
    
    Primary --> Storage
    Replica1 --> Storage
    Replica2 --> Storage
    
    Storage --> AZ1
    Storage --> AZ2
    Storage --> AZ3
    
    Cluster --> SubnetGroup
    Cluster --> SG
    
    style Cluster fill:#FFE4B5
    style Primary fill:#90EE90
    style Storage fill:#87CEEB
```

#### レイヤー構成

- **クラスター層**: エンドポイントと全体設定の管理
- **インスタンス層**: コンピューティングリソースの提供
- **ストレージ層**: 自動レプリケーションと拡張

#### データフロー

1. アプリケーションがクラスターエンドポイントに接続
2. 書き込みはプライマリインスタンスで処理
3. データは6つのコピーに自動レプリケート

</details>

---

## Why - なぜaws_rds_clusterが必要なのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### 従来の問題点

1. **データベースの可用性問題**
   - 影響: ダウンタイムによるビジネス損失
   - 例: 手動フェイルオーバーの遅延

2. **スケーラビリティの制限**
   - 影響: 成長に伴うパフォーマンス劣化
   - 例: ストレージ容量の事前計画の必要性

#### aws_rds_clusterによる解決策

- 自動フェイルオーバーによる高可用性の実現
- ストレージの自動スケーリングによる柔軟性
- マネージドサービスによる運用負荷の削減

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

#### ビジネス面のメリット

1. **コスト削減**
   - 使用した分のみのストレージ課金
   - 運用人員の削減

2. **生産性向上**
   - 自動バックアップとリストア
   - パッチ適用の自動化

3. **スケーラビリティ**
   - リードレプリカの簡単な追加
   - ストレージの自動拡張

#### 技術面のメリット

1. **高パフォーマンス**
   - SSDベースの高速ストレージ
   - 並列クエリ処理

2. **データ保護**
   - 自動バックアップと暗号化
   - ポイントインタイムリカバリ

</details>

### デメリット

<details>
<summary>デメリットと対策</summary>

| デメリット | 影響 | 対策 |
|-----------|------|------|
| ベンダーロックイン | AWSへの依存 | データエクスポート戦略の準備 |
| コスト | 小規模では割高 | 適切なインスタンスサイズの選択 |
| カスタマイズ制限 | 特殊な設定が困難 | Aurora互換性の事前確認 |

</details>

### 他の選択肢との比較

<details>
<summary>比較表</summary>

| 項目 | Aurora | RDS (MySQL/PostgreSQL) | EC2上の自己管理DB |
|------|--------|----------------------|------------------|
| コスト | 中〜高 | 中 | 低〜高 |
| 性能 | 非常に高い | 高い | カスタマイズ次第 |
| 可用性 | 自動フェイルオーバー | 手動/自動選択可 | 自己実装 |
| 管理負荷 | 非常に低い | 低い | 高い |

</details>

---

## How - aws_rds_clusterの実装方法

### 基本的な使い方

<details>
<summary>基本実装例</summary>

#### セットアップ

```hcl
# プロバイダーの設定
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-northeast-1"
}
```

#### 基本的な実装

```hcl
# DBサブネットグループ
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet-${var.environment}"
  subnet_ids = var.private_subnet_ids

  tags = {
    Name = "${var.project_name}-db-subnet-${var.environment}"
  }
}

# RDSクラスター
resource "aws_rds_cluster" "main" {
  cluster_identifier      = "${var.project_name}-${var.environment}"
  engine                  = "aurora-postgresql"
  engine_version          = "15.4"
  database_name           = var.database_name
  master_username         = "postgres"
  master_password         = random_password.db_password.result
  port                    = 5432
  
  vpc_security_group_ids  = [aws_security_group.rds.id]
  db_subnet_group_name    = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  preferred_backup_window = "17:00-18:00"  # UTC
  
  storage_encrypted       = true
  kms_key_id             = aws_kms_key.rds.arn
  
  tags = {
    Name        = "${var.project_name}-aurora-${var.environment}"
    Environment = var.environment
  }
}

# パスワード生成
resource "random_password" "db_password" {
  length  = 32
  special = true
}

# Secrets Managerに保存
resource "aws_secretsmanager_secret" "db_password" {
  name = "${var.project_name}-db-password-${var.environment}"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db_password.result
}
```

#### 実行例

```bash
# 初期化
terraform init

# 計画の確認
terraform plan

# 適用
terraform apply

# 接続エンドポイントの確認
terraform output cluster_endpoint
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. セキュリティの強化

```hcl
# 暗号化の設定
resource "aws_rds_cluster" "secure" {
  cluster_identifier = "${var.project_name}-secure"
  
  # ストレージ暗号化
  storage_encrypted = true
  kms_key_id       = aws_kms_key.rds.arn
  
  # IAM認証の有効化
  iam_database_authentication_enabled = true
  
  # パスワードをSecrets Managerで管理
  master_password = random_password.master.result
  
  # 削除保護
  deletion_protection = var.environment == "production" ? true : false
  
  # 最終スナップショットの作成
  skip_final_snapshot       = false
  final_snapshot_identifier = "${var.project_name}-final-snapshot-${timestamp()}"
}

# セキュリティグループの制限
resource "aws_security_group" "rds" {
  name_prefix = "${var.project_name}-rds-"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]  # アプリケーションからのみ許可
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**理由**: データベースの機密性を保護し、不正アクセスを防止

#### 2. 高可用性の確保

```hcl
resource "aws_rds_cluster" "highly_available" {
  cluster_identifier = "${var.project_name}-ha"
  
  # Multi-AZ配置
  availability_zones = data.aws_availability_zones.available.names
  
  # バックアップ設定
  backup_retention_period      = 35  # 最大値
  preferred_backup_window      = "17:00-18:00"
  preferred_maintenance_window = "sun:18:00-sun:19:00"
  
  # バックトラック（MySQL Auroraのみ）
  backtrack_window = var.engine == "aurora-mysql" ? 72 : 0
  
  # 自動マイナーバージョンアップグレード
  auto_minor_version_upgrade = true
}

# 複数のインスタンス
resource "aws_rds_cluster_instance" "instances" {
  count              = var.instance_count
  identifier         = "${var.project_name}-${count.index}"
  cluster_identifier = aws_rds_cluster.highly_available.id
  instance_class     = var.instance_class
  engine             = aws_rds_cluster.highly_available.engine
  engine_version     = aws_rds_cluster.highly_available.engine_version
  
  performance_insights_enabled = true
  monitoring_interval         = 60
  monitoring_role_arn        = aws_iam_role.rds_monitoring.arn
}
```

**理由**: ダウンタイムを最小化し、データの可用性を確保

#### 3. モニタリングの設定

```hcl
resource "aws_rds_cluster" "monitored" {
  cluster_identifier = "${var.project_name}-monitored"
  
  # CloudWatchログのエクスポート
  enabled_cloudwatch_logs_exports = [
    var.engine == "aurora-postgresql" ? "postgresql" : "audit",
    var.engine == "aurora-postgresql" ? "postgresql" : "error",
    var.engine == "aurora-postgresql" ? "postgresql" : "general",
    var.engine == "aurora-postgresql" ? "postgresql" : "slowquery"
  ]
  
  # パフォーマンスインサイト（インスタンスレベルで設定）
}

# CloudWatchアラーム
resource "aws_cloudwatch_metric_alarm" "database_cpu" {
  alarm_name          = "${var.project_name}-rds-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/RDS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "This metric monitors RDS CPU utilization"
  
  dimensions = {
    DBClusterIdentifier = aws_rds_cluster.monitored.cluster_identifier
  }
}
```

**理由**: パフォーマンス問題の早期発見と対応

</details>

### よくある実装パターン

<details>
<summary>実装パターン集</summary>

#### パターン1: 標準的な本番環境構成

**用途**: 高可用性が求められる本番環境

```hcl
resource "aws_rds_cluster" "production" {
  cluster_identifier      = "${var.project_name}-production"
  engine                  = "aurora-postgresql"
  engine_version          = "15.4"
  database_name           = var.database_name
  master_username         = "dbadmin"
  master_password         = random_password.master.result
  
  vpc_security_group_ids  = [aws_security_group.rds_prod.id]
  db_subnet_group_name    = aws_db_subnet_group.prod.name
  
  # 高可用性設定
  availability_zones      = slice(data.aws_availability_zones.available.names, 0, 3)
  backup_retention_period = 30
  
  # セキュリティ設定
  storage_encrypted                   = true
  kms_key_id                         = aws_kms_key.rds.arn
  iam_database_authentication_enabled = true
  deletion_protection                 = true
  
  # ログエクスポート
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  tags = {
    Name        = "${var.project_name}-aurora-production"
    Environment = "production"
    Backup      = "critical"
  }
}

# 本番用インスタンス（3台構成）
resource "aws_rds_cluster_instance" "production" {
  count              = 3
  identifier         = "${var.project_name}-prod-${count.index}"
  cluster_identifier = aws_rds_cluster.production.id
  instance_class     = "db.r6g.xlarge"
  
  performance_insights_enabled = true
  monitoring_interval         = 60
}
```

#### パターン2: 開発環境用最小構成

**用途**: コストを抑えた開発・テスト環境

```hcl
resource "aws_rds_cluster" "development" {
  cluster_identifier      = "${var.project_name}-dev"
  engine                  = "aurora-postgresql"
  engine_version          = "15.4"
  database_name           = var.database_name
  master_username         = "developer"
  master_password         = "DevPassword123!"  # 開発環境のみ
  
  # 最小構成
  backup_retention_period = 1
  skip_final_snapshot     = true
  deletion_protection     = false
  
  # 基本的な暗号化のみ
  storage_encrypted = true
  
  tags = {
    Name        = "${var.project_name}-aurora-dev"
    Environment = "development"
  }
}

# 開発用インスタンス（1台のみ）
resource "aws_rds_cluster_instance" "development" {
  identifier         = "${var.project_name}-dev-instance"
  cluster_identifier = aws_rds_cluster.development.id
  instance_class     = "db.t4g.medium"
}
```

#### パターン3: サーバーレス構成

**用途**: 変動するワークロードや間欠的な使用

```hcl
resource "aws_rds_cluster" "serverless" {
  cluster_identifier = "${var.project_name}-serverless"
  engine             = "aurora-postgresql"
  engine_mode        = "serverless"
  engine_version     = "13.7"
  database_name      = var.database_name
  master_username    = "serverless_user"
  master_password    = random_password.master.result
  
  scaling_configuration {
    auto_pause               = true
    min_capacity            = 0.5
    max_capacity            = 1
    seconds_until_auto_pause = 300
  }
  
  # サーバーレスではインスタンスは不要
  
  tags = {
    Name = "${var.project_name}-aurora-serverless"
    Type = "serverless"
  }
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: InvalidDBClusterStateFault

**原因**: クラスターが適切な状態でない（作成中、削除中など）
**解決方法**:

```hcl
# 依存関係を明示的に設定
resource "aws_rds_cluster_instance" "example" {
  cluster_identifier = aws_rds_cluster.main.id
  # ...
  
  depends_on = [
    aws_rds_cluster.main
  ]
}

# タイムアウトの延長
resource "aws_rds_cluster" "main" {
  # ...
  
  timeouts {
    create = "60m"
    update = "120m"
    delete = "60m"
  }
}
```

#### エラー2: DBSubnetGroupNotFoundFault

**原因**: 指定されたDBサブネットグループが存在しない
**解決方法**:

```hcl
# サブネットグループを先に作成
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet"
  subnet_ids = var.private_subnet_ids

  tags = {
    Name = "${var.project_name}-db-subnet"
  }
}

resource "aws_rds_cluster" "main" {
  # ...
  db_subnet_group_name = aws_db_subnet_group.main.name
  
  depends_on = [aws_db_subnet_group.main]
}
```

#### エラー3: InsufficientDBClusterCapacityFault

**原因**: 指定されたインスタンスタイプが利用できない
**解決方法**:

```hcl
# 複数のインスタンスタイプから選択
locals {
  instance_types = [
    "db.r6g.large",
    "db.r5.large",
    "db.r6i.large"
  ]
}

# データソースで利用可能なインスタンスタイプを確認
data "aws_rds_orderable_db_instance" "available" {
  engine         = "aurora-postgresql"
  engine_version = "15.4"
  
  preferred_instance_classes = local.instance_types
}

resource "aws_rds_cluster_instance" "main" {
  # ...
  instance_class = data.aws_rds_orderable_db_instance.available.instance_class
}
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.08.07.11.48 - what_aws_rds_cluster_terraform_resource.md](daily/2025.08.07.11.48_what_aws_rds_cluster_terraform_resource.md)
  - Amazon Aurora DBクラスターの基本概念、設定項目、Aurora特有の機能について
  
- [2025.08.04.16.34 - what_rds_replication.md](daily/2025.08.04.16.34_what_rds_replication.md)
  - RDSレプリケーションの仕組みについて

### Why関連

- 現在のところ、aws_rds_clusterの「なぜ」に関するdaily-TILファイルはありません

### How関連

- [2025.08.07.11.52 - how_secure_rds_password_with_secrets_manager.md](daily/2025.08.07.11.52_how_secure_rds_password_with_secrets_manager.md)
  - RDSパスワードのSecrets Manager管理方法
  
- [2025.08.07.12.09 - how_use_secrets_manager_for_rds_in_terraform.md](daily/2025.08.07.12.09_how_use_secrets_manager_for_rds_in_terraform.md)
  - TerraformでのSecrets Manager統合実装

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 |
|-----------|---------|-------------|
| 1.0.0 | 2025-08-11 | 初版作成 |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。