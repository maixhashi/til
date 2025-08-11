# About Terraform aws_subnet Resource

> [!NOTE]
> このドキュメントはTerraform aws_subnetリソースに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [About Terraform aws\_subnet Resource](#about-terraform-aws_subnet-resource)
  - [目次](#目次)
  - [概要](#概要)
    - [キーポイント](#キーポイント)
  - [What - aws\_subnetリソースとは](#what---aws_subnetリソースとは)
    - [基本構文](#基本構文)
      - [リソース名の構成](#リソース名の構成)
    - [主要なパラメータ](#主要なパラメータ)
      - [必須パラメータ](#必須パラメータ)
      - [重要なオプションパラメータ](#重要なオプションパラメータ)
      - [ネットワーク設定パラメータ](#ネットワーク設定パラメータ)
      - [DNS設定パラメータ](#dns設定パラメータ)
    - [出力属性](#出力属性)
  - [Why - なぜaws\_subnetリソースを使うのか](#why---なぜaws_subnetリソースを使うのか)
    - [解決する課題](#解決する課題)
      - [ネットワーク設計の複雑性](#ネットワーク設計の複雑性)
      - [Terraformによる解決](#terraformによる解決)
    - [メリット](#メリット)
    - [注意点](#注意点)
  - [How - aws\_subnetリソースの実装方法](#how---aws_subnetリソースの実装方法)
    - [基本的な実装](#基本的な実装)
      - [シンプルなサブネット](#シンプルなサブネット)
      - [パブリック/プライベートサブネット](#パブリックプライベートサブネット)
    - [高度な実装パターン](#高度な実装パターン)
      - [マルチAZ構成](#マルチaz構成)
      - [IPv6対応サブネット](#ipv6対応サブネット)
      - [Outpost対応サブネット](#outpost対応サブネット)
    - [ベストプラクティス](#ベストプラクティス)
      - [1. CIDR計画の標準化](#1-cidr計画の標準化)
      - [2. タグ戦略](#2-タグ戦略)
      - [3. セキュリティグループとの統合](#3-セキュリティグループとの統合)
      - [4. ルートテーブルの自動関連付け](#4-ルートテーブルの自動関連付け)
    - [トラブルシューティング](#トラブルシューティング)
      - [エラー1: CIDR範囲の競合](#エラー1-cidr範囲の競合)
      - [エラー2: 利用可能なIPアドレス不足](#エラー2-利用可能なipアドレス不足)
      - [エラー3: AZが利用不可](#エラー3-azが利用不可)
      - [エラー4: サブネット削除エラー](#エラー4-サブネット削除エラー)
  - [参照：daily-TIL](#参照daily-til)
    - [What関連](#what関連)
    - [Why関連](#why関連)
    - [How関連](#how関連)
  - [バージョン履歴](#バージョン履歴)

</details>

---

## 概要

aws_subnetリソースは、AWS VPC内にサブネットを作成・管理するためのTerraformリソースです。アベイラビリティーゾーン（AZ）ごとにネットワークセグメントを分割し、パブリック/プライベートなネットワーク層を構築します。

### キーポイント

- **ネットワーク分割**: VPC内のIPアドレス空間を論理的に分割
- **AZ分散**: 高可用性のためのマルチAZ構成
- **セキュリティ層**: パブリック/プライベート/データベース層の分離

---

## What - aws_subnetリソースとは

### 基本構文

<details>
<summary>基本構文の詳細</summary>

```hcl
resource "aws_subnet" "example" {
  # 必須パラメータ
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  
  # オプションパラメータ（重要）
  availability_zone               = "us-east-1a"
  map_public_ip_on_launch         = false
  assign_ipv6_address_on_creation = false
  
  # IPv6設定
  ipv6_cidr_block                 = cidrsubnet(aws_vpc.main.ipv6_cidr_block, 8, 1)
  enable_dns64                    = false
  ipv6_native                     = false
  
  # リソースマップ設定
  enable_resource_name_dns_a_record_on_launch    = false
  enable_resource_name_dns_aaaa_record_on_launch = false
  
  # タグ
  tags = {
    Name = "my-subnet"
    Type = "private"
  }
}
```

#### リソース名の構成

- **リソースタイプ**: `aws_subnet`
- **リソース名**: 任意の識別子（例: `public`, `private`, `database`）
- **参照方法**: `aws_subnet.example.id`, `aws_subnet.example.cidr_block`

</details>

### 主要なパラメータ

<details>
<summary>パラメータの詳細説明</summary>

#### 必須パラメータ

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `vpc_id` | サブネットを作成するVPCのID | `aws_vpc.main.id` |
| `cidr_block` または `ipv6_cidr_block` | サブネットのIPアドレス範囲 | `"10.0.1.0/24"` |

#### 重要なオプションパラメータ

| パラメータ | デフォルト | 説明 |
|-----------|-----------|------|
| `availability_zone` | ランダム | サブネットを配置するAZ |
| `availability_zone_id` | - | AZの安定したID（推奨） |
| `map_public_ip_on_launch` | `false` | 起動時にパブリックIPを自動割り当て |
| `assign_ipv6_address_on_creation` | `false` | 起動時にIPv6アドレスを自動割り当て |

#### ネットワーク設定パラメータ

| パラメータ | 説明 |
|-----------|------|
| `customer_owned_ipv4_pool` | Outpostで使用するカスタマー所有のIPv4プール |
| `map_customer_owned_ip_on_launch` | カスタマー所有IPの自動割り当て |
| `outpost_arn` | AWS OutpostのARN |

#### DNS設定パラメータ

| パラメータ | 説明 |
|-----------|------|
| `enable_dns64` | DNS64サポートの有効化（IPv6のみ） |
| `enable_resource_name_dns_a_record_on_launch` | EC2インスタンスのAレコード自動作成 |
| `enable_resource_name_dns_aaaa_record_on_launch` | EC2インスタンスのAAAAレコード自動作成 |
| `private_dns_hostname_type_on_launch` | プライベートDNSホスト名のタイプ |

</details>

### 出力属性

<details>
<summary>出力属性の詳細</summary>

| 属性 | 説明 | 使用例 |
|------|------|--------|
| `id` | サブネットのID | `aws_subnet.example.id` |
| `arn` | サブネットのARN | `aws_subnet.example.arn` |
| `ipv6_cidr_block_association_id` | IPv6 CIDR関連付けID | `aws_subnet.example.ipv6_cidr_block_association_id` |
| `owner_id` | サブネット所有者のAWSアカウントID | `aws_subnet.example.owner_id` |
| `available_ip_address_count` | 利用可能なIPアドレス数 | `aws_subnet.example.available_ip_address_count` |

</details>

---

## Why - なぜaws_subnetリソースを使うのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### ネットワーク設計の複雑性

1. **手動設定の問題**
   - CIDRブロックの計算ミス
   - AZ配置の不整合
   - 環境間での設定差異

2. **セキュリティ境界の曖昧さ**
   - パブリック/プライベート層の混在
   - 不適切なルーティング設定
   - セキュリティグループの管理困難

3. **可用性の課題**
   - 単一AZへの依存
   - 障害時の影響範囲

#### Terraformによる解決

```hcl
# マルチAZ構成の自動化
variable "availability_zones" {
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

resource "aws_subnet" "public" {
  count = length(var.availability_zones)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = var.availability_zones[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${var.availability_zones[count.index]}"
  }
}
```

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

1. **自動化されたネットワーク設計**
   - CIDR計算の自動化（`cidrsubnet`関数）
   - 一貫したAZ配置
   - 環境間での再現性

2. **セキュリティの向上**
   - 明確なネットワーク層の分離
   - タグベースのアクセス制御
   - 監査証跡の自動化

3. **高可用性の実現**
   - マルチAZ展開の簡素化
   - 障害時の自動フェイルオーバー
   - リソースの均等分散

4. **運用効率の改善**
   - 設定のコード化
   - 変更管理の容易性
   - ドキュメントとしてのコード

</details>

### 注意点

<details>
<summary>注意点と対策</summary>

| 注意点 | 影響 | 対策 |
|--------|------|------|
| CIDR変更不可 | サブネット作成後のCIDR変更は再作成が必要 | 初期設計を慎重に |
| AZ固定 | 作成後のAZ変更は不可 | AZ IDの使用を検討 |
| IPアドレス予約 | 各サブネットで5つのIPが予約される | 適切なサイズ計画 |
| 削除順序 | ENIが存在すると削除不可 | 依存関係の管理 |

</details>

---

## How - aws_subnetリソースの実装方法

### 基本的な実装

<details>
<summary>基本実装例</summary>

#### シンプルなサブネット

```hcl
# 単一のプライベートサブネット
resource "aws_subnet" "simple" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "simple-subnet"
  }
}
```

#### パブリック/プライベートサブネット

```hcl
# パブリックサブネット
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.0.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-1a"
    Type = "public"
  }
}

# プライベートサブネット
resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.10.0/24"
  availability_zone = "us-east-1a"
  
  tags = {
    Name = "private-subnet-1a"
    Type = "private"
  }
}

# データベースサブネット
resource "aws_subnet" "database" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.20.0/24"
  availability_zone = "us-east-1a"
  
  tags = {
    Name = "database-subnet-1a"
    Type = "database"
  }
}
```

</details>

### 高度な実装パターン

<details>
<summary>高度な実装例</summary>

#### マルチAZ構成

```hcl
# 利用可能なAZを取得
data "aws_availability_zones" "available" {
  state = "available"
}

# ローカル変数で設定を定義
locals {
  azs = slice(data.aws_availability_zones.available.names, 0, 3)
  
  subnet_config = {
    public = {
      cidr_newbits = 4
      cidr_offset  = 0
      public_ip    = true
    }
    private = {
      cidr_newbits = 4
      cidr_offset  = 4
      public_ip    = false
    }
    database = {
      cidr_newbits = 4
      cidr_offset  = 8
      public_ip    = false
    }
  }
}

# 動的サブネット作成
resource "aws_subnet" "subnets" {
  for_each = {
    for item in flatten([
      for type, config in local.subnet_config : [
        for idx, az in local.azs : {
          key        = "${type}-${az}"
          type       = type
          az         = az
          az_index   = idx
          config     = config
        }
      ]
    ]) : item.key => item
  }
  
  vpc_id            = aws_vpc.main.id
  availability_zone = each.value.az
  
  cidr_block = cidrsubnet(
    aws_vpc.main.cidr_block,
    each.value.config.cidr_newbits,
    each.value.config.cidr_offset + each.value.az_index
  )
  
  map_public_ip_on_launch = each.value.config.public_ip
  
  tags = {
    Name = "${var.project_name}-${each.value.type}-${each.value.az}"
    Type = each.value.type
    AZ   = each.value.az
  }
}

# 出力の整理
output "subnet_ids" {
  value = {
    public   = [for k, v in aws_subnet.subnets : v.id if can(regex("^public-", k))]
    private  = [for k, v in aws_subnet.subnets : v.id if can(regex("^private-", k))]
    database = [for k, v in aws_subnet.subnets : v.id if can(regex("^database-", k))]
  }
}
```

#### IPv6対応サブネット

```hcl
# IPv6対応サブネット
resource "aws_subnet" "ipv6" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  
  # IPv6設定
  ipv6_cidr_block                 = cidrsubnet(aws_vpc.main.ipv6_cidr_block, 8, 1)
  assign_ipv6_address_on_creation = true
  
  # DNS64有効化（IPv6のみの環境でIPv4リソースへのアクセス）
  enable_dns64 = true
  
  # リソース名DNSレコード
  enable_resource_name_dns_aaaa_record_on_launch = true
  
  tags = {
    Name = "ipv6-enabled-subnet"
  }
}
```

#### Outpost対応サブネット

```hcl
# AWS Outpost用サブネット
resource "aws_subnet" "outpost" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.100.0/24"
  
  # Outpost設定
  outpost_arn                      = var.outpost_arn
  customer_owned_ipv4_pool         = var.customer_owned_pool
  map_customer_owned_ip_on_launch  = true
  
  tags = {
    Name = "outpost-subnet"
    Type = "outpost"
  }
}
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. CIDR計画の標準化

```hcl
# CIDR計画の変数化
variable "subnet_cidr_config" {
  description = "Subnet CIDR configuration"
  type = object({
    vpc_cidr     = string
    newbits      = number
    az_count     = number
    subnet_types = list(string)
  })
  
  default = {
    vpc_cidr     = "10.0.0.0/16"
    newbits      = 8  # /24サブネット
    az_count     = 3
    subnet_types = ["public", "private", "database"]
  }
}

# CIDR計算の自動化
locals {
  subnet_cidrs = {
    for idx, subnet in setproduct(var.subnet_cidr_config.subnet_types, range(var.subnet_cidr_config.az_count)) : 
    "${subnet[0]}-${subnet[1]}" => cidrsubnet(
      var.subnet_cidr_config.vpc_cidr,
      var.subnet_cidr_config.newbits,
      idx
    )
  }
}
```

#### 2. タグ戦略

```hcl
# 共通タグの定義
locals {
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    CreatedAt   = timestamp()
  }
  
  subnet_tags = {
    public = {
      Tier           = "public"
      InternetAccess = "true"
    }
    private = {
      Tier           = "private"
      InternetAccess = "false"
    }
    database = {
      Tier           = "database"
      InternetAccess = "false"
    }
  }
}

# タグの適用
resource "aws_subnet" "main" {
  # ... 他の設定 ...
  
  tags = merge(
    local.common_tags,
    local.subnet_tags[var.subnet_type],
    {
      Name = "${var.project_name}-${var.subnet_type}-${var.availability_zone}"
    }
  )
}
```

#### 3. セキュリティグループとの統合

```hcl
# サブネットタイプ別のデフォルトセキュリティグループ
resource "aws_security_group" "subnet_default" {
  for_each = toset(var.subnet_cidr_config.subnet_types)
  
  name_prefix = "${var.project_name}-${each.key}-default-"
  description = "Default security group for ${each.key} subnets"
  vpc_id      = aws_vpc.main.id
  
  # タイプ別のルール
  dynamic "ingress" {
    for_each = each.key == "public" ? [1] : []
    content {
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS from anywhere"
    }
  }
  
  dynamic "ingress" {
    for_each = each.key == "private" ? [1] : []
    content {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = [aws_vpc.main.cidr_block]
      description = "All traffic from VPC"
    }
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
    description = "All outbound traffic"
  }
  
  tags = {
    Name = "${var.project_name}-${each.key}-default-sg"
    Type = each.key
  }
}
```

#### 4. ルートテーブルの自動関連付け

```hcl
# サブネットタイプ別ルートテーブル
resource "aws_route_table" "main" {
  for_each = toset(var.subnet_cidr_config.subnet_types)
  
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-${each.key}-rt"
    Type = each.key
  }
}

# パブリックサブネット用のインターネットゲートウェイルート
resource "aws_route" "public_internet" {
  for_each = {
    for rt_key, rt in aws_route_table.main : rt_key => rt
    if rt_key == "public"
  }
  
  route_table_id         = each.value.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.main.id
}

# サブネットとルートテーブルの関連付け
resource "aws_route_table_association" "main" {
  for_each = aws_subnet.subnets
  
  subnet_id      = each.value.id
  route_table_id = aws_route_table.main[split("-", each.key)[0]].id
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: CIDR範囲の競合

**エラーメッセージ**:
```
Error: Error creating subnet: InvalidSubnet.Conflict: The CIDR '10.0.1.0/24' conflicts with another subnet
```

**原因**: 既存のサブネットとCIDRが重複

**解決方法**:
```bash
# 既存サブネットのCIDRを確認
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-xxxxx" \
  --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]' \
  --output table

# Terraformでの確認
terraform state list | grep aws_subnet
terraform state show aws_subnet.existing
```

#### エラー2: 利用可能なIPアドレス不足

**原因**: サブネットサイズが小さすぎる

**解決方法**:
```hcl
# サブネットサイズの計算
# /28 = 16 IPs - 5 reserved = 11 usable
# /27 = 32 IPs - 5 reserved = 27 usable  
# /24 = 256 IPs - 5 reserved = 251 usable

# 適切なサイズの選択
resource "aws_subnet" "app" {
  vpc_id     = aws_vpc.main.id
  cidr_block = cidrsubnet(aws_vpc.main.cidr_block, 4, 0)  # /20 from /16
}
```

#### エラー3: AZが利用不可

**エラーメッセージ**:
```
Error: Error creating subnet: InvalidParameterValue: The availability zone 'us-east-1e' is not supported
```

**原因**: 指定したAZがアカウントで利用不可

**解決方法**:
```hcl
# 利用可能なAZを動的に取得
data "aws_availability_zones" "available" {
  state = "available"
  
  # 特定のAZを除外
  exclude_names = ["us-east-1e"]
}

resource "aws_subnet" "main" {
  availability_zone = data.aws_availability_zones.available.names[0]
  # ...
}
```

#### エラー4: サブネット削除エラー

**エラーメッセージ**:
```
Error: Error deleting subnet: DependencyViolation: The subnet has dependencies and cannot be deleted
```

**原因**: サブネット内にENIが存在

**解決方法**:
```bash
# 依存リソースの確認
aws ec2 describe-network-interfaces \
  --filters "Name=subnet-id,Values=subnet-xxxxx" \
  --query 'NetworkInterfaces[*].[NetworkInterfaceId,Description,Status]' \
  --output table

# 強制削除（注意が必要）
terraform destroy -target=aws_instance.example
terraform destroy -target=aws_subnet.example
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.08.04.21.34 - what_terraform_aws_subnet_complete_reference.md](../daily/2025.08.04.21.34_what_terraform_aws_subnet_complete_reference.md)
  - aws_subnetリソースの完全リファレンス
- [2025.08.07.07.47 - what_map_public_ip_on_launch_in_aws_subnet.md](../daily/2025.08.07.07.47_what_map_public_ip_on_launch_in_aws_subnet.md)
  - map_public_ip_on_launch設定の詳細
- [2025.08.07.09.30 - what_route_table_association.md](../daily/2025.08.07.09.30_what_route_table_association.md)
  - サブネットとルートテーブルの関連付け

### Why関連

- [2025.08.04.15.34 - why_alb_must_assign_in_public_subnet.md](../daily/2025.08.04.15.34_why_alb_must_assign_in_public_subnet.md)
  - ALBのパブリックサブネット配置要件
- [2025.08.04.15.08 - why_s3_assigned_outside_of_subnet.md](../daily/2025.08.04.15.08_why_s3_assigned_outside_of_subnet.md)
  - マネージドサービスとサブネットの関係

### How関連

- [2025.08.04.15.27 - how_connect_between_private_subnets.md](../daily/2025.08.04.15.27_how_connect_between_private_subnets.md)
  - プライベートサブネット間の接続方法
- [2025.08.07.09.47 - how_aws_route_table_settings_in_this_project.md](../daily/2025.08.07.09.47_how_aws_route_table_settings_in_this_project.md)
  - 実プロジェクトでのルートテーブル設定

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 |
|-----------|---------|-------------|
| 1.0.0 | 2025-08-11 | 初版作成 |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。
