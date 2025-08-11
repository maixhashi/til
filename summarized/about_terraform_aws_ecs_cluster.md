# About Terraform aws_ecs_cluster Resource

> [!NOTE]
> このドキュメントはTerraform aws_ecs_clusterリソースに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [概要](#概要)
- [What - aws_ecs_clusterリソースとは](#what---aws_ecs_clusterリソースとは)
  - [基本構文](#基本構文)
  - [主要なパラメータ](#主要なパラメータ)
  - [出力属性](#出力属性)
- [Why - なぜaws_ecs_clusterリソースを使うのか](#why---なぜaws_ecs_clusterリソースを使うのか)
  - [解決する課題](#解決する課題)
  - [メリット](#メリット)
  - [注意点](#注意点)
- [How - aws_ecs_clusterリソースの実装方法](#how---aws_ecs_clusterリソースの実装方法)
  - [基本的な実装](#基本的な実装)
  - [高度な実装パターン](#高度な実装パターン)
  - [ベストプラクティス](#ベストプラクティス)
  - [トラブルシューティング](#トラブルシューティング)
- [参照：daily-TIL](#参照daily-til)

</details>

---

## 概要

aws_ecs_clusterリソースは、AWS Elastic Container Service (ECS)でコンテナ化されたアプリケーションを実行するための論理的なグループを作成・管理するTerraformリソースです。EC2インスタンスやFargateなどのコンピューティングリソースとタスク、サービスを管理する基盤となります。

### キーポイント

- **コンテナオーケストレーション**: Dockerコンテナの実行・管理の中心
- **2つの起動タイプ**: Fargate（サーバーレス）とEC2（完全制御）
- **統合監視**: Container Insightsによる詳細なメトリクス収集

---

## What - aws_ecs_clusterリソースとは

### 基本構文

<details>
<summary>基本構文の詳細</summary>

```hcl
resource "aws_ecs_cluster" "example" {
  # 基本設定
  name = "my-cluster"
  
  # Container Insights設定
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
  
  # キャパシティプロバイダー設定
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight            = 1
    base              = 0
  }
  
  # 実行コマンド設定（ECS Exec）
  configuration {
    execute_command_configuration {
      kms_key_id = aws_kms_key.ecs.arn
      logging    = "OVERRIDE"
      
      log_configuration {
        cloud_watch_encryption_enabled = true
        cloud_watch_log_group_name     = aws_cloudwatch_log_group.ecs_exec.name
        s3_bucket_name                 = aws_s3_bucket.ecs_exec_logs.id
        s3_key_prefix                  = "exec-logs"
      }
    }
  }
  
  # タグ
  tags = {
    Name = "my-ecs-cluster"
  }
}
```

#### リソース名の構成

- **リソースタイプ**: `aws_ecs_cluster`
- **リソース名**: 任意の識別子（例: `main`, `production`, `web`）
- **参照方法**: `aws_ecs_cluster.example.id`, `aws_ecs_cluster.example.arn`

</details>

### 主要なパラメータ

<details>
<summary>パラメータの詳細説明</summary>

#### 基本パラメータ

| パラメータ | 必須 | 説明 | デフォルト |
|-----------|------|------|-----------|
| `name` | はい | クラスター名 | - |
| `tags` | いいえ | タグのマップ | - |

#### Container Insights設定

| 設定名 | 値 | 説明 |
|--------|-----|------|
| `containerInsights` | `enabled`/`disabled` | CloudWatch Container Insightsの有効化 |

```hcl
setting {
  name  = "containerInsights"
  value = "enabled"
}
```

#### キャパシティプロバイダー設定

| パラメータ | 説明 |
|-----------|------|
| `capacity_providers` | 使用可能なキャパシティプロバイダーのリスト |
| `default_capacity_provider_strategy` | デフォルトの配置戦略 |

キャパシティプロバイダーの種類：
- `FARGATE`: Fargateの標準料金
- `FARGATE_SPOT`: Fargateのスポット料金（最大70%割引）
- カスタムキャパシティプロバイダー（EC2 Auto Scalingグループ）

#### 実行コマンド設定（ECS Exec）

| パラメータ | 説明 |
|-----------|------|
| `kms_key_id` | セッション暗号化用のKMSキー |
| `logging` | ログ設定（`NONE`, `DEFAULT`, `OVERRIDE`） |
| `log_configuration` | ログの出力先設定 |

</details>

### 出力属性

<details>
<summary>出力属性の詳細</summary>

| 属性 | 説明 | 使用例 |
|------|------|--------|
| `id` | クラスターのID/名前 | `aws_ecs_cluster.example.id` |
| `arn` | クラスターのARN | `aws_ecs_cluster.example.arn` |
| `capacity_providers` | 関連付けられたキャパシティプロバイダー | `aws_ecs_cluster.example.capacity_providers` |
| `default_capacity_provider_strategy` | デフォルトの配置戦略 | `aws_ecs_cluster.example.default_capacity_provider_strategy` |
| `setting` | クラスター設定 | `aws_ecs_cluster.example.setting` |
| `configuration` | クラスター構成 | `aws_ecs_cluster.example.configuration` |

</details>

---

## Why - なぜaws_ecs_clusterリソースを使うのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### コンテナ管理の複雑性

1. **手動管理の限界**
   - コンテナの配置とスケジューリング
   - リソース使用率の最適化
   - 障害時の自動復旧

2. **インフラストラクチャの課題**
   - サーバー管理の負担
   - スケーリングの複雑性
   - セキュリティパッチの適用

3. **監視とログ管理**
   - 分散されたログの収集
   - パフォーマンスメトリクスの統合
   - トラブルシューティングの困難さ

#### Terraformによる解決

```hcl
# インフラストラクチャのコード化
module "ecs_cluster" {
  source = "./modules/ecs_cluster"
  
  cluster_name = "${var.project_name}-${var.environment}"
  enable_container_insights = true
  
  # Fargateでサーバー管理を排除
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
}
```

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

1. **サーバーレスオプション（Fargate）**
   - インフラ管理不要
   - 自動的なセキュリティパッチ
   - タスク単位の課金

2. **柔軟なスケーリング**
   - 自動スケーリング対応
   - 複数のキャパシティプロバイダー
   - コスト最適化（FARGATE_SPOT）

3. **統合された監視**
   - Container Insightsによる詳細メトリクス
   - CloudWatchとの深い統合
   - ECS Execによるデバッグ支援

4. **エンタープライズ対応**
   - IAMによる細かなアクセス制御
   - KMS暗号化サポート
   - コンプライアンス準拠

</details>

### 注意点

<details>
<summary>注意点と対策</summary>

| 注意点 | 影響 | 対策 |
|--------|------|------|
| Container Insightsのコスト | CloudWatchログとメトリクスの料金 | 環境別に有効/無効を設定 |
| クラスター名の一意性 | リージョン内で重複不可 | 環境とプロジェクト名を含める |
| Fargate制限 | GPU非対応、永続ストレージ制限 | EC2起動タイプの検討 |
| リージョン制限 | クラスターはリージョン固定 | マルチリージョン設計 |

</details>

---

## How - aws_ecs_clusterリソースの実装方法

### 基本的な実装

<details>
<summary>基本実装例</summary>

#### 最小構成

```hcl
# シンプルなECSクラスター
resource "aws_ecs_cluster" "minimal" {
  name = "my-minimal-cluster"
}
```

#### 標準的な構成

```hcl
# 本番環境向けの標準構成
resource "aws_ecs_cluster" "standard" {
  name = "${var.project_name}-${var.environment}"
  
  # Container Insights有効化
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
  
  # タグ付け
  tags = {
    Name        = "${var.project_name}-cluster-${var.environment}"
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
  }
}
```

#### Fargate専用クラスター

```hcl
# Fargateのみを使用するクラスター
resource "aws_ecs_cluster" "fargate_only" {
  name = "${var.project_name}-fargate-${var.environment}"
  
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  # コスト最適化戦略
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE_SPOT"
    weight            = 2
  }
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight            = 1
    base              = 1  # 最低1タスクは通常のFargateで実行
  }
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

</details>

### 高度な実装パターン

<details>
<summary>高度な実装例</summary>

#### ECS Exec対応クラスター

```hcl
# KMSキーの作成
resource "aws_kms_key" "ecs_exec" {
  description             = "KMS key for ECS Exec"
  deletion_window_in_days = 7
  
  tags = {
    Name = "${var.project_name}-ecs-exec-key"
  }
}

# CloudWatchロググループ
resource "aws_cloudwatch_log_group" "ecs_exec" {
  name              = "/ecs/exec/${var.project_name}-${var.environment}"
  retention_in_days = 7
  
  kms_key_id = aws_kms_key.ecs_exec.arn
}

# S3バケット（ログアーカイブ用）
resource "aws_s3_bucket" "ecs_exec_logs" {
  bucket = "${var.project_name}-ecs-exec-logs-${var.environment}"
}

# ECS Exec対応クラスター
resource "aws_ecs_cluster" "with_exec" {
  name = "${var.project_name}-${var.environment}"
  
  configuration {
    execute_command_configuration {
      kms_key_id = aws_kms_key.ecs_exec.arn
      logging    = "OVERRIDE"
      
      log_configuration {
        cloud_watch_encryption_enabled = true
        cloud_watch_log_group_name     = aws_cloudwatch_log_group.ecs_exec.name
        s3_bucket_name                 = aws_s3_bucket.ecs_exec_logs.id
        s3_key_prefix                  = "exec-logs"
      }
    }
  }
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

#### ハイブリッドクラスター（EC2 + Fargate）

```hcl
# EC2用のAuto Scalingグループ
resource "aws_autoscaling_group" "ecs" {
  name                = "${var.project_name}-ecs-asg"
  vpc_zone_identifier = var.private_subnet_ids
  min_size            = 0
  max_size            = 10
  desired_capacity    = 2
  
  launch_template {
    id      = aws_launch_template.ecs.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "${var.project_name}-ecs-instance"
    propagate_at_launch = true
  }
}

# キャパシティプロバイダー
resource "aws_ecs_capacity_provider" "ec2" {
  name = "${var.project_name}-ec2-capacity-provider"
  
  auto_scaling_group_provider {
    auto_scaling_group_arn         = aws_autoscaling_group.ecs.arn
    managed_termination_protection = "ENABLED"
    
    managed_scaling {
      maximum_scaling_step_size = 10
      minimum_scaling_step_size = 1
      status                    = "ENABLED"
      target_capacity           = 100
    }
  }
}

# ハイブリッドクラスター
resource "aws_ecs_cluster" "hybrid" {
  name = "${var.project_name}-hybrid-${var.environment}"
  
  capacity_providers = [
    aws_ecs_capacity_provider.ec2.name,
    "FARGATE",
    "FARGATE_SPOT"
  ]
  
  # デフォルト戦略：EC2を優先
  default_capacity_provider_strategy {
    capacity_provider = aws_ecs_capacity_provider.ec2.name
    weight            = 2
    base              = 2
  }
  
  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight            = 1
  }
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

#### マルチ環境対応

```hcl
# 環境別設定
locals {
  cluster_config = {
    dev = {
      container_insights = false
      capacity_providers = ["FARGATE_SPOT"]
      retention_days     = 3
    }
    stg = {
      container_insights = true
      capacity_providers = ["FARGATE", "FARGATE_SPOT"]
      retention_days     = 7
    }
    prod = {
      container_insights = true
      capacity_providers = ["FARGATE"]
      retention_days     = 30
    }
  }
}

# 環境別クラスター
resource "aws_ecs_cluster" "multi_env" {
  name = "${var.project_name}-${var.environment}"
  
  capacity_providers = local.cluster_config[var.environment].capacity_providers
  
  setting {
    name  = "containerInsights"
    value = local.cluster_config[var.environment].container_insights ? "enabled" : "disabled"
  }
  
  tags = {
    Name        = "${var.project_name}-cluster-${var.environment}"
    Environment = var.environment
    Project     = var.project_name
  }
}
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. 命名規則の統一

```hcl
# 一貫した命名規則
locals {
  cluster_name = "${var.organization}-${var.project_name}-${var.environment}"
  
  # 例:
  # mycompany-web-app-dev
  # mycompany-web-app-stg
  # mycompany-web-app-prod
}

resource "aws_ecs_cluster" "main" {
  name = local.cluster_name
  
  tags = {
    Name        = local.cluster_name
    Environment = var.environment
    Project     = var.project_name
    Organization = var.organization
  }
}
```

#### 2. 環境別の最適化

```hcl
# 環境別の設定管理
variable "environment_config" {
  description = "Environment-specific configuration"
  type = map(object({
    enable_insights    = bool
    use_spot_instances = bool
    exec_logging       = string
  }))
  
  default = {
    dev = {
      enable_insights    = false
      use_spot_instances = true
      exec_logging       = "NONE"
    }
    stg = {
      enable_insights    = true
      use_spot_instances = true
      exec_logging       = "DEFAULT"
    }
    prod = {
      enable_insights    = true
      use_spot_instances = false
      exec_logging       = "OVERRIDE"
    }
  }
}

resource "aws_ecs_cluster" "optimized" {
  name = "${var.project_name}-${var.environment}"
  
  # Container Insights
  setting {
    name  = "containerInsights"
    value = var.environment_config[var.environment].enable_insights ? "enabled" : "disabled"
  }
  
  # キャパシティプロバイダー
  capacity_providers = var.environment_config[var.environment].use_spot_instances ? 
    ["FARGATE", "FARGATE_SPOT"] : ["FARGATE"]
  
  # ECS Exec設定
  dynamic "configuration" {
    for_each = var.environment_config[var.environment].exec_logging != "NONE" ? [1] : []
    
    content {
      execute_command_configuration {
        logging = var.environment_config[var.environment].exec_logging
      }
    }
  }
}
```

#### 3. モニタリングとアラート

```hcl
# CloudWatchアラーム
resource "aws_cloudwatch_metric_alarm" "cluster_cpu" {
  alarm_name          = "${var.project_name}-cluster-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "This metric monitors cluster CPU utilization"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
  }
  
  alarm_actions = [aws_sns_topic.alerts.arn]
}

# メモリアラーム
resource "aws_cloudwatch_metric_alarm" "cluster_memory" {
  alarm_name          = "${var.project_name}-cluster-memory-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "MemoryUtilization"
  namespace           = "AWS/ECS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  
  dimensions = {
    ClusterName = aws_ecs_cluster.main.name
  }
  
  alarm_actions = [aws_sns_topic.alerts.arn]
}
```

#### 4. セキュリティ強化

```hcl
# IAMロールの制限
data "aws_iam_policy_document" "ecs_assume_role" {
  statement {
    actions = ["sts:AssumeRole"]
    
    principals {
      type        = "Service"
      identifiers = ["ecs.amazonaws.com"]
    }
  }
}

# クラスター用IAMロール
resource "aws_iam_role" "ecs_cluster" {
  name               = "${var.project_name}-ecs-cluster-role"
  assume_role_policy = data.aws_iam_policy_document.ecs_assume_role.json
  
  tags = {
    Name = "${var.project_name}-ecs-cluster-role"
  }
}

# 必要最小限の権限
resource "aws_iam_role_policy_attachment" "ecs_cluster" {
  role       = aws_iam_role.ecs_cluster.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSServiceRolePolicy"
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: クラスター名の重複

**エラーメッセージ**:
```
Error: error creating ECS cluster: ClientException: The specified cluster name already exists
```

**原因**: リージョン内で既に同名のクラスターが存在

**解決方法**:
```hcl
# ユニークな名前を生成
resource "random_string" "suffix" {
  length  = 8
  special = false
  upper   = false
}

resource "aws_ecs_cluster" "unique" {
  name = "${var.project_name}-${var.environment}-${random_string.suffix.result}"
}
```

#### エラー2: Container Insightsの権限不足

**エラーメッセージ**:
```
AccessDeniedException: User is not authorized to perform: ecs:PutAccountSetting
```

**解決方法**:
```hcl
# 必要な権限をIAMポリシーに追加
data "aws_iam_policy_document" "container_insights" {
  statement {
    effect = "Allow"
    actions = [
      "ecs:PutAccountSetting",
      "ecs:PutAccountSettingDefault"
    ]
    resources = ["*"]
  }
}
```

#### エラー3: キャパシティプロバイダーが見つからない

**エラーメッセージ**:
```
InvalidParameterException: The specified capacity provider was not found
```

**原因**: カスタムキャパシティプロバイダーが作成されていない

**解決方法**:
```hcl
# キャパシティプロバイダーの作成を確認
resource "aws_ecs_cluster_capacity_providers" "example" {
  cluster_name = aws_ecs_cluster.main.name
  
  capacity_providers = ["FARGATE", "FARGATE_SPOT"]
  
  default_capacity_provider_strategy {
    base              = 1
    weight            = 100
    capacity_provider = "FARGATE"
  }
}
```

#### エラー4: ECS Execの設定エラー

**エラーメッセージ**:
```
InvalidParameterException: KMS key not found
```

**解決方法**:
```hcl
# KMSキーの依存関係を明示
resource "aws_ecs_cluster" "with_exec" {
  name = var.cluster_name
  
  configuration {
    execute_command_configuration {
      kms_key_id = aws_kms_key.ecs_exec.arn
      # ...
    }
  }
  
  # 依存関係を明示
  depends_on = [
    aws_kms_key.ecs_exec,
    aws_kms_alias.ecs_exec
  ]
}
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.08.07.11.16 - what_aws_ecs_cluster_resource.md](../daily/2025.08.07.11.16_what_aws_ecs_cluster_resource.md)
  - ECSクラスターリソースの基本概念と構成要素

### How関連

- [2025.08.07.11.22 - how_docker_and_ecs_work_together.md](../daily/2025.08.07.11.22_how_docker_and_ecs_work_together.md)
  - DockerとECSの連携方法

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 |
|-----------|---------|-------------|
| 1.0.0 | 2025-08-11 | 初版作成 |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。
