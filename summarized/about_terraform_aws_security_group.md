# About Terraform aws_security_group Resource

> [!NOTE]
> このドキュメントはTerraform aws_security_groupリソースに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [About Terraform aws\_security\_group Resource](#about-terraform-aws_security_group-resource)
  - [目次](#目次)
  - [概要](#概要)
    - [キーポイント](#キーポイント)
  - [What - aws\_security\_groupリソースとは](#what---aws_security_groupリソースとは)
    - [基本構文](#基本構文)
      - [リソース名の構成](#リソース名の構成)
    - [主要なパラメータ](#主要なパラメータ)
      - [基本パラメータ](#基本パラメータ)
      - [インバウンドルール（ingress）](#インバウンドルールingress)
      - [アウトバウンドルール（egress）](#アウトバウンドルールegress)
      - [その他のパラメータ](#その他のパラメータ)
    - [出力属性](#出力属性)
  - [Why - なぜaws\_security\_groupリソースを使うのか](#why---なぜaws_security_groupリソースを使うのか)
    - [解決する課題](#解決する課題)
      - [セキュリティ管理の複雑性](#セキュリティ管理の複雑性)
      - [Terraformによる解決](#terraformによる解決)
    - [メリット](#メリット)
    - [注意点](#注意点)
  - [How - aws\_security\_groupリソースの実装方法](#how---aws_security_groupリソースの実装方法)
    - [基本的な実装](#基本的な実装)
      - [Webサーバー用セキュリティグループ](#webサーバー用セキュリティグループ)
      - [データベース用セキュリティグループ](#データベース用セキュリティグループ)
    - [高度な実装パターン](#高度な実装パターン)
      - [動的ルール管理](#動的ルール管理)
      - [ECS/Fargate用セキュリティグループ](#ecsfargate用セキュリティグループ)
      - [Lambda VPC用セキュリティグループ](#lambda-vpc用セキュリティグループ)
      - [踏み台サーバー用セキュリティグループ](#踏み台サーバー用セキュリティグループ)
    - [ベストプラクティス](#ベストプラクティス)
      - [1. 最小権限の原則](#1-最小権限の原則)
      - [2. 説明の活用](#2-説明の活用)
      - [3. 動的なIP範囲の管理](#3-動的なip範囲の管理)
      - [4. モジュール化](#4-モジュール化)
    - [トラブルシューティング](#トラブルシューティング)
      - [エラー1: 循環参照](#エラー1-循環参照)
      - [エラー2: ルール数の上限](#エラー2-ルール数の上限)
      - [エラー3: 名前の重複](#エラー3-名前の重複)
      - [エラー4: 不正なプロトコル番号](#エラー4-不正なプロトコル番号)
  - [参照：daily-TIL](#参照daily-til)
    - [What関連](#what関連)
    - [Why関連](#why関連)
    - [How関連](#how関連)
  - [バージョン履歴](#バージョン履歴)

</details>

---

## 概要

aws_security_groupリソースは、AWS VPC内のインスタンスレベルファイアウォールを定義・管理するためのTerraformリソースです。ステートフルなトラフィック制御により、インバウンド（ingress）とアウトバウンド（egress）の通信を細かく制御します。

### キーポイント

- **ステートフルファイアウォール**: 接続状態を追跡し、戻りトラフィックを自動許可
- **最小権限の原則**: 必要な通信のみを許可する設計
- **動的ルール管理**: 他のセキュリティグループやCIDRブロックとの柔軟な連携

---

## What - aws_security_groupリソースとは

### 基本構文

<details>
<summary>基本構文の詳細</summary>

```hcl
resource "aws_security_group" "example" {
  # 基本設定
  name        = "example-security-group"
  description = "Security group for example application"
  vpc_id      = aws_vpc.main.id
  
  # インバウンドルール（複数定義可能）
  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  # アウトバウンドルール
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  # タグ
  tags = {
    Name = "example-sg"
  }
  
  # ライフサイクル設定
  lifecycle {
    create_before_destroy = true
  }
}
```

#### リソース名の構成

- **リソースタイプ**: `aws_security_group`
- **リソース名**: 任意の識別子（例: `web`, `app`, `database`）
- **参照方法**: `aws_security_group.example.id`

</details>

### 主要なパラメータ

<details>
<summary>パラメータの詳細説明</summary>

#### 基本パラメータ

| パラメータ | 必須 | 説明 | デフォルト |
|-----------|------|------|-----------|
| `name` | いいえ | セキュリティグループ名 | Terraformが生成 |
| `name_prefix` | いいえ | 名前のプレフィックス | - |
| `description` | いいえ | セキュリティグループの説明 | "Managed by Terraform" |
| `vpc_id` | いいえ | VPCのID | デフォルトVPC |

#### インバウンドルール（ingress）

| パラメータ | 説明 |
|-----------|------|
| `from_port` | 開始ポート番号（0-65535） |
| `to_port` | 終了ポート番号（0-65535） |
| `protocol` | プロトコル（tcp, udp, icmp, all/-1） |
| `cidr_blocks` | 許可するIPv4 CIDRブロックのリスト |
| `ipv6_cidr_blocks` | 許可するIPv6 CIDRブロックのリスト |
| `security_groups` | 許可するセキュリティグループIDのリスト |
| `self` | 同じセキュリティグループからの通信を許可 |
| `description` | ルールの説明 |

#### アウトバウンドルール（egress）

インバウンドルールと同じパラメータ構造を持ちます。

#### その他のパラメータ

| パラメータ | 説明 |
|-----------|------|
| `revoke_rules_on_delete` | 削除時に関連するルールを取り消すか |
| `tags` | タグのマップ |

</details>

### 出力属性

<details>
<summary>出力属性の詳細</summary>

| 属性 | 説明 | 使用例 |
|------|------|--------|
| `id` | セキュリティグループのID | `aws_security_group.example.id` |
| `arn` | セキュリティグループのARN | `aws_security_group.example.arn` |
| `vpc_id` | 所属するVPCのID | `aws_security_group.example.vpc_id` |
| `owner_id` | AWSアカウントID | `aws_security_group.example.owner_id` |
| `name` | セキュリティグループ名 | `aws_security_group.example.name` |
| `description` | セキュリティグループの説明 | `aws_security_group.example.description` |

</details>

---

## Why - なぜaws_security_groupリソースを使うのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### セキュリティ管理の複雑性

1. **手動設定の問題**
   - ルールの重複や矛盾
   - 設定ミスによるセキュリティホール
   - 変更履歴の追跡困難

2. **スケーラビリティの課題**
   - 大規模環境での一貫性維持
   - 環境間でのルール同期
   - 動的なサービス間通信の管理

3. **コンプライアンス要件**
   - 監査証跡の不足
   - セキュリティポリシーの適用漏れ
   - 最小権限原則の徹底困難

#### Terraformによる解決

```hcl
# セキュリティグループのモジュール化
module "security_groups" {
  source = "./modules/security_groups"
  
  vpc_id      = aws_vpc.main.id
  environment = var.environment
  
  # 組織のセキュリティポリシーを適用
  allowed_ssh_cidrs = var.allowed_ssh_cidrs
  allowed_web_cidrs = var.allowed_web_cidrs
}
```

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

1. **宣言的セキュリティ管理**
   - 意図した状態を明確に定義
   - ルールの可視化と文書化
   - バージョン管理による変更追跡

2. **動的な関係性管理**
   - セキュリティグループ間の参照
   - 自動的な依存関係解決
   - サービスディスカバリとの統合

3. **再利用性と一貫性**
   - モジュール化による標準化
   - 環境間での設定共有
   - ベストプラクティスの強制

4. **監査とコンプライアンス**
   - 設定の自動検証
   - 変更履歴の完全な記録
   - コンプライアンス要件の実装

</details>

### 注意点

<details>
<summary>注意点と対策</summary>

| 注意点 | 影響 | 対策 |
|--------|------|------|
| ルール数の制限 | SGあたり60ルール（in/out各） | ルールの集約と最適化 |
| 循環参照 | 相互参照時のエラー | aws_security_group_ruleリソースの使用 |
| デフォルトegress | 作成時に全許可ルールが追加 | 明示的なegress定義 |
| 名前の一意性 | VPC内で重複不可 | name_prefixの使用 |

</details>

---

## How - aws_security_groupリソースの実装方法

### 基本的な実装

<details>
<summary>基本実装例</summary>

#### Webサーバー用セキュリティグループ

```hcl
# ALB用セキュリティグループ
resource "aws_security_group" "alb" {
  name        = "alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description = "HTTP from anywhere"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "alb-sg"
  }
}

# アプリケーション用セキュリティグループ
resource "aws_security_group" "app" {
  name        = "app-sg"
  description = "Security group for application servers"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description     = "HTTP from ALB"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }
  
  egress {
    description = "All outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "app-sg"
  }
}
```

#### データベース用セキュリティグループ

```hcl
# RDS用セキュリティグループ
resource "aws_security_group" "database" {
  name        = "database-sg"
  description = "Security group for RDS database"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description     = "MySQL/Aurora from app servers"
    from_port       = 3306
    to_port         = 3306
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
  
  # 明示的にegressを定義（デフォルトを上書き）
  egress {
    description = "No outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    self        = true  # 自分自身への通信のみ許可
  }
  
  tags = {
    Name = "database-sg"
  }
}
```

</details>

### 高度な実装パターン

<details>
<summary>高度な実装例</summary>

#### 動的ルール管理

```hcl
# ルール定義の変数化
variable "app_sg_rules" {
  description = "Security group rules for application"
  type = map(object({
    type        = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = optional(list(string))
    source_sg   = optional(string)
    description = string
  }))
  
  default = {
    http = {
      type        = "ingress"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      source_sg   = "alb"
      description = "HTTP from ALB"
    }
    https = {
      type        = "ingress"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      source_sg   = "alb"
      description = "HTTPS from ALB"
    }
    ssh_bastion = {
      type        = "ingress"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      source_sg   = "bastion"
      description = "SSH from bastion"
    }
  }
}

# セキュリティグループの作成
resource "aws_security_group" "main" {
  name_prefix = "${var.project_name}-${var.environment}-"
  description = "Managed by Terraform"
  vpc_id      = var.vpc_id
  
  lifecycle {
    create_before_destroy = true
  }
  
  tags = {
    Name = "${var.project_name}-${var.environment}-sg"
  }
}

# 個別ルールリソースの使用（循環参照回避）
resource "aws_security_group_rule" "rules" {
  for_each = var.app_sg_rules
  
  security_group_id = aws_security_group.main.id
  type              = each.value.type
  from_port         = each.value.from_port
  to_port           = each.value.to_port
  protocol          = each.value.protocol
  description       = each.value.description
  
  # 条件付きでソースを設定
  cidr_blocks              = each.value.cidr_blocks
  source_security_group_id = each.value.source_sg != null ? var.security_group_ids[each.value.source_sg] : null
}
```

#### ECS/Fargate用セキュリティグループ

```hcl
# ECSタスク用セキュリティグループ
resource "aws_security_group" "ecs_tasks" {
  name_prefix = "${var.project_name}-ecs-tasks-"
  description = "Security group for ECS tasks"
  vpc_id      = var.vpc_id
  
  tags = {
    Name = "${var.project_name}-ecs-tasks-sg"
  }
}

# ALBからECSタスクへのアクセス許可
resource "aws_security_group_rule" "ecs_from_alb" {
  type                     = "ingress"
  from_port                = var.container_port
  to_port                  = var.container_port
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.alb.id
  security_group_id        = aws_security_group.ecs_tasks.id
  description              = "Allow traffic from ALB"
}

# ECSタスクからのアウトバウンド
resource "aws_security_group_rule" "ecs_egress" {
  type              = "egress"
  from_port         = 0
  to_port           = 0
  protocol          = "-1"
  cidr_blocks       = ["0.0.0.0/0"]
  security_group_id = aws_security_group.ecs_tasks.id
  description       = "Allow all outbound traffic"
}

# ECSタスク間の通信（サービスディスカバリ使用時）
resource "aws_security_group_rule" "ecs_self" {
  type              = "ingress"
  from_port         = 0
  to_port           = 65535
  protocol          = "tcp"
  self              = true
  security_group_id = aws_security_group.ecs_tasks.id
  description       = "Allow communication between tasks"
}
```

#### Lambda VPC用セキュリティグループ

```hcl
# Lambda関数用セキュリティグループ
resource "aws_security_group" "lambda" {
  name_prefix = "${var.project_name}-lambda-"
  description = "Security group for Lambda functions in VPC"
  vpc_id      = var.vpc_id
  
  tags = {
    Name = "${var.project_name}-lambda-sg"
  }
}

# Lambdaからのアウトバウンドルール
locals {
  lambda_egress_rules = {
    rds = {
      port        = 3306
      protocol    = "tcp"
      target_sg   = aws_security_group.database.id
      description = "Access to RDS"
    }
    elasticache = {
      port        = 6379
      protocol    = "tcp"
      target_sg   = aws_security_group.elasticache.id
      description = "Access to ElastiCache"
    }
    https = {
      port        = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS to internet (via NAT)"
    }
  }
}

resource "aws_security_group_rule" "lambda_egress" {
  for_each = local.lambda_egress_rules
  
  type                     = "egress"
  from_port                = each.value.port
  to_port                  = each.value.port
  protocol                 = each.value.protocol
  security_group_id        = aws_security_group.lambda.id
  description              = each.value.description
  
  # 条件付きターゲット
  source_security_group_id = lookup(each.value, "target_sg", null)
  cidr_blocks              = lookup(each.value, "cidr_blocks", null)
}
```

#### 踏み台サーバー用セキュリティグループ

```hcl
# 踏み台サーバー用セキュリティグループ
resource "aws_security_group" "bastion" {
  name_prefix = "${var.project_name}-bastion-"
  description = "Security group for bastion host"
  vpc_id      = var.vpc_id
  
  tags = {
    Name = "${var.project_name}-bastion-sg"
  }
}

# 許可するIPアドレスの管理
variable "allowed_ssh_ips" {
  description = "List of IP addresses allowed to SSH"
  type = list(object({
    cidr        = string
    description = string
  }))
  
  default = [
    {
      cidr        = "203.0.113.0/24"
      description = "Office network"
    },
    {
      cidr        = "198.51.100.0/24"
      description = "VPN network"
    }
  ]
}

# SSH接続ルール
resource "aws_security_group_rule" "bastion_ssh" {
  for_each = { for idx, ip in var.allowed_ssh_ips : idx => ip }
  
  type              = "ingress"
  from_port         = 22
  to_port           = 22
  protocol          = "tcp"
  cidr_blocks       = [each.value.cidr]
  security_group_id = aws_security_group.bastion.id
  description       = "SSH from ${each.value.description}"
}

# 踏み台からプライベートインスタンスへのSSH
resource "aws_security_group_rule" "bastion_to_private" {
  type                     = "egress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.private_instances.id
  security_group_id        = aws_security_group.bastion.id
  description              = "SSH to private instances"
}
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. 最小権限の原則

```hcl
# ❌ 悪い例：過度に広い許可
resource "aws_security_group" "bad_example" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ✅ 良い例：必要最小限の許可
resource "aws_security_group" "good_example" {
  ingress {
    description = "HTTPS from CloudFront"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = data.aws_ip_ranges.cloudfront.cidr_blocks
  }
}
```

#### 2. 説明の活用

```hcl
# すべてのルールに説明を追加
resource "aws_security_group_rule" "example" {
  type              = "ingress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = ["10.0.0.0/8"]
  security_group_id = aws_security_group.main.id
  
  # 説明には以下を含める
  description = "HTTPS from internal network - Ticket: INFRA-1234"
}
```

#### 3. 動的なIP範囲の管理

```hcl
# AWS IPレンジの動的取得
data "aws_ip_ranges" "cloudfront" {
  regions  = ["global"]
  services = ["cloudfront"]
}

# GitHub Actions IPの取得
data "http" "github_meta" {
  url = "https://api.github.com/meta"
  
  request_headers = {
    Accept = "application/vnd.github.v3+json"
  }
}

locals {
  github_actions_ips = jsondecode(data.http.github_meta.response_body).actions
}

# 動的IPレンジの適用
resource "aws_security_group_rule" "github_actions" {
  type              = "ingress"
  from_port         = 443
  to_port           = 443
  protocol          = "tcp"
  cidr_blocks       = local.github_actions_ips
  security_group_id = aws_security_group.api.id
  description       = "HTTPS from GitHub Actions"
}
```

#### 4. モジュール化

```hcl
# modules/security_group/main.tf
resource "aws_security_group" "this" {
  name_prefix = var.name_prefix
  description = var.description
  vpc_id      = var.vpc_id
  
  tags = merge(
    var.tags,
    {
      Name = "${var.name_prefix}sg"
    }
  )
  
  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group_rule" "ingress" {
  for_each = var.ingress_rules
  
  security_group_id = aws_security_group.this.id
  type              = "ingress"
  
  from_port   = each.value.from_port
  to_port     = each.value.to_port
  protocol    = each.value.protocol
  description = each.value.description
  
  cidr_blocks              = lookup(each.value, "cidr_blocks", null)
  ipv6_cidr_blocks         = lookup(each.value, "ipv6_cidr_blocks", null)
  source_security_group_id = lookup(each.value, "source_security_group_id", null)
  self                     = lookup(each.value, "self", null)
}

# modules/security_group/variables.tf
variable "ingress_rules" {
  description = "Map of ingress rules"
  type = map(object({
    from_port                = number
    to_port                  = number
    protocol                 = string
    description              = string
    cidr_blocks              = optional(list(string))
    ipv6_cidr_blocks         = optional(list(string))
    source_security_group_id = optional(string)
    self                     = optional(bool)
  }))
  default = {}
}

# 使用例
module "web_sg" {
  source = "./modules/security_group"
  
  name_prefix = "${var.project_name}-web-"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  ingress_rules = {
    http = {
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP from anywhere"
    }
    https = {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS from anywhere"
    }
  }
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: 循環参照

**エラーメッセージ**:
```
Error: Cycle: aws_security_group.a, aws_security_group.b
```

**原因**: セキュリティグループ間の相互参照

**解決方法**:
```hcl
# ❌ 循環参照を起こす例
resource "aws_security_group" "a" {
  ingress {
    security_groups = [aws_security_group.b.id]
  }
}

resource "aws_security_group" "b" {
  ingress {
    security_groups = [aws_security_group.a.id]
  }
}

# ✅ 解決策：個別ルールリソースを使用
resource "aws_security_group" "a" {
  name = "sg-a"
}

resource "aws_security_group" "b" {
  name = "sg-b"
}

resource "aws_security_group_rule" "a_from_b" {
  type                     = "ingress"
  from_port                = 80
  to_port                  = 80
  protocol                 = "tcp"
  security_group_id        = aws_security_group.a.id
  source_security_group_id = aws_security_group.b.id
}

resource "aws_security_group_rule" "b_from_a" {
  type                     = "ingress"
  from_port                = 80
  to_port                  = 80
  protocol                 = "tcp"
  security_group_id        = aws_security_group.b.id
  source_security_group_id = aws_security_group.a.id
}
```

#### エラー2: ルール数の上限

**エラーメッセージ**:
```
RulesPerSecurityGroupLimitExceeded: The maximum number of rules per security group has been reached
```

**原因**: セキュリティグループあたり60ルール（インバウンド/アウトバウンド各）の制限

**解決方法**:
```hcl
# ルールの集約
# ❌ 個別ポートごとのルール
resource "aws_security_group_rule" "bad" {
  count = 100  # 制限超過
  
  type        = "ingress"
  from_port   = 8000 + count.index
  to_port     = 8000 + count.index
  protocol    = "tcp"
  cidr_blocks = ["10.0.0.0/8"]
}

# ✅ ポート範囲での集約
resource "aws_security_group_rule" "good" {
  type        = "ingress"
  from_port   = 8000
  to_port     = 8099
  protocol    = "tcp"
  cidr_blocks = ["10.0.0.0/8"]
}
```

#### エラー3: 名前の重複

**エラーメッセージ**:
```
InvalidGroup.Duplicate: The security group 'my-sg' already exists
```

**原因**: VPC内でセキュリティグループ名が重複

**解決方法**:
```hcl
# name_prefixを使用して一意性を確保
resource "aws_security_group" "example" {
  name_prefix = "${var.project_name}-${var.environment}-"
  description = "Managed by Terraform"
  vpc_id      = var.vpc_id
  
  lifecycle {
    create_before_destroy = true
  }
}
```

#### エラー4: 不正なプロトコル番号

**エラーメッセージ**:
```
InvalidParameterValue: Invalid protocol
```

**解決方法**:
```hcl
# 正しいプロトコル指定
# TCP: "tcp" または 6
# UDP: "udp" または 17  
# ICMP: "icmp" または 1
# すべて: "-1" または "all"

# カスタムプロトコル番号
resource "aws_security_group_rule" "custom_protocol" {
  type        = "ingress"
  from_port   = 0
  to_port     = 0
  protocol    = "50"  # ESP（IPsec）
  cidr_blocks = ["10.0.0.0/8"]
}
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.08.07.10.15 - what_internal_setting_in_aws_lb.md](../daily/2025.08.07.10.15_what_internal_setting_in_aws_lb.md)
  - ALB用セキュリティグループの設定パターン
- [2025.08.07.11.33 - what_assign_public_ip_in_ecs_network_configuration.md](../daily/2025.08.07.11.33_what_assign_public_ip_in_ecs_network_configuration.md)
  - ECSタスク用セキュリティグループ設定
- [2025.08.04.21.12 - what_difference_between_name_and_tags_name_in_terraform.md](../daily/2025.08.04.21.12_what_difference_between_name_and_tags_name_in_terraform.md)
  - セキュリティグループの命名規則

### Why関連

- [2025.08.04.16.13 - why_efs_not_need_same_subnet_as_fargate_target.md](../daily/2025.08.04.16.13_why_efs_not_need_same_subnet_as_fargate_target.md)
  - マウントターゲットとタスク間のセキュリティグループ設定

### How関連

- [2025.08.04.23.45 - how_terraform_bastion_server_setup.md](../daily/2025.08.04.23.45_how_terraform_bastion_server_setup.md)
  - 踏み台サーバー用セキュリティグループの実装
- [2025.08.07.11.48 - what_aws_rds_cluster_terraform_resource.md](../daily/2025.08.07.11.48_what_aws_rds_cluster_terraform_resource.md)
  - RDSクラスター用セキュリティグループ
- [2025.07.28.16.54 - what_aws_lambda_vpc_access.md](../daily/2025.07.28.16.54_what_aws_lambda_vpc_access.md)
  - Lambda VPCアクセス用セキュリティグループ

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 | 
|-----------|---------|-------------|
| 1.0.0 | 2025-08-11 | 初版作成 |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。
