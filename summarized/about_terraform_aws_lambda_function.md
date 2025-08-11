# About aws_lambda_function
<!-- このファイルはaws_lambda_functionに関する包括的な知識をまとめたものです -->
<!-- daily-TILから重要な内容を抽出・整理し、体系的にまとめています -->

> [!NOTE]
> このドキュメントはaws_lambda_functionに関する学習内容を体系的にまとめた要約版です。
> 詳細な実装例や日々の学習記録は、参照セクションのdaily-TILリンクをご確認ください。

## 目次

<details>
<summary>目次を開く</summary>

- [About aws_lambda_function](#about-aws_lambda_function)
  - [目次](#目次)
  - [概要](#概要)
    - [キーポイント](#キーポイント)
  - [What - aws_lambda_functionとは何か](#what---aws_lambda_functionとは何か)
    - [基本概念](#基本概念)
      - [定義](#定義)
      - [構成要素](#構成要素)
    - [主要な特徴](#主要な特徴)
    - [アーキテクチャ](#アーキテクチャ)
      - [レイヤー構成](#レイヤー構成)
      - [データフロー](#データフロー)
  - [Why - なぜaws_lambda_functionが必要なのか](#why---なぜaws_lambda_functionが必要なのか)
    - [解決する課題](#解決する課題)
      - [従来の問題点](#従来の問題点)
      - [aws_lambda_functionによる解決策](#aws_lambda_functionによる解決策)
    - [メリット](#メリット)
      - [ビジネス面のメリット](#ビジネス面のメリット)
      - [技術面のメリット](#技術面のメリット)
    - [デメリット](#デメリット)
    - [他の選択肢との比較](#他の選択肢との比較)
  - [How - aws_lambda_functionの実装方法](#how---aws_lambda_functionの実装方法)
    - [基本的な使い方](#基本的な使い方)
      - [セットアップ](#セットアップ)
      - [基本的な実装](#基本的な実装)
      - [実行例](#実行例)
    - [ベストプラクティス](#ベストプラクティス)
      - [1. コールドスタート対策](#1-コールドスタート対策)
      - [2. セキュアな環境変数管理](#2-セキュアな環境変数管理)
      - [3. 適切なエラーハンドリング](#3-適切なエラーハンドリング)
    - [よくある実装パターン](#よくある実装パターン)
      - [パターン1: API バックエンド](#パターン1-api-バックエンド)
      - [パターン2: S3イベント処理](#パターン2-s3イベント処理)
      - [パターン3: 定期実行バッチ](#パターン3-定期実行バッチ)
    - [トラブルシューティング](#トラブルシューティング)
      - [エラー1: InvalidParameterValueException](#エラー1-invalidparametervalueexception)
      - [エラー2: ResourceConflictException](#エラー2-resourceconflictexception)
      - [エラー3: CodeStorageExceededException](#エラー3-codestorageexceededexception)
  - [参照：daily-TIL](#参照daily-til)
    - [What関連](#what関連)
    - [Why関連](#why関連)
    - [How関連](#how関連)
  - [バージョン履歴](#バージョン履歴)

</details>

---

## 概要

aws_lambda_functionはTerraformでAWS Lambdaファンクションを作成・管理するためのリソースです。サーバーレスコンピューティングサービスであるLambdaを使用して、インフラストラクチャの管理なしでコードを実行でき、イベントドリブンなアプリケーションの構築を可能にします。

### キーポイント

- サーバーレスファンクションの宣言的な構成管理
- イベントドリブンアーキテクチャの実現
- 使用した分だけの従量課金による効率的なコスト管理

---

## What - aws_lambda_functionとは何か

### 基本概念

<details>
<summary>基本概念の詳細</summary>

aws_lambda_functionリソースは、AWS Lambda関数をTerraformで管理するためのリソースタイプです。サーバーのプロビジョニングや管理なしでコードを実行でき、必要な時にのみ起動して処理を行うサーバーレスコンピューティングを実現します。

#### 定義

aws_lambda_functionは、イベントに応答してコードを実行するサーバーレス関数を定義するTerraformリソースです。様々なAWSサービスやHTTPリクエストをトリガーとして、自動的にスケールしながらコードを実行します。

#### 構成要素

1. **関数コード**
   - 実行するアプリケーションロジック

2. **ランタイム環境**
   - Node.js、Python、Java、Go等のサポート

3. **実行設定**
   - メモリ、タイムアウト、環境変数など

</details>

### 主要な特徴

<details>
<summary>特徴の詳細</summary>

1. **自動スケーリング**
   - 同時実行数に基づいて自動的にスケール
   - 利点: トラフィックに応じた柔軟な対応

2. **イベントドリブン実行**
   - 多様なトリガーによる自動実行
   - 利点: リアクティブなアプリケーション構築

3. **従量課金モデル**
   - 実行時間とメモリ使用量に基づく課金
   - 利点: アイドル時のコストゼロ

</details>

### アーキテクチャ

<details>
<summary>アーキテクチャ図と説明</summary>

```mermaid
graph TB
    subgraph "Lambda Function Architecture"
        subgraph "Event Sources"
            API[API Gateway]
            S3[S3 Bucket]
            SQS[SQS Queue]
            Schedule[EventBridge Schedule]
            DDB[DynamoDB Streams]
        end
        
        subgraph "Lambda Service"
            LambdaFunc[Lambda Function]
            Runtime[Runtime Environment]
            Handler[Handler Code]
            
            subgraph "Configuration"
                Memory[Memory: 128MB-10GB]
                Timeout[Timeout: 1s-15min]
                EnvVars[Environment Variables]
                Layers[Lambda Layers]
            end
        end
        
        subgraph "Security"
            ExecutionRole[Execution Role]
            KMS[KMS Encryption]
            VPC[VPC Config]
        end
        
        subgraph "Destinations"
            CloudWatch[CloudWatch Logs]
            DLQ[Dead Letter Queue]
            Destinations[Event Destinations]
        end
    end
    
    API -->|Sync| LambdaFunc
    S3 -->|Async| LambdaFunc
    SQS -->|Poll| LambdaFunc
    Schedule -->|Trigger| LambdaFunc
    DDB -->|Stream| LambdaFunc
    
    LambdaFunc --> Runtime
    Runtime --> Handler
    
    ExecutionRole --> LambdaFunc
    KMS --> EnvVars
    VPC --> LambdaFunc
    
    LambdaFunc --> CloudWatch
    LambdaFunc -->|Failed| DLQ
    LambdaFunc -->|Success/Failure| Destinations
    
    style LambdaFunc fill:#FFE4B5
    style ExecutionRole fill:#90EE90
    style Handler fill:#87CEEB
```

#### レイヤー構成

- **イベントソース層**: 関数をトリガーする各種サービス
- **Lambda実行層**: 関数の実行環境と設定
- **セキュリティ層**: IAMロールとネットワーク設定

#### データフロー

1. イベントソースから関数がトリガーされる
2. Lambda実行環境でコードが実行される
3. 結果がCloudWatchログや宛先に送信される

</details>

---

## Why - なぜaws_lambda_functionが必要なのか

### 解決する課題

<details>
<summary>課題の詳細</summary>

#### 従来の問題点

1. **サーバー管理の負担**
   - 影響: インフラ管理に時間とリソースが必要
   - 例: EC2インスタンスのパッチ適用やスケーリング

2. **リソースの無駄**
   - 影響: アイドル時もコストが発生
   - 例: 夜間や週末の低トラフィック時

#### aws_lambda_functionによる解決策

- サーバーレスによるインフラ管理の削減
- 使用時のみの課金による効率的なコスト管理
- 自動スケーリングによる柔軟な対応

</details>

### メリット

<details>
<summary>メリットの詳細</summary>

#### ビジネス面のメリット

1. **コスト削減**
   - アイドル時のコストがゼロ
   - 100万リクエストまで無料枠

2. **生産性向上**
   - インフラ管理が不要
   - 開発に集中可能

3. **市場投入時間の短縮**
   - 迅速なデプロイメント
   - プロトタイピングの高速化

#### 技術面のメリット

1. **高可用性**
   - マルチAZ自動レプリケーション
   - 自動フェイルオーバー

2. **柔軟な統合**
   - 多数のAWSサービスとの連携
   - イベントドリブンアーキテクチャ

</details>

### デメリット

<details>
<summary>デメリットと対策</summary>

| デメリット | 影響 | 対策 |
|-----------|------|------|
| コールドスタート | 初回実行の遅延 | プロビジョンドコンカレンシー使用 |
| 実行時間制限 | 15分まで | 長時間処理はStep Functionsで分割 |
| ベンダーロックイン | AWS依存 | 標準的なランタイムとコード設計 |

</details>

### 他の選択肢との比較

<details>
<summary>比較表</summary>

| 項目 | Lambda | EC2 | Fargate |
|------|--------|-----|---------|
| 管理負荷 | なし | 高い | 中程度 |
| 起動時間 | ミリ秒 | 分単位 | 秒〜分 |
| 課金 | 使用時のみ | 常時 | タスク実行時 |
| 実行時間 | 最大15分 | 無制限 | 無制限 |

</details>

---

## How - aws_lambda_functionの実装方法

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
# Lambda関数のコードをZIP化
data "archive_file" "lambda_zip" {
  type        = "zip"
  source_dir  = "${path.module}/lambda_function"
  output_path = "${path.module}/lambda_function.zip"
}

# IAM実行ロール
resource "aws_iam_role" "lambda_role" {
  name = "${var.project_name}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

# 基本的な実行ポリシーをアタッチ
resource "aws_iam_role_policy_attachment" "lambda_logs" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# Lambda関数
resource "aws_lambda_function" "example" {
  filename         = data.archive_file.lambda_zip.output_path
  function_name    = "${var.project_name}-function"
  role            = aws_iam_role.lambda_role.arn
  handler         = "index.handler"
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256
  runtime         = "python3.9"
  memory_size     = 256
  timeout         = 60

  environment {
    variables = {
      ENVIRONMENT = var.environment
      LOG_LEVEL   = "INFO"
    }
  }

  tags = {
    Name        = "${var.project_name}-lambda"
    Environment = var.environment
  }
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

# 関数のテスト実行
aws lambda invoke --function-name example-function output.json
```

</details>

### ベストプラクティス

<details>
<summary>推奨される実装方法</summary>

#### 1. コールドスタート対策

```hcl
# プロビジョンドコンカレンシーの設定
resource "aws_lambda_provisioned_concurrency_config" "example" {
  function_name                     = aws_lambda_function.example.function_name
  provisioned_concurrent_executions = 5
  qualifier                         = aws_lambda_function.example.version
}

# 環境変数でコネクション再利用
resource "aws_lambda_function" "optimized" {
  function_name = "${var.project_name}-optimized"
  # ... 他の設定 ...

  environment {
    variables = {
      # Node.jsでHTTP接続を再利用
      AWS_NODEJS_CONNECTION_REUSE_ENABLED = "1"
      # Pythonでboto3クライアントを再利用
      PYTHONUNBUFFERED = "1"
    }
  }

  # アーキテクチャの最適化（Graviton2）
  architectures = ["arm64"]
}
```

**理由**: 初回実行の遅延を削減し、パフォーマンスを向上

#### 2. セキュアな環境変数管理

```hcl
# KMSキーで環境変数を暗号化
resource "aws_kms_key" "lambda" {
  description = "Lambda environment variables encryption"
}

resource "aws_lambda_function" "secure" {
  function_name = "${var.project_name}-secure"
  # ... 他の設定 ...

  kms_key_arn = aws_kms_key.lambda.arn

  environment {
    variables = {
      # 非機密情報
      APP_ENV = var.environment
      # 機密情報はSecrets Managerから取得
      SECRET_ARN = aws_secretsmanager_secret.app_secret.arn
    }
  }
}

# Secrets Managerへのアクセス権限
resource "aws_iam_role_policy" "secrets_access" {
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "secretsmanager:GetSecretValue"
      ]
      Resource = aws_secretsmanager_secret.app_secret.arn
    }]
  })
}
```

**理由**: 機密情報の安全な管理とコンプライアンス対応

#### 3. 適切なエラーハンドリング

```hcl
# デッドレターキューの設定
resource "aws_sqs_queue" "dlq" {
  name = "${var.project_name}-lambda-dlq"
}

resource "aws_lambda_function" "with_dlq" {
  function_name = "${var.project_name}-with-dlq"
  # ... 他の設定 ...

  dead_letter_config {
    target_arn = aws_sqs_queue.dlq.arn
  }

  # 非同期実行の設定
  reserved_concurrent_executions = 100
}

# イベント宛先の設定
resource "aws_lambda_function_event_invoke_config" "example" {
  function_name = aws_lambda_function.with_dlq.function_name

  destination_config {
    on_success {
      destination = aws_sns_topic.success.arn
    }

    on_failure {
      destination = aws_sqs_queue.dlq.arn
    }
  }

  maximum_retry_attempts = 2
  maximum_event_age_in_seconds = 3600
}
```

**理由**: エラー時の適切な処理とデバッグの容易性

</details>

### よくある実装パターン

<details>
<summary>実装パターン集</summary>

#### パターン1: API バックエンド

**用途**: REST APIのバックエンド処理

```hcl
# API Gateway統合
resource "aws_lambda_function" "api_backend" {
  function_name = "${var.project_name}-api"
  runtime       = "nodejs18.x"
  handler       = "index.handler"
  # ... 他の設定 ...

  environment {
    variables = {
      TABLE_NAME = aws_dynamodb_table.api_data.name
      CORS_ORIGIN = var.allowed_origin
    }
  }
}

# API Gatewayの権限
resource "aws_lambda_permission" "api_gateway" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.api_backend.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_api_gateway_rest_api.example.execution_arn}/*/*"
}
```

#### パターン2: S3イベント処理

**用途**: アップロードされたファイルの処理

```hcl
resource "aws_lambda_function" "s3_processor" {
  function_name = "${var.project_name}-s3-processor"
  runtime       = "python3.9"
  handler       = "handler.process_file"
  memory_size   = 3008  # 画像処理用に増加
  timeout       = 300   # 5分
  # ... 他の設定 ...

  environment {
    variables = {
      DESTINATION_BUCKET = aws_s3_bucket.processed.id
      THUMBNAIL_SIZE = "300x300"
    }
  }
}

# S3バケットからの実行権限
resource "aws_lambda_permission" "allow_s3" {
  statement_id  = "AllowExecutionFromS3"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.s3_processor.function_name
  principal     = "s3.amazonaws.com"
  source_arn    = aws_s3_bucket.uploads.arn
}

# S3イベント通知
resource "aws_s3_bucket_notification" "file_upload" {
  bucket = aws_s3_bucket.uploads.id

  lambda_function {
    lambda_function_arn = aws_lambda_function.s3_processor.arn
    events              = ["s3:ObjectCreated:*"]
    filter_prefix       = "uploads/"
    filter_suffix       = ".jpg"
  }
}
```

#### パターン3: 定期実行バッチ

**用途**: スケジュールされたバッチ処理

```hcl
resource "aws_lambda_function" "scheduled_batch" {
  function_name = "${var.project_name}-batch"
  runtime       = "python3.9"
  handler       = "batch.handler"
  memory_size   = 1024
  timeout       = 900  # 15分（最大）
  # ... 他の設定 ...

  environment {
    variables = {
      DATABASE_URL = var.database_url
      BATCH_SIZE   = "100"
    }
  }
}

# EventBridgeルール
resource "aws_cloudwatch_event_rule" "schedule" {
  name                = "${var.project_name}-batch-schedule"
  description         = "Trigger batch processing"
  schedule_expression = "rate(1 hour)"
}

resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.schedule.name
  target_id = "LambdaTarget"
  arn       = aws_lambda_function.scheduled_batch.arn
}

resource "aws_lambda_permission" "allow_eventbridge" {
  statement_id  = "AllowExecutionFromEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.scheduled_batch.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.schedule.arn
}
```

</details>

### トラブルシューティング

<details>
<summary>よくある問題と解決方法</summary>

#### エラー1: InvalidParameterValueException

**原因**: 無効なランタイムやハンドラー指定
**解決方法**:

```hcl
# 正しいランタイムとハンドラー形式
resource "aws_lambda_function" "valid" {
  # Pythonの場合
  runtime = "python3.9"  # サポートされているバージョン
  handler = "main.lambda_handler"  # ファイル名.関数名

  # Node.jsの場合
  # runtime = "nodejs18.x"
  # handler = "index.handler"  # ファイル名.エクスポート名
}
```

#### エラー2: ResourceConflictException

**原因**: 同じ名前の関数が既に存在
**解決方法**:

```hcl
# 一意な関数名を生成
resource "aws_lambda_function" "unique" {
  function_name = "${var.project_name}-${var.environment}-${data.aws_caller_identity.current.account_id}"
  # または
  function_name = "${var.project_name}-${random_id.function.hex}"
  # ... 他の設定 ...
}

resource "random_id" "function" {
  byte_length = 4
}
```

#### エラー3: CodeStorageExceededException

**原因**: アカウントのコードストレージ制限超過（75GB）
**解決方法**:

```hcl
# 古いバージョンの自動削除
resource "aws_lambda_function" "with_cleanup" {
  function_name = var.function_name
  # ... 他の設定 ...

  # 最新バージョンのみ保持
  publish = false
}

# またはS3からコードをデプロイ
resource "aws_lambda_function" "from_s3" {
  function_name = var.function_name
  s3_bucket     = aws_s3_bucket.lambda_code.id
  s3_key        = "lambda/${var.function_name}.zip"
  # ... 他の設定 ...
}
```

</details>

---

## 参照：daily-TIL

このドキュメントは以下のdaily-TILファイルから情報を集約・整理しています：

### What関連

- [2025.07.28.17.36 - what_is-aws-lambda.md](daily/2025.07.28.17.36_what_is-aws-lambda.md)
  - AWS Lambdaの包括的な概要、特徴、ユースケース、制限事項について

- [2025.07.28.16.45 - what_aws_lambda_security_restrictions.md](daily/2025.07.28.16.45_what_aws_lambda_security_restrictions.md)
  - Lambdaのセキュリティ制限について

- [2025.07.28.16.47 - what_aws_lambda_eventsource_distinction.md](daily/2025.07.28.16.47_what_aws_lambda_eventsource_distinction.md)
  - イベントソースの種類と区別について

- [2025.07.28.16.50 - what_aws_lambda_calltype_distinction.md](daily/2025.07.28.16.50_what_aws_lambda_calltype_distinction.md)
  - 呼び出しタイプの区別について

- [2025.07.28.16.52 - what_aws_lambda_retry_specifications.md](daily/2025.07.28.16.52_what_aws_lambda_retry_specifications.md)
  - リトライ仕様について

- [2025.07.28.16.54 - what_aws_lambda_vpc_access.md](daily/2025.07.28.16.54_what_aws_lambda_vpc_access.md)
  - VPCアクセスについて

### Why関連

- [2025.07.28.16.40 - why_server_complexity_vs_aws_lambda.md](daily/2025.07.28.16.40_why_server_complexity_vs_aws_lambda.md)
  - サーバー管理の複雑性とLambdaの利点について

- [2025.07.28.16.42 - why_serverless_concept_aws_lambda.md](daily/2025.07.28.16.42_why_serverless_concept_aws_lambda.md)
  - サーバーレスコンセプトとLambdaの必要性について

### How関連

- 現在のところ、aws_lambda_functionの実装方法に関するdaily-TILファイルはありません

---

## バージョン履歴

| バージョン | 更新日 | 主な変更内容 |
|-----------|---------|-------------|
| 1.0.0 | 2025-08-11 | 初版作成 |

---

> [!TIP]
> より詳細な情報や具体的な実装例については、上記のdaily-TILリンクを参照してください。
> このドキュメントは定期的に更新され、新しい学習内容が追加されます。