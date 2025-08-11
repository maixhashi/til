# About Terraform aws_ecs_service Resource

> [!NOTE]
> このドキュメントはTerraform aws_ecs_serviceリソースに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [概要](#概要)
- [What - aws_ecs_serviceリソースとは](#what---aws_ecs_serviceリソースとは)
  - [基本構文](#基本構文)
  - [主要なパラメータ](#主要なパラメータ)
  - [出力属性](#出力属性)
- [Why - なぜaws_ecs_serviceリソースを使うのか](#why---なぜaws_ecs_serviceリソースを使うのか)
  - [解決する課題](#解決する課題)
  - [メリット](#メリット)
  - [注意点](#注意点)
- [How - aws_ecs_serviceリソースの実装方法](#how---aws_ecs_serviceリソースの実装方法)
  - [基本的な実装](#基本的な実装)
  - [高度な実装パターン](#高度な実装パターン)
  - [ベストプラクティス](#ベストプラクティス)
  - [トラブルシューティング](#トラブルシューティング)
- [参照：daily-TIL](#参照daily-til)

</details>

---

## 概要

aws_ecs_serviceリソースは、指定された数のタスクを常に実行し続けることを保証するECSのコア機能です。タスクの起動、停止、置き換えを自動的に管理し、ロードバランサーとの統合やヘルスチェック、自動スケーリングなどの高度な機能を提供します。

### キーポイント

- **タスク管理**: 指定された数のタスクを維持・管理
- **高可用性**: 自動復旧とローリングアップデート
- **統合機能**: ALB/NLB、Auto Scaling、Service Discoveryとの連携

---

## What - aws_ecs_serviceリソースとは

### 基本構文

<details>
<summary>基本構文の詳細</summary>

```hcl
resource "aws_ecs_service" "example" {
  # 基本設定
  name            = "my-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2
  
  # 起動タイプ設定
  launch_type = "FARGATE"
  
  # ネットワーク設定（Fargate必須）
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  # ロードバランサー設定
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 3000
  }
  
  # デプロイメント設定
  deployment_minimum_healthy_percent = 100
  deployment_maximum_percent         = 200
  
  # Circuit Breaker
  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }
  
  # プレイスメント戦略
  ordered_placement_strategy {
    type  = "spread"
    field = "attribute:ecs.availability-zone"
  }
  
  # タグ
  tags = {
    Name = "my-ecs-service"
  }
  
  # 依存関係
  depends_on = [
    aws_lb_listener.app
  ]
}
```

#### リソース名の構成

- **リソースタイプ**: `aws_ecs_service`
- **リソース名**: 任意の識別子（例: `web`, `api`, `worker`）
- **参照方法**: `aws_ecs_service.example.id`, `aws_ecs_service.example.name`

</details>

### 主要なパラメータ

<details>
<summary>パラメータの詳細説明</summary>

#### 基本パラメータ

| パラメータ | 必須 | 説明 | デフォルト |
|-----------|------|------|-----------|
| `name` | はい | サービス名 | - |
| `cluster` | いいえ | ECSクラスターのARNまたは名前 | デフォルトクラスター |
| `task_definition` | はい | タスク定義のARN | - |
| `desired_count` | いいえ | 実行したいタスク数 | 0 |

#### 起動タイプとキャパシティプロバイダー

| パラメータ | 説明 |
|-----------|------|
| `launch_type` | `EC2`、`FARGATE`、`EXTERNAL` |
| `capacity_provider_strategy` | キャパシティプロバイダーの使用戦略 |
| `platform_version` | Fargateプラットフォームバージョン |

#### ネットワーク設定（Fargate/awsvpc必須）

```hcl
network_configuration {
  subnets          = list(string)  # サブネットID
  security_groups  = list(string)  # セキュリティグループID
  assign_public_ip = bool          # パブリックIP割り当て
}
```

#### ロードバランサー設定

```hcl
load_balancer {
  target_group_arn = string  # ターゲットグループARN
  container_name   = string  # コンテナ名
  container_port   = number  # コンテナポート
  elb_name         = string  # Classic ELB名（非推奨）
}
```

#### デプロイメント設定

| パラメータ | 説明 | デフォルト |
|-----------|------|-----------|
| `deployment_minimum_healthy_percent` | 最小ヘルシー率 | 100 (Fargate) |
| `deployment_maximum_percent` | 最大起動率 | 200 |
| `force_new_deployment` | 強制的に新規デプロイ | false |
| `wait_for_steady_state` | 安定状態まで待機 | false |

#### Circuit Breaker設定

```hcl
deployment_circuit_breaker {
  enable   = bool  # Circuit Breaker有効化
  rollback = bool  # 失敗時の自動ロールバック
}
```

#### Service Discovery設定

```hcl
service_registries {
  registry_arn   = string  # Service Registry ARN
  port           = number  # ポート番号
  container_port = number  # コンテナポート
  container_name = string  # コンテナ名
}
```

#### プレイスメント設定

```hcl
# プレイスメント戦略
ordered_placement_strategy {
  type  = string  # spread, binpack, random
  field = string  # attribute:ecs.availability-zone等
}

# プレイスメント制約
placement_constraints {
  type       = string  # distinctInstance, memberOf
  expression = string  # 制約式
}
```

</details>

### 出力属性

<details>
<summary>出力属性の詳細</summary>

| 属性 | 説明 | 使用例 |
|------|------|--------|
| `id` | サービスのARN | `aws_ecs_service.example.id` |
| `name` | サービス名 | `aws_ecs_service.example.name` |
| `cluster` | クラスター名/ARN | `aws_ecs_service.example.cluster` |
| `iam_role` | サービスロールARN | `aws_ecs_service.example.iam_role` |
| `desired_count` | 希望タスク数 | `aws_ecs_service.example.desired_count` |

</details>

---

## Why - なぜaws_ecs_serviceリソースを使うのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### コンテナ運用の複雑性

1. **手動管理の限界**
   - タスクの死活監視
   - 障害時の手動復旧
   - スケーリング作業

2. **デプロイメントリスク**
   - サービス中断のリスク
   - ロールバック作業の複雑さ
   - バージョン管理の困難さ

3. **負荷分散の課題**
   - トラフィック分散の手動設定
   - ヘルスチェックの実装
   - 動的なターゲット管理

#### Terraformによる解決

```hcl
# 自動化されたコンテナ管理
module "ecs_service" {
  source = "./modules/ecs_service"
  
  service_name    = "${var.project_name}-${var.environment}"
  desired_count   = var.task_count
  
  # 自動復旧とスケーリング
  enable_circuit_breaker = true
  enable_auto_scaling    = true
}
```

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

1. **自動タスク管理**
   - 指定数のタスク維持
   - 自動ヘルスチェック
   - 障害時の自動置き換え

2. **安全なデプロイメント**
   - ローリングアップデート
   - Circuit Breakerによる保護
   - 自動ロールバック

3. **柔軟なスケーリング**
   - Auto Scalingとの統合
   - メトリクスベースの拡張
   - スケジュールスケーリング

4. **高度な統合**
   - ALB/NLBとの自動連携
   - Service Discovery
   - CloudWatch監視

</details>

### 注意点

<details>
<summary>注意点と対策</summary>

| 注意点 | 影響 | 対策 |
|--------|------|------|
| タスク起動時間 | デプロイ遅延 | ヘルスチェック間隔の調整 |
| リソース制限 | タスク起動失敗 | 適切なリソース割り当て |
| ネットワーク設定 | 接続不可 | セキュリティグループの確認 |
| IAMロール | 権限エラー | 必要な権限の付与 |

</details>

---

## How - aws_ecs_serviceリソースの実装方法

### 基本的な実装

<details>
<summary>基本実装例</summary>

#### シンプルなWebサービス

```hcl
# 基本的なECSサービス
resource "aws_ecs_service" "web" {
  name            = "web-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.web.arn
  desired_count   = 2
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  tags = {
    Name = "web-service"
  }
}
```

#### ALB統合サービス

```hcl
# ALBと統合されたECSサービス
resource "aws_ecs_service" "app" {
  name            = "${var.project_name}-${var.environment}"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.desired_count
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = var.container_name
    container_port   = var.container_port
  }
  
  # 安全なデプロイメント
  deployment_minimum_healthy_percent = 100
  deployment_maximum_percent         = 200
  
  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }
  
  depends_on = [
    aws_lb_listener.app,
    aws_iam_role_policy_attachment.ecs_task_execution
  ]
}
```

</details>

### 高度な実装パターン

<details>
<summary>高度な実装例</summary>

#### Blue/Greenデプロイメント対応

```hcl
# CodeDeployと統合されたECSサービス
resource "aws_ecs_service" "blue_green" {
  name            = "${var.project_name}-bg"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.desired_count
  
  deployment_controller {
    type = "CODE_DEPLOY"
  }
  
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.blue.arn
    container_name   = var.container_name
    container_port   = var.container_port
  }
  
  lifecycle {
    ignore_changes = [
      task_definition,
      load_balancer
    ]
  }
}
```

#### Service Discovery統合

```hcl
# Service Discovery用名前空間
resource "aws_service_discovery_private_dns_namespace" "internal" {
  name        = "internal.local"
  description = "Internal service discovery"
  vpc         = aws_vpc.main.id
}

# Service Discoveryサービス
resource "aws_service_discovery_service" "app" {
  name = "app"
  
  dns_config {
    namespace_id = aws_service_discovery_private_dns_namespace.internal.id
    
    dns_records {
      ttl  = 10
      type = "A"
    }
    
    routing_policy = "MULTIVALUE"
  }
  
  health_check_custom_config {
    failure_threshold = 1
  }
}

# Service Discovery対応ECSサービス
resource "aws_ecs_service" "with_discovery" {
  name            = "app-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 3
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  service_registries {
    registry_arn = aws_service_discovery_service.app.arn
  }
  
  # サービス間通信用の設定
  deployment_minimum_healthy_percent = 50
  deployment_maximum_percent         = 200
}
```

#### 複数ターゲットグループ対応

```hcl
# 複数のロードバランサー/ターゲットグループに対応
resource "aws_ecs_service" "multi_target" {
  name            = "multi-target-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
  
  # 外部向けALB
  load_balancer {
    target_group_arn = aws_lb_target_group.external.arn
    container_name   = "app"
    container_port   = 8080
  }
  
  # 内部向けALB
  load_balancer {
    target_group_arn = aws_lb_target_group.internal.arn
    container_name   = "app"
    container_port   = 8080
  }
  
  # 管理用NLB
  load_balancer {
    target_group_arn = aws_lb_target_group.admin.arn
    container_name   = "admin"
    container_port   = 9090
  }
}
```

#### スポットインスタンス活用

```hcl
# キャパシティプロバイダー戦略
resource "aws_ecs_service" "with_spot" {
  name            = "cost-optimized-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 10
  
  capacity_provider_strategy {
    capacity_provider = "FARGATE_SPOT"
    weight            = 4
    base              = 0
  }
  
  capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight            = 1
    base              = 2  # 最低2タスクは通常のFargate
  }
  
  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }
}
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. ヘルスチェックの最適化

```hcl
# ターゲットグループのヘルスチェック設定
resource "aws_lb_target_group" "app" {
  name     = "${var.project_name}-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200-299"
    path                = "/health"
    port                = "traffic-port"
    timeout             = 5
    unhealthy_threshold = 2
  }
  
  # ECSタスクの登録解除遅延
  deregistration_delay = 30
}

# ECSサービスでの考慮
resource "aws_ecs_service" "app" {
  # ... 他の設定 ...
  
  health_check_grace_period_seconds = 60  # 起動時の猶予期間
}
```

#### 2. 環境別設定の管理

```hcl
# 環境別の設定
locals {
  service_config = {
    dev = {
      desired_count                      = 1
      deployment_minimum_healthy_percent = 0
      deployment_maximum_percent         = 100
      enable_circuit_breaker             = false
    }
    stg = {
      desired_count                      = 2
      deployment_minimum_healthy_percent = 50
      deployment_maximum_percent         = 150
      enable_circuit_breaker             = true
    }
    prod = {
      desired_count                      = 4
      deployment_minimum_healthy_percent = 100
      deployment_maximum_percent         = 200
      enable_circuit_breaker             = true
    }
  }
}

resource "aws_ecs_service" "app" {
  name            = "${var.project_name}-${var.environment}"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  
  desired_count                      = local.service_config[var.environment].desired_count
  deployment_minimum_healthy_percent = local.service_config[var.environment].deployment_minimum_healthy_percent
  deployment_maximum_percent         = local.service_config[var.environment].deployment_maximum_percent
  
  deployment_circuit_breaker {
    enable   = local.service_config[var.environment].enable_circuit_breaker
    rollback = local.service_config[var.environment].enable_circuit_breaker
  }
  
  # ... 他の設定 ...
}
```

#### 3. Auto Scaling統合

```hcl
# Auto Scalingターゲット
resource "aws_appautoscaling_target" "ecs" {
  max_capacity       = 10
  min_capacity       = 2
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.app.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

# CPU使用率ベースのスケーリング
resource "aws_appautoscaling_policy" "cpu" {
  name               = "${var.project_name}-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs.service_namespace
  
  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value       = 70.0
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# メモリ使用率ベースのスケーリング
resource "aws_appautoscaling_policy" "memory" {
  name               = "${var.project_name}-memory-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs.service_namespace
  
  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }
    target_value = 80.0
  }
}
```

#### 4. セキュリティとアクセス制御

```hcl
# タスク実行ロール
resource "aws_iam_role" "ecs_task_execution" {
  name = "${var.project_name}-ecs-task-execution"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
    }]
  })
}

# 必要最小限の権限
resource "aws_iam_role_policy_attachment" "ecs_task_execution" {
  role       = aws_iam_role.ecs_task_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

# Secrets Manager アクセス権限
resource "aws_iam_role_policy" "secrets_access" {
  name = "${var.project_name}-secrets-access"
  role = aws_iam_role.ecs_task_execution.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "secretsmanager:GetSecretValue"
      ]
      Resource = [
        "arn:aws:secretsmanager:${var.region}:${var.account_id}:secret:${var.project_name}/*"
      ]
    }]
  })
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: タスクが起動しない

**症状**: desired_countを設定してもタスクが0のまま

**原因と解決方法**:
```bash
# CloudWatch Logsでエラーを確認
aws logs tail /ecs/${cluster_name}/${service_name}

# よくある原因:
# 1. メモリ/CPU不足
# 2. IAMロール権限不足
# 3. イメージ取得失敗
# 4. ヘルスチェック失敗
```

```hcl
# 解決例: リソース増加
resource "aws_ecs_task_definition" "app" {
  # ... 他の設定 ...
  
  cpu    = "512"   # 256 → 512
  memory = "1024"  # 512 → 1024
}
```

#### エラー2: デプロイメントが完了しない

**症状**: サービスが "UPDATE_IN_PROGRESS" のまま

**解決方法**:
```hcl
# タイムアウトと待機設定
resource "aws_ecs_service" "app" {
  # ... 他の設定 ...
  
  # ヘルスチェック猶予期間を増やす
  health_check_grace_period_seconds = 120
  
  # タスク起動の余裕を持たせる
  deployment_minimum_healthy_percent = 50
  deployment_maximum_percent         = 200
}
```

#### エラー3: ロードバランサー登録失敗

**エラーメッセージ**:
```
service was unable to place a task because no container instance met all of its requirements
```

**解決方法**:
```hcl
# セキュリティグループの確認
resource "aws_security_group_rule" "alb_to_ecs" {
  type                     = "ingress"
  from_port                = var.container_port
  to_port                  = var.container_port
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.alb.id
  security_group_id        = aws_security_group.ecs_tasks.id
}

# サブネット設定の確認
resource "aws_ecs_service" "app" {
  network_configuration {
    # ALBと同じVPC内のサブネット
    subnets = var.private_subnet_ids
  }
}
```

#### エラー4: Circuit Breakerによるロールバック

**症状**: 新しいデプロイメントが自動的にロールバックされる

**調査方法**:
```bash
# サービスイベントの確認
aws ecs describe-services \
  --cluster ${cluster_name} \
  --services ${service_name} \
  --query 'services[0].events[0:5]'
```

**解決方法**:
```hcl
# Circuit Breakerの一時的な無効化（デバッグ用）
resource "aws_ecs_service" "app" {
  deployment_circuit_breaker {
    enable   = false  # 一時的に無効化
    rollback = false
  }
  
  # または強制デプロイメント
  force_new_deployment = true
}
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.08.07.07.23 - what_desired_count_in_terraform_variables.md](../daily/2025.08.07.07.23_what_desired_count_in_terraform_variables.md)
  - desired_countパラメータの詳細
- [2025.08.07.11.33 - what_assign_public_ip_in_ecs_network_configuration.md](../daily/2025.08.07.11.33_what_assign_public_ip_in_ecs_network_configuration.md)
  - ネットワーク設定のassign_public_ipプロパティ
- [2025.08.07.11.37 - what_aws_ecs_task_definition.md](../daily/2025.08.07.11.37_what_aws_ecs_task_definition.md)
  - タスク定義との関係

### How関連

- [2025.08.07.11.27 - how_terraform_aws_ecs_service_implementation.md](../daily/2025.08.07.11.27_how_terraform_aws_ecs_service_implementation.md)
  - ECSサービスの詳細な実装方法
- [2025.08.07.11.11 - how_terraform_ecs_autoscaling_implementation.md](../daily/2025.08.07.11.11_how_terraform_ecs_autoscaling_implementation.md)
  - Auto Scaling設定の実装

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 | 作成者 |
|-----------|---------|-------------|---------|
| 1.0.0 | 2025-08-11 | 初版作成 | - |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。