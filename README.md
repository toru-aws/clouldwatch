README
# CloudFormationを使った運用監視の環境構築

このプロジェクトは、**AWS CloudFormation** ＋運用監視を目的としたインフラ環境を自動構築した学習です。  
監視・通知・セキュリティの要素を含むインフラをテンプレート化し、再現性と自動化を意識して構築しました。

## 概要

CloudFormationテンプレートを使用して、以下のAWSリソースを自動構築しました。

- VPC（CIDR: 10.0.0.0/16）
- Public / Private サブネット（2つのAZに配置）
- Internet Gateway / Route Table
- EC2（Amazon Linux 2, t2.micro）
- RDS（MySQL 8.0.39）
- ALB（Application Load Balancer）およびターゲットグループ
- セキュリティグループ設定（SSH, HTTP, 8080, RDSアクセス等）
- CloudWatch（メトリクス、アラーム）
- SNS（メール通知設定）
- WAF（WebACL）と WAF ログの CloudWatch Logs 連携
- IAM ロール（WAF ログ送信用等）

### リポジトリ構成
```bash 
├── clouldwatch
│ └── clouldwatch.yaml
│ └── README.md
```

## 使用技術

| カテゴリ | 使用技術 |
|---|---|
| インフラ構築 | AWS CloudFormation |
| サービス | VPC, EC2, RDS, ALB, WAF, SNS, CloudWatch, IAM, CloudWatch Logs |
| 開発環境 | VSCode / GitHub / PowerShell / AWS CLI |
| OS・AMI | Amazon Linux 2 |
| DB | MySQL 8.0.39 |
| 監視・通知 | CloudWatch, CloudWatch Alarm, SNS（メール通知） |

## デプロイ手順

1. CloudFormationテンプレート（YAML）をAWSコンソールまたは、AWS CLIからアップロードする。  
2. 以下のパラメータを入力してスタックを作成する。  
   - `DBMasterUsername`  
   - `DBMasterUserPassword`（NoEcho により秘匿）  
   - `MyKeyName`（SSH接続用キーペア名）  
3. スタック作成の進捗を CloudFormation コンソールで確認する。（推定作成時間５分前後）
4. SNSで設定した特定のメールアドレス宛に購読メールが届くため、購読承認（Confirm）を行う。
5. 以下の動作確認項目をチェックする
6. CloudFormationを使った運用監視の環境構築の完成

> 注意：テンプレート内のLog Group 名やIAMロール名には命名制約がある場合があります。デプロイ前にテンプレート定義を確認してください。
> ほとんど無料範囲でAWSリソースの作成をしているが、仕様上課金対象になる可能性があるため、不要になったら必ずスタックを削除してください。

## 動作確認

| 確認項目 | 内容 |
|---|---|
| スタック作成確認 | CloudFormationにより実環境の各AWSリソースが正常に作成されることを確認 |
| CloudWatchアラーム | CPU 使用率 0.1% 超過でアラームが発火することを確認（学習目的の閾値設定なので、本番運用では閾値変更すること） |
| SNS通知 | 設定したメールアドレスに警告メールが届くことを確認 |
| WAF WebACL | 作成したWebACLがALBに紐づいていることを確認 |
| WAFログ出力 | WAFのログがCloudWatch Logsに出力されていることを確認 |
| IAMロール |WAF Logging 用IAMロール等が正しく作成され、必要な権限が付与されていることを確認 |

## 学んだこと・工夫した点

- 命名規則の制約を理解  
  - MyWAFLogGroupのように、リソース名の形式や命名制約に従わないと CloudFormation でエラーになる点を学習。
- IAM ロールを CloudFormationで定義して使用  
  - WAFログをCloudWatch Logsに送るためのIAMロールをテンプレート上で実装し、権限委譲の流れを理解した。
- YAML 構文の厳密さとテンプレート整合性の重要性  
  - インデントや配列のミスでエラーが発生したため、テンプレート作成時は細心の注意が必要であると実感した。
- 依存関係と参照関係の整理と把握の難しさ
  - `!Ref` / `!GetAtt` の参照が増えると可読性が下がるため、命名ルールやコメントで少し整理する工夫を行った。
  - また、依存関係と参照関係の把握に大変苦労した。再度勉強してしっかりと覚えたい。
- 監視・通知の実践理解  
  - CloudWatchアラーム、SNS 通知、WAF ログ連携を実際に動かし、運用監視の基礎を体得した。
- 関連する講座動画で実際のサービス運用から客先との仕事が本番という意識を強く持つようになった。
  転職時に保守・運用・監視からインフラキャリアのスタートになったら、この意識を持って仕事に励みたい。

## 今後の改善点

- テンプレートのパラメータ化を進め、再利用性を高める（例：VPC CIDR、インスタンスタイプ、AZ 数のパラメータ化）
- CloudWatch Logs Insights を活用したログ分析クエリを追加する。
- SNS トピックを用途別に分割し、通知先をより細かく管理する（運用アラート／開発通知等）。  
- ALBのHTTPS 化（ACM による証明書管理）
