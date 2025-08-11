# TIL (Today I Learned) - ナレッジベース インデックス

> [!NOTE]
> TIL（Today I Learned）関連の知識のメインエントリーファイルです。
> TIL（Today I Learned）ファイルを目的別（Why/What/How）に整理・集約しています。

## 目次

<details>
<summary>目次を開く</summary>

- [AWS Services](#aws-services)
  - [ACM (AWS Certificate Manager)](#acm-aws-certificate-manager)
  - [ALB (Application Load Balancer)](#alb-application-load-balancer)
  - [CloudFront](#cloudfront)
  - [DynamoDB](#dynamodb)
  - [ECS](#ecs)
  - [EFS](#efs)
  - [Fargate](#fargate)
  - [Lambda](#lambda)
  - [RDS (Relational Database Service)](#rds-relational-database-service)
  - [Route53](#route53)
  - [S3 (Simple Storage Service)](#s3-simple-storage-service)
  - [VPC & Networking](#vpc--networking)
  - [その他のAWSサービス](#その他のawsサービス)
- [Terraform](#terraform)
  - [General Terraform](#general-terraform)
  - [Terraform AWS Resources](#terraform-aws-resources)
- [Security & Certificates](#security--certificates)
  - [SSL/TLS & Certificates](#ssltls--certificates)
- [Architecture & Design](#architecture--design)
  - [Software Architecture](#software-architecture)
- [Development Tools & Libraries](#development-tools--libraries)
  - [Rails Gems](#rails-gems)

</details>

---

## TILs

### AWS Services

#### ACM (AWS Certificate Manager)
<details>
<summary>AWS Services : ACM (AWS Certificate Manager)</summary>

- AWS Services
  - ACM (AWS Certificate Manager)
    - summarized-TIL
      - [about_aws_acm_certificates.md](summarized/about_aws_acm_certificates.md)
        - ACM証明書管理の包括的ガイド
    - daily-TIL
      - what
        - [2025.07.24.08.47_what_aws_acm_certificate.md](daily/2025.07.24.08.47_what_aws_acm_certificate.md)
          - AWS ACMの基本概念。SSL/TLS証明書の無料発行・管理サービス
        - [2025.08.02.14.35_what_aws_acm_certificate_configuration_items_detailed_reference.md](daily/2025.08.02.14.35_what_aws_acm_certificate_configuration_items_detailed_reference.md)
          - ACM証明書の詳細な設定項目リファレンス
      - why
        - [2025.08.01.13.47_why_acm_certificate_management_patterns_comparison.md](daily/2025.08.01.13.47_why_acm_certificate_management_patterns_comparison.md)
          - ACM証明書管理パターンの比較。なぜ特定のパターンを選択するのか
      - how
        - [2025.07.24.08.35_how_aws_acm_certificate_tutorial.md](daily/2025.07.24.08.35_how_aws_acm_certificate_tutorial.md)
          - AWS ACMを使用したSSL/TLS証明書の設定手順。ドメイン検証から適用まで
        - [2025.08.01.14.31_how_aws_acm_certificate_terraform_configuration_guide.md](daily/2025.08.01.14.31_how_aws_acm_certificate_terraform_configuration_guide.md)
          - TerraformでのACM証明書設定ガイド
        - [2025.08.02.13.21_how_terraform_acm_certificate_type_determination_method.md](daily/2025.08.02.13.21_how_terraform_acm_certificate_type_determination_method.md)
          - TerraformでのACM証明書タイプ決定方法

</details>

#### ALB (Application Load Balancer)
<details>
<summary>AWS Services : ALB (Application Load Balancer)</summary>

- AWS Services
  - ALB (Application Load Balancer)
    - summarized-TIL
      - [about_aws_alb_load_balancer.md](summarized/about_aws_alb_load_balancer.md)
        - ALBロードバランサーの包括的ガイド
    - daily-TIL
      - what
        - [2025.08.07.10.15_what_internal_setting_in_aws_lb.md](daily/2025.08.07.10.15_what_internal_setting_in_aws_lb.md)
          - ALBの内部設定の詳細
        - [2025.08.07.11.00_what_enable_deletion_protection_for_aws_alb.md](daily/2025.08.07.11.00_what_enable_deletion_protection_for_aws_alb.md)
          - ALBの削除保護機能
        - [2025.08.07.11.05_what_enable_http2_for_aws_alb.md](daily/2025.08.07.11.05_what_enable_http2_for_aws_alb.md)
          - ALBでのHTTP/2有効化設定
        - [2025.08.07.11.42_what_aws_lb_listener.md](daily/2025.08.07.11.42_what_aws_lb_listener.md)
          - ALBリスナーの設定と動作
      - why
        - [2025.08.04.15.34_why_alb_must_assign_in_public_subnet.md](daily/2025.08.04.15.34_why_alb_must_assign_in_public_subnet.md)
          - ALBがパブリックサブネットに配置される必要がある理由

</details>

#### CloudFront
<details>
<summary>AWS Services : CloudFront</summary>

- AWS Services
  - CloudFront
    - summarized-TIL
      - [about_aws_cloudfront_cdn.md](summarized/about_aws_cloudfront_cdn.md)
        - CloudFront CDNの包括的ガイド
    - daily-TIL
      - what
        - [2025.08.02.09.30_what_aws_cloudfront_static_asset_delivery.md](daily/2025.08.02.09.30_what_aws_cloudfront_static_asset_delivery.md)
          - CloudFrontによる静的アセット配信の基本概念
        - [2025.08.07.13.50_what_geo_restriction_cloudfront_terraform.md](daily/2025.08.07.13.50_what_geo_restriction_cloudfront_terraform.md)
          - CloudFrontの地理的制限設定
        - [2025.08.07.13.57_what_default_root_object_cloudfront_terraform.md](daily/2025.08.07.13.57_what_default_root_object_cloudfront_terraform.md)
          - デフォルトルートオブジェクトの設定
        - [2025.08.07.14.05_what_ssl_support_method_cloudfront_terraform.md](daily/2025.08.07.14.05_what_ssl_support_method_cloudfront_terraform.md)
          - SSL/TLSサポート方法の設定

</details>

#### DynamoDB
<details>
<summary>AWS Services : DynamoDB</summary>

- AWS Services
  - DynamoDB
    - daily-TIL
      - what
        - [2025.08.07.06.18_what_dynamodb.md](daily/2025.08.07.06.18_what_dynamodb.md)
          - DynamoDBの基本概念。フルマネージドNoSQLデータベース
      - why
        - [2025.08.07.06.22_why_dynamodb_not_mandatory_for_backend_tf.md](daily/2025.08.07.06.22_why_dynamodb_not_mandatory_for_backend_tf.md)
          - Terraformバックエンドで必須でない理由
        - [2025.08.07.06.25_why_dynamodb_does_not_interfere_with_rds.md](daily/2025.08.07.06.25_why_dynamodb_does_not_interfere_with_rds.md)
          - DynamoDBとRDSが干渉しない理由
      - how
        - [2025.08.07.06.28_how_to_use_distinction_between_rds_and_dynamodb.md](daily/2025.08.07.06.28_how_to_use_distinction_between_rds_and_dynamodb.md)
          - RDSとDynamoDBの使い分け方法

</details>

#### ECS
<details>
<summary>AWS Services : ECS</summary>

- AWS Services
  - ECS
    - summarized-TIL
      - [about_aws_ecs.md](summarized/about_aws_ecs.md)
        - ECSコンテナオーケストレーションの包括的ガイド
    - daily-TIL
      - what
        - [2025.08.07.11.16_what_aws_ecs_cluster_resource.md](daily/2025.08.07.11.16_what_aws_ecs_cluster_resource.md)
          - ECSクラスターリソースの詳細
        - [2025.08.07.11.33_what_assign_public_ip_in_ecs_network_configuration.md](daily/2025.08.07.11.33_what_assign_public_ip_in_ecs_network_configuration.md)
          - ECSネットワーク設定でのパブリックIP割り当て
        - [2025.08.07.11.37_what_aws_ecs_task_definition.md](daily/2025.08.07.11.37_what_aws_ecs_task_definition.md)
          - ECSタスク定義の詳細。コンテナ実行に必要な設定項目
      - how
        - [2025.08.07.11.22_how_docker_and_ecs_work_together.md](daily/2025.08.07.11.22_how_docker_and_ecs_work_together.md)
          - DockerとECSの連携方法
        - [2025.08.07.11.27_how_terraform_aws_ecs_service_implementation.md](daily/2025.08.07.11.27_how_terraform_aws_ecs_service_implementation.md)
          - ECSサービスのTerraform実装。Fargateタイプでの本番向け設定
        - [2025.08.07.11.11_how_terraform_ecs_autoscaling_implementation.md](daily/2025.08.07.11.11_how_terraform_ecs_autoscaling_implementation.md)
          - ECS Auto Scalingポリシーの実装。CPU/メモリベースの自動スケーリング

</details>

#### EFS
<details>
<summary>AWS Services : EFS</summary>

- AWS Services
  - EFS
    - summarized-TIL
      - [about_aws_efs.md](summarized/about_aws_efs.md)
        - EFS共有ファイルシステムの包括的ガイド

</details>

#### Fargate
<details>
<summary>AWS Services : Fargate</summary>

- AWS Services
  - Fargate
    - summarized-TIL
      - [about_aws_fargate.md](summarized/about_aws_fargate.md)
        - Fargateコンテナ実行環境の包括的ガイド
    - daily-TIL
      - what
        - [2025.07.28.16.50_what_is-aws-fargate.md](daily/2025.07.28.16.50_what_is-aws-fargate.md)
          - AWS Fargateの概要。サーバー管理不要のコンテナ実行プラットフォーム

</details>

#### Lambda
<details>
<summary>AWS Services : Lambda</summary>

- AWS Services
  - Lambda
    - summarized-TIL
      - [about_aws_lambda.md](summarized/about_aws_lambda.md)
        - Lambdaサーバーレスコンピューティングの包括的ガイド
    - daily-TIL
      - what
        - [2025.07.28.17.36_what_is-aws-lambda.md](daily/2025.07.28.17.36_what_is-aws-lambda.md)
          - AWS Lambdaの基本概念。イベント駆動型で従量課金制のコンピューティングサービス
        - [2025.07.28.16.45_what_aws_lambda_security_restrictions.md](daily/2025.07.28.16.45_what_aws_lambda_security_restrictions.md)
          - AWS Lambdaのセキュリティ制限事項
        - [2025.07.28.16.47_what_aws_lambda_eventsource_distinction.md](daily/2025.07.28.16.47_what_aws_lambda_eventsource_distinction.md)
          - AWS Lambdaのイベントソースの種類と特徴
        - [2025.07.28.16.50_what_aws_lambda_calltype_distinction.md](daily/2025.07.28.16.50_what_aws_lambda_calltype_distinction.md)
          - Lambda呼び出しタイプの区別
        - [2025.07.28.16.52_what_aws_lambda_retry_specifications.md](daily/2025.07.28.16.52_what_aws_lambda_retry_specifications.md)
          - Lambdaのリトライ仕様
        - [2025.07.28.16.54_what_aws_lambda_vpc_access.md](daily/2025.07.28.16.54_what_aws_lambda_vpc_access.md)
          - LambdaのVPCアクセス設定
      - why
        - [2025.07.28.16.40_why_server_complexity_vs_aws_lambda.md](daily/2025.07.28.16.40_why_server_complexity_vs_aws_lambda.md)
          - 従来のサーバー管理の複雑さとAWS Lambdaのサーバーレスアプローチの比較
        - [2025.07.28.16.42_why_serverless_concept_aws_lambda.md](daily/2025.07.28.16.42_why_serverless_concept_aws_lambda.md)
          - サーバーレスアーキテクチャのパラダイムシフト。インフラ管理から解放される理由

</details>

#### RDS (Relational Database Service)
<details>
<summary>AWS Services : RDS (Relational Database Service)</summary>

- AWS Services
  - RDS (Relational Database Service)
    - summarized-TIL
      - [about_aws_rds_database.md](summarized/about_aws_rds_database.md)
        - RDSマネージドデータベースの包括的ガイド
    - daily-TIL
      - what
        - [2025.08.04.16.34_what_rds_replication.md](daily/2025.08.04.16.34_what_rds_replication.md)
          - RDSのレプリケーション機能
        - [2025.08.07.11.48_what_aws_rds_cluster_terraform_resource.md](daily/2025.08.07.11.48_what_aws_rds_cluster_terraform_resource.md)
          - RDSクラスターのTerraformリソース定義
      - how
        - [2025.08.07.11.52_how_secure_rds_password_with_secrets_manager.md](daily/2025.08.07.11.52_how_secure_rds_password_with_secrets_manager.md)
          - Secrets Managerを使用したRDSパスワードの安全な管理方法
        - [2025.08.07.12.09_how_use_secrets_manager_for_rds_in_terraform.md](daily/2025.08.07.12.09_how_use_secrets_manager_for_rds_in_terraform.md)
          - TerraformでのSecrets Manager統合

</details>

#### Route53
<details>
<summary>AWS Services : Route53</summary>

- AWS Services
  - Route53
    - summarized-TIL
      - [about_terraform_aws_route53_zone.md](summarized/about_terraform_aws_route53_zone.md)
        - Route53ゾーン管理の包括的ガイド
    - daily-TIL
      - what
        - [2025.07.17.12.21_what_aws_route53.md](daily/2025.07.17.12.21_what_aws_route53.md)
          - AWS Route53の基本概念、主な機能、セキュリティ機能について
      - how
        - [2025.07.24.08.58_how_aws_route53_tutorial.md](daily/2025.07.24.08.58_how_aws_route53_tutorial.md)
          - TerraformでのRoute53実装チュートリアル、各種レコードタイプの設定方法

</details>

#### S3 (Simple Storage Service)
<details>
<summary>AWS Services : S3 (Simple Storage Service)</summary>

- AWS Services
  - S3 (Simple Storage Service)
    - summarized-TIL
      - [about_aws_s3.md](summarized/about_aws_s3.md)
        - S3の包括的ガイド
    - daily-TIL
      - what
        - [2025.08.02.11.50_what_s3_only_static_assets_broadcasting.md](daily/2025.08.02.11.50_what_s3_only_static_assets_broadcasting.md)
          - S3による静的アセット配信の基本概念
        - [2025.08.04.15.13_what_s3_inside_subnet_patterns.md](daily/2025.08.04.15.13_what_s3_inside_subnet_patterns.md)
          - S3とVPCの接続パターン
        - [2025.08.07.07.53_what_aws_s3_bucket_server_side_encryption_configuration.md](daily/2025.08.07.07.53_what_aws_s3_bucket_server_side_encryption_configuration.md)
          - S3バケットのサーバーサイド暗号化設定
        - [2025.08.07.08.00_what_aws_s3_bucket_public_access_block.md](daily/2025.08.07.08.00_what_aws_s3_bucket_public_access_block.md)
          - S3バケットのパブリックアクセスブロック設定
        - [2025.08.07.09.18_what_s3_bucket_versioning.md](daily/2025.08.07.09.18_what_s3_bucket_versioning.md)
          - S3バケットのバージョニング機能
        - [2025.08.07.09.21_what_s3_bucket_policy.md](daily/2025.08.07.09.21_what_s3_bucket_policy.md)
          - S3バケットポリシーの基本。JSON形式でのアクセス制御設定
      - why
        - [2025.08.04.15.08_why_s3_assigned_outside_of_subnet.md](daily/2025.08.04.15.08_why_s3_assigned_outside_of_subnet.md)
          - S3がVPCの外部に配置される理由
        - [2025.08.07.09.07_why_block_public_acls_prevents_new_acl_settings.md](daily/2025.08.07.09.07_why_block_public_acls_prevents_new_acl_settings.md)
          - パブリックACLをブロックする理由と影響

</details>

#### VPC & Networking
<details>
<summary>AWS Services : VPC & Networking</summary>

- AWS Services
  - VPC & Networking
    - summarized-TIL
      - [about_aws_vpc_networking.md](summarized/about_aws_vpc_networking.md)
        - VPCとネットワーキングの包括的ガイド
    - daily-TIL
      - what
        - [2025.08.04.16.52_what_vpc_flow_logs.md](daily/2025.08.04.16.52_what_vpc_flow_logs.md)
          - VPCフローログの基本。ネットワークトラフィックの記録と分析
        - [2025.08.04.17.03_what_vpc_flow_logs_can_manage.md](daily/2025.08.04.17.03_what_vpc_flow_logs_can_manage.md)
          - VPCフローログで管理できる内容
        - [2025.08.04.20.18_what_terraform_aws_vpc_official_complete_reference.md](daily/2025.08.04.20.18_what_terraform_aws_vpc_official_complete_reference.md)
          - TerraformでのAWS VPC設定の完全リファレンス
        - [2025.08.04.21.28_what_enable_dns_hostnames_in_terraform_aws_vpc.md](daily/2025.08.04.21.28_what_enable_dns_hostnames_in_terraform_aws_vpc.md)
          - VPCでのDNSホスト名有効化設定
        - [2025.08.07.07.38_what_instance_tenancy_in_aws_vpc.md](daily/2025.08.07.07.38_what_instance_tenancy_in_aws_vpc.md)
          - VPCのインスタンステナンシー設定の詳細
        - [2025.08.07.07.44_what_enable_dns_support_in_aws_vpc.md](daily/2025.08.07.07.44_what_enable_dns_support_in_aws_vpc.md)
          - VPCでのDNSサポート有効化
        - [2025.08.07.07.47_what_map_public_ip_on_launch_in_aws_subnet.md](daily/2025.08.07.07.47_what_map_public_ip_on_launch_in_aws_subnet.md)
          - サブネットでのパブリックIP自動割り当て設定
        - [2025.08.07.09.30_what_route_table_association.md](daily/2025.08.07.09.30_what_route_table_association.md)
          - ルートテーブルの関連付け設定
      - why
        - [2025.08.04.16.56_why_vpc_flow_logs_is_managed_service.md](daily/2025.08.04.16.56_why_vpc_flow_logs_is_managed_service.md)
          - VPCフローログがマネージドサービスである理由
      - how
        - [2025.08.04.15.27_how_connect_between_private_subnets.md](daily/2025.08.04.15.27_how_connect_between_private_subnets.md)
          - プライベートサブネット間の接続方法
        - [2025.08.07.09.47_how_terraform_aws_route_table_implementation.md](daily/2025.08.07.09.47_how_terraform_aws_route_table_implementation.md)
          - Terraformでのルートテーブル実装方法

</details>

#### その他のAWSサービス
<details>
<summary>AWS Services : その他のAWSサービス</summary>

- AWS Services
  - その他のAWSサービス
    - daily-TIL
      - SQS
        - [2025.07.28.16.37_what_amazon_sqs.md](daily/2025.07.28.16.37_what_amazon_sqs.md)
          - Amazon SQSの基本概念
      - API Gateway
        - [2025.08.04.17.20_what_aws_api_gateway.md](daily/2025.08.04.17.20_what_aws_api_gateway.md)
          - API Gatewayの基本概念
        - [2025.08.04.17.28_how_judge_necessity_of_api_gateway.md](daily/2025.08.04.17.28_how_judge_necessity_of_api_gateway.md)
          - API Gatewayの必要性判断方法
      - WAF
        - [2025.08.04.17.15_what_services_aws_waf_protects.md](daily/2025.08.04.17.15_what_services_aws_waf_protects.md)
          - AWS WAFが保護するサービス
        - [2025.08.04.17.10_why_waf_is_aws_managed_service.md](daily/2025.08.04.17.10_why_waf_is_aws_managed_service.md)
          - WAFがマネージドサービスである理由

</details>

### Terraform

#### General Terraform
<details>
<summary>Terraform : General Terraform</summary>

- Terraform
  - General Terraform
    - summarized-TIL
      - [about_terraform.md](summarized/about_terraform.md)
        - Terraform全般の包括的ガイド
    - daily-TIL
      - what
        - [2025.08.07.05.43_what_terraform_backend_tf.md](daily/2025.08.07.05.43_what_terraform_backend_tf.md)
          - Terraformのbackend設定ファイルの詳細
        - [2025.08.07.06.46_what_tflint_hcl.md](daily/2025.08.07.06.46_what_tflint_hcl.md)
          - TFLintの設定ファイル詳細
        - [2025.08.07.06.51_what_terraform_standard_module_structure_rule.md](daily/2025.08.07.06.51_what_terraform_standard_module_structure_rule.md)
          - Terraformモジュールの標準構造ルール
        - [2025.08.07.07.23_what_desired_count_in_terraform_variables.md](daily/2025.08.07.07.23_what_desired_count_in_terraform_variables.md)
          - Terraform変数でのdesired_count設定
        - [2025.08.07.13.28_what_create_before_destroy_lifecycle_in_terraform.md](daily/2025.08.07.13.28_what_create_before_destroy_lifecycle_in_terraform.md)
          - Terraformのcreate_before_destroyライフサイクル設定
      - why
        - [2025.08.07.05.46_why_backend_tf_sometimes_unnecessary.md](daily/2025.08.07.05.46_why_backend_tf_sometimes_unnecessary.md)
          - backend.tfが不要な場合とその理由
        - [2025.08.07.03.29_why_terraform_aws_components_need_tags.md](daily/2025.08.07.03.29_why_terraform_aws_components_need_tags.md)
          - AWSリソースにタグを付ける重要性
        - [2025.08.04.20.57_why_terraform_resource_naming_by_environment.md](daily/2025.08.04.20.57_why_terraform_resource_naming_by_environment.md)
          - 環境別のリソース命名が必要な理由
        - [2025.08.04.21.19_why_tags_name_should_distinct_by_environment.md](daily/2025.08.04.21.19_why_tags_name_should_distinct_by_environment.md)
          - タグ名を環境別に区別する理由
      - how
        - [2025.07.24.07.14_how_to-create-terraform-module.md](daily/2025.07.24.07.14_how_to-create-terraform-module.md)
          - Terraformモジュールの作成方法
        - [2025.08.04.12.01_how_compose_terraform_directory_architecture_separated_by_env.md](daily/2025.08.04.12.01_how_compose_terraform_directory_architecture_separated_by_env.md)
          - 環境別のTerraformディレクトリ構成方法
        - [2025.08.07.06.13_how_terraform_backend_tf_implementation.md](daily/2025.08.07.06.13_how_terraform_backend_tf_implementation.md)
          - Terraformバックエンド設定の実装方法
        - [2025.08.07.06.48_how_tflint_configuration_best_practices.md](daily/2025.08.07.06.48_how_tflint_configuration_best_practices.md)
          - TFLint設定のベストプラクティス
        - [2025.08.07.10.07_how_set_terraform_variable_values.md](daily/2025.08.07.10.07_how_set_terraform_variable_values.md)
          - Terraform変数値の設定方法
        - [2025.08.07.17.07_how_terraform_github_actions_deployment.md](daily/2025.08.07.17.07_how_terraform_github_actions_deployment.md)
          - GitHub ActionsでのTerraformデプロイメント自動化
        - [2025.08.07.20.11_how_terraform_code_error_checking.md](daily/2025.08.07.20.11_how_terraform_code_error_checking.md)
          - Terraformコードのエラーチェック方法

</details>

#### Terraform AWS Resources
<details>
<summary>Terraform : Terraform AWS Resources</summary>

- Terraform
  - Terraform AWS Resources
    - summarized-TIL
      - VPC関連
        - [about_terraform_aws_vpc.md](summarized/about_terraform_aws_vpc.md)
          - VPCリソース管理の包括的ガイド
        - [about_terraform_aws_subnet.md](summarized/about_terraform_aws_subnet.md)
          - サブネット構成の包括的ガイド
        - [about_terraform_aws_route_table.md](summarized/about_terraform_aws_route_table.md)
          - ルートテーブル設定の包括的ガイド
        - [about_terraform_aws_security_group.md](summarized/about_terraform_aws_security_group.md)
          - セキュリティグループの包括的ガイド
      - コンテナ関連
        - [about_terraform_aws_ecs_cluster.md](summarized/about_terraform_aws_ecs_cluster.md)
          - ECSクラスターの包括的ガイド
        - [about_terraform_aws_ecs_service.md](summarized/about_terraform_aws_ecs_service.md)
          - ECSサービスの包括的ガイド
        - [about_terraform_aws_ecs_task_definition.md](summarized/about_terraform_aws_ecs_task_definition.md)
          - タスク定義の包括的ガイド
      - ロードバランサー関連
        - [about_terraform_aws_lb_listener.md](summarized/about_terraform_aws_lb_listener.md)
          - ALBリスナー設定の包括的ガイド
      - コンピューティング関連
        - [about_terraform_aws_lambda_function.md](summarized/about_terraform_aws_lambda_function.md)
          - Lambda関数の包括的ガイド
      - データベース関連
        - [about_terraform_aws_rds_cluster.md](summarized/about_terraform_aws_rds_cluster.md)
          - RDSクラスターの包括的ガイド
      - ストレージ・ポリシー関連
        - [about_terraform_aws_s3_bucket_policy.md](summarized/about_terraform_aws_s3_bucket_policy.md)
          - S3バケットポリシーの包括的ガイド
      - DNS関連
        - [about_terraform_aws_route53_zone.md](summarized/about_terraform_aws_route53_zone.md)
          - Route53ゾーン管理の包括的ガイド

</details>

### Security & Certificates

#### SSL/TLS & Certificates
<details>
<summary>Security & Certificates : SSL/TLS & Certificates</summary>

- Security & Certificates
  - SSL/TLS & Certificates
    - daily-TIL
      - what
        - [2025.07.17.12.20_what_ssl_tls_certificates.md](daily/2025.07.17.12.20_what_ssl_tls_certificates.md)
          - SSL/TLS証明書の基本概念
        - [2025.08.02.12.57_what_public_certificate_vs_private_certificate_differences_and_usage.md](daily/2025.08.02.12.57_what_public_certificate_vs_private_certificate_differences_and_usage.md)
          - パブリック証明書とプライベート証明書の違いと用途

</details>

### Architecture & Design

#### Software Architecture
<details>
<summary>Architecture & Design : Software Architecture</summary>

- Architecture & Design
  - Software Architecture
    - daily-TIL
      - what
        - [2025.08.04.23.41_what_conway_law.md](daily/2025.08.04.23.41_what_conway_law.md)
          - コンウェイの法則
      - why
        - [2025.07.31.11.10_why_microservices_modular_monolith_monolith_comparison.md](daily/2025.07.31.11.10_why_microservices_modular_monolith_monolith_comparison.md)
          - マイクロサービス、モジュラーモノリス、モノリスの比較
        - [2025.08.04.23.54_why_bastion_ssh_tunnel_not_through_nat_gateway.md](daily/2025.08.04.23.54_why_bastion_ssh_tunnel_not_through_nat_gateway.md)
          - 踏み台サーバーのSSHトンネルがNATゲートウェイを通らない理由

</details>

### Development Tools & Libraries

#### Rails Gems
<details>
<summary>Development Tools & Libraries : Rails Gems</summary>

- Development Tools & Libraries
  - Rails Gems
    - daily-TIL
      - what
        - [2025.07.31.10.55_what_packwerk_rails_gem_documentation.md](daily/2025.07.31.10.55_what_packwerk_rails_gem_documentation.md)
          - Packwerk Railsジェムのドキュメント
        - [2025.07.31.11.15_what_packs_rails_gem_documentation.md](daily/2025.07.31.11.15_what_packs_rails_gem_documentation.md)
          - Packs Railsジェムのドキュメント

</details>