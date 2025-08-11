# About Terraform aws_vpc Resource

> [!NOTE]
> このドキュメントはTerraform aws_vpcリソースに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [About Terraform aws\_vpc Resource](#about-terraform-aws_vpc-resource)
  - [目次](#目次)
  - [概要](#概要)
    - [キーポイント](#キーポイント)
  - [What - aws\_vpcリソースとは](#what---aws_vpcリソースとは)
    - [基本構文](#基本構文)
      - [リソース名の構成](#リソース名の構成)
    - [主要なパラメータ](#主要なパラメータ)
      - [必須パラメータ](#必須パラメータ)
      - [重要なオプションパラメータ](#重要なオプションパラメータ)
      - [IPv6関連パラメータ](#ipv6関連パラメータ)
      - [IPAM関連パラメータ](#ipam関連パラメータ)
    - [出力属性](#出力属性)
  - [Why - なぜaws\_vpcリソースを使うのか](#why---なぜaws_vpcリソースを使うのか)
    - [解決する課題](#解決する課題)
      - [手動管理の問題点](#手動管理の問題点)
      - [Terraformによる解決](#terraformによる解決)
    - [メリット](#メリット)
    - [注意点](#注意点)
  - [How - aws\_vpcリソースの実装方法](#how---aws_vpcリソースの実装方法)
    - [基本的な実装](#基本的な実装)
      - [シンプルなVPC](#シンプルなvpc)
      - [本番環境向けVPC](#本番環境向けvpc)
    - [高度な実装パターン](#高度な実装パターン)
      - [IPAM統合VPC](#ipam統合vpc)
      - [マルチCIDR VPC](#マルチcidr-vpc)
      - [専用テナンシーVPC](#専用テナンシーvpc)
    - [ベストプラクティス](#ベストプラクティス)
      - [1. 変数化による柔軟性](#1-変数化による柔軟性)
      - [2. タグ管理の標準化](#2-タグ管理の標準化)
      - [3. デフォルトリソースの管理](#3-デフォルトリソースの管理)
    - [トラブルシューティング](#トラブルシューティング)
      - [エラー1: CIDR範囲の競合](#エラー1-cidr範囲の競合)
      - [エラー2: DNS設定の問題](#エラー2-dns設定の問題)
      - [エラー3: 削除時のエラー](#エラー3-削除時のエラー)
  - [参照：daily-TIL](#参照daily-til)
    - [What関連](#what関連)
    - [Why関連](#why関連)
  - [バージョン履歴](#バージョン履歴)

</details>

---

## 概要

aws_vpcリソースは、AWS Virtual Private Cloud (VPC)を作成・管理するためのTerraformリソースです。ネットワークの基盤となる仮想プライベートクラウドを定義し、IPアドレス範囲、DNS設定、テナンシーモードなどを構成します。

### キーポイント

- **ネットワーク基盤**: AWSリソースを配置する論理的に分離されたネットワーク空間
- **CIDR管理**: IPv4/IPv6アドレス空間の定義と管理
- **DNS統合**: Route 53との統合によるDNS名前解決

---

## What - aws_vpcリソースとは

### 基本構文

<details>
<summary>基本構文の詳細</summary>

```hcl
resource "aws_vpc" "main" {
  # 必須パラメータ
  cidr_block = "10.0.0.0/16"
  
  # オプションパラメータ
  instance_tenancy                 = "default"
  enable_dns_support               = true
  enable_dns_hostnames             = true
  enable_network_address_usage_metrics = false
  
  # IPv6設定
  assign_generated_ipv6_cidr_block = false
  
  # タグ
  tags = {
    Name = "my-vpc"
  }
}
```

#### リソース名の構成

- **リソースタイプ**: `aws_vpc`
- **リソース名**: 任意の識別子（例: `main`, `production`, `development`）
- **参照方法**: `aws_vpc.main.id`, `aws_vpc.main.cidr_block`

</details>

### 主要なパラメータ

<details>
<summary>パラメータの詳細説明</summary>

#### 必須パラメータ

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `cidr_block` | VPCのIPv4 CIDRブロック | `"10.0.0.0/16"` |

#### 重要なオプションパラメータ

| パラメータ | デフォルト | 説明 |
|-----------|-----------|------|
| `instance_tenancy` | `"default"` | インスタンスのテナンシー（`default`, `dedicated`, `host`） |
| `enable_dns_support` | `true` | DNS解決のサポート |
| `enable_dns_hostnames` | `false` | DNSホスト名の割り当て |
| `enable_network_address_usage_metrics` | `false` | ネットワークアドレス使用状況メトリクスの有効化 |

#### IPv6関連パラメータ

| パラメータ | 説明 |
|-----------|------|
| `assign_generated_ipv6_cidr_block` | Amazon提供のIPv6 CIDRブロックを割り当て |
| `ipv6_cidr_block` | カスタムIPv6 CIDRブロック（BYOIPの場合） |
| `ipv6_ipam_pool_id` | IPAM IPv6プールID |
| `ipv6_netmask_length` | IPAMプールからのネットマスク長 |

#### IPAM関連パラメータ

| パラメータ | 説明 |
|-----------|------|
| `ipv4_ipam_pool_id` | IPAM IPv4プールID |
| `ipv4_netmask_length` | IPAMプールからのネットマスク長（16-28） |

</details>

### 出力属性

<details>
<summary>出力属性の詳細</summary>

| 属性 | 説明 | 使用例 |
|------|------|--------|
| `id` | VPCのID | `aws_vpc.main.id` |
| `arn` | VPCのARN | `aws_vpc.main.arn` |
| `cidr_block` | VPCのCIDRブロック | `aws_vpc.main.cidr_block` |
| `default_security_group_id` | デフォルトセキュリティグループID | `aws_vpc.main.default_security_group_id` |
| `default_network_acl_id` | デフォルトネットワークACL ID | `aws_vpc.main.default_network_acl_id` |
| `default_route_table_id` | デフォルトルートテーブルID | `aws_vpc.main.default_route_table_id` |
| `ipv6_association_id` | IPv6 CIDR関連付けID | `aws_vpc.main.ipv6_association_id` |
| `ipv6_cidr_block` | IPv6 CIDRブロック | `aws_vpc.main.ipv6_cidr_block` |
| `main_route_table_id` | メインルートテーブルID | `aws_vpc.main.main_route_table_id` |
| `owner_id` | VPC所有者のAWSアカウントID | `aws_vpc.main.owner_id` |

</details>

---

## Why - なぜaws_vpcリソースを使うのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### 手動管理の問題点

1. **一貫性の欠如**
   - 環境間での設定差異
   - 命名規則の不統一

2. **変更追跡の困難さ**
   - 設定変更の履歴管理
   - 監査証跡の不足

3. **再現性の問題**
   - 災害復旧時の再構築
   - 新環境の構築時間

#### Terraformによる解決

- **Infrastructure as Code**: 設定をコードとして管理
- **バージョン管理**: Gitによる変更履歴の追跡
- **自動化**: 一貫した環境構築の実現

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

1. **宣言的な設定**
   - 望ましい状態を記述するだけ
   - Terraformが現在の状態との差分を計算

2. **依存関係の自動解決**
   - サブネット、ルートテーブルなどの関連リソースとの依存関係
   - 正しい順序でのリソース作成・削除

3. **環境の複製**
   - 開発・ステージング・本番環境の統一的な管理
   - 変数による環境別の設定

4. **ドリフト検出**
   - 手動変更の検出
   - 設定の一貫性維持

</details>

### 注意点

<details>
<summary>注意点と対策</summary>

| 注意点 | 影響 | 対策 |
|--------|------|------|
| CIDR変更不可 | VPC作成後のCIDR変更は再作成が必要 | 初期設計を慎重に |
| デフォルトリソース | 自動作成されるリソースの管理 | 明示的に管理または無効化 |
| リージョン制限 | VPCはリージョンに固定 | マルチリージョン設計の考慮 |

</details>

---

## How - aws_vpcリソースの実装方法

### 基本的な実装

<details>
<summary>基本実装例</summary>

#### シンプルなVPC

```hcl
# 最小構成のVPC
resource "aws_vpc" "simple" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "simple-vpc"
  }
}
```

#### 本番環境向けVPC

```hcl
# プロダクション環境向けの設定
resource "aws_vpc" "production" {
  cidr_block           = var.vpc_cidr
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true
  
  # IPv6有効化
  assign_generated_ipv6_cidr_block = true
  
  # ネットワーク使用状況メトリクス有効化
  enable_network_address_usage_metrics = true
  
  tags = {
    Name        = "${var.project_name}-vpc-${var.environment}"
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

</details>

### 高度な実装パターン

<details>
<summary>高度な実装例</summary>

#### IPAM統合VPC

```hcl
# IPAMプールからのCIDR割り当て
resource "aws_vpc" "ipam_managed" {
  ipv4_ipam_pool_id   = var.ipam_pool_id
  ipv4_netmask_length = 24
  
  # IPv6もIPAMから割り当て
  ipv6_ipam_pool_id   = var.ipv6_ipam_pool_id
  ipv6_netmask_length = 56
  
  enable_dns_support   = true
  enable_dns_hostnames = true
  
  tags = {
    Name = "${var.project_name}-ipam-vpc"
  }
}
```

#### マルチCIDR VPC

```hcl
# プライマリVPC
resource "aws_vpc" "multi_cidr" {
  cidr_block = "10.0.0.0/16"
  
  enable_dns_support   = true
  enable_dns_hostnames = true
  
  tags = {
    Name = "multi-cidr-vpc"
  }
}

# 追加のCIDRブロック
resource "aws_vpc_ipv4_cidr_block_association" "secondary" {
  vpc_id     = aws_vpc.multi_cidr.id
  cidr_block = "10.1.0.0/16"
}

# 別の追加CIDR（IPAMから）
resource "aws_vpc_ipv4_cidr_block_association" "from_ipam" {
  vpc_id              = aws_vpc.multi_cidr.id
  ipv4_ipam_pool_id   = var.secondary_ipam_pool_id
  ipv4_netmask_length = 24
}
```

#### 専用テナンシーVPC

```hcl
# 専用ハードウェアでの実行が必要な場合
resource "aws_vpc" "dedicated" {
  cidr_block       = "10.0.0.0/16"
  instance_tenancy = "dedicated"
  
  tags = {
    Name = "dedicated-vpc"
    Note = "All instances will run on dedicated hardware"
  }
}
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. 変数化による柔軟性

```hcl
variable "vpc_config" {
  description = "VPC configuration"
  type = object({
    cidr_block           = string
    enable_dns_support   = bool
    enable_dns_hostnames = bool
    enable_ipv6          = bool
  })
  
  default = {
    cidr_block           = "10.0.0.0/16"
    enable_dns_support   = true
    enable_dns_hostnames = true
    enable_ipv6          = false
  }
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_config.cidr_block
  enable_dns_support   = var.vpc_config.enable_dns_support
  enable_dns_hostnames = var.vpc_config.enable_dns_hostnames
  
  assign_generated_ipv6_cidr_block = var.vpc_config.enable_ipv6
  
  tags = local.common_tags
}
```

#### 2. タグ管理の標準化

```hcl
locals {
  common_tags = {
    Name        = "${var.project_name}-vpc-${var.environment}"
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    CreatedAt   = timestamp()
  }
}
```

#### 3. デフォルトリソースの管理

```hcl
# デフォルトセキュリティグループの管理
resource "aws_default_security_group" "default" {
  vpc_id = aws_vpc.main.id
  
  # すべてのトラフィックを拒否
  # （明示的なルールなし）
  
  tags = {
    Name = "${var.project_name}-default-sg"
    Note = "Managed by Terraform - No rules allowed"
  }
}

# デフォルトネットワークACLの管理
resource "aws_default_network_acl" "default" {
  default_network_acl_id = aws_vpc.main.default_network_acl_id
  
  # 基本的なルールを定義
  ingress {
    protocol   = -1
    rule_no    = 100
    action     = "allow"
    cidr_block = aws_vpc.main.cidr_block
    from_port  = 0
    to_port    = 0
  }
  
  egress {
    protocol   = -1
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }
  
  tags = {
    Name = "${var.project_name}-default-nacl"
  }
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: CIDR範囲の競合

**原因**: 既存のVPCまたはVPC Peeringと競合
**解決方法**:

```bash
# 既存VPCのCIDRを確認
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock]' --output table

# 使用可能なCIDR範囲を計画
# プライベートIPアドレス範囲:
# 10.0.0.0/8 (10.0.0.0 - 10.255.255.255)
# 172.16.0.0/12 (172.16.0.0 - 172.31.255.255)
# 192.168.0.0/16 (192.168.0.0 - 192.168.255.255)
```

#### エラー2: DNS設定の問題

**原因**: enable_dns_supportまたはenable_dns_hostnamesが無効
**解決方法**:

```hcl
# 両方を有効化
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true  # 必須
  enable_dns_hostnames = true  # EC2のDNS名が必要な場合
}
```

#### エラー3: 削除時のエラー

**原因**: VPC内にリソースが残っている
**解決方法**:

```bash
# VPC内のリソースを確認
aws ec2 describe-vpc-attribute --vpc-id vpc-xxxxx --attribute enableDnsSupport

# 依存リソースの確認
terraform state list | grep vpc-xxxxx

# 強制削除（注意が必要）
terraform destroy -target=aws_vpc.main
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.08.04.20.18 - what_terraform_aws_vpc_official_complete_reference.md](../daily/2025.08.04.20.18_what_terraform_aws_vpc_official_complete_reference.md)
  - Terraform aws_vpcリソースの公式完全リファレンス
- [2025.08.04.21.28 - what_enable_dns_hostnames_in_terraform_aws_vpc.md](../daily/2025.08.04.21.28_what_enable_dns_hostnames_in_terraform_aws_vpc.md)
  - enable_dns_hostnamesパラメータの詳細
- [2025.08.07.07.44 - what_enable_dns_support_in_aws_vpc.md](../daily/2025.08.07.07.44_what_enable_dns_support_in_aws_vpc.md)
  - enable_dns_supportパラメータの詳細
- [2025.08.07.07.38 - what_instance_tenancy_in_aws_vpc.md](../daily/2025.08.07.07.38_what_instance_tenancy_in_aws_vpc.md)
  - instance_tenancyパラメータの詳細

### Why関連

- [2025.08.04.21.19 - why_tags_name_should_distinct_by_environment.md](../daily/2025.08.04.21.19_why_tags_name_should_distinct_by_environment.md)
  - 環境別タグ付けの重要性
- [2025.08.04.21.12 - what_difference_between_name_and_tags_name_in_terraform.md](../daily/2025.08.04.21.12_what_difference_between_name_and_tags_name_in_terraform.md)
  - リソース名とタグ名の違い

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 |
|-----------|---------|-------------|
| 1.0.0 | 2025-08-11 | 初版作成 |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。
