README
# CloudFormationを使った運用監視の環境構築

# CloudFormationを使った運用監視の環境構築

このプロジェクトは、**AWS CloudFormation** を使用して運用監視を目的としたインフラ環境を自動構築した学習課題です。  
RaiseTech の学習カリキュラムに沿って、監視・通知・セキュリティの要素を含むインフラをテンプレート化し、再現性と自動化を意識して構築しました。

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

## 構成図（Architecture）

![Architecture Diagram](./images/cloudformation-monitoring.png)

※ 構成図例：VPC内に2つのAZを配置し、PublicサブネットにALB・EC2、PrivateサブネットにRDSを配置。  
ALB → EC2 → RDS の通信をセキュリティグループで制御。WAFでALBを保護し、WAFログをCloudWatch Logsへ送信。CloudWatchアラームはSNS経由でメール通知。

## デプロイ手順（Setup）

1. CloudFormationテンプレート（YAML）を AWS コンソールまたは AWS CLI からアップロードする。  
2. 以下のパラメータを入力してスタックを作成する。  
   - `DBMasterUsername`  
   - `DBMasterUserPassword`（NoEcho により秘匿）  
   - `MyKeyName`（SSH接続用キーペア名）  
3. スタック作成の進捗を CloudFormation コンソールで確認する。  
4. SNS の購読メールが届くため、購読承認（Confirm）を行う。  
5. 必要に応じて EC2 上で負荷（例：CPU ストレス）を発生させ、CloudWatch アラームの発火と SNS 通知を検証する。  
6. WAF の WebACL と ALB の紐付け、WAF ログが CloudWatch Logs に出力されることを確認する。

> 注意：テンプレート内の Log Group 名や IAM ロール名には命名制約がある場合があります。デプロイ前にテンプレート定義を確認してください。AWS リソースの作成は課金対象になる可能性があるため、不要になったらスタックを削除してください。

## 動作確認

| 確認項目 | 内容 |
|---|---|
| スタック作成確認 | CloudFormation により各リソースが正常に作成されることを確認 |
| CloudWatchアラーム | CPU 使用率 0.1% 超過でアラームが発火することを確認（学習目的の閾値設定） |
| SNS通知 | 設定したメールアドレスに警告メールが届くことを確認 |
| WAF WebACL | 作成した WebACL が ALB に紐づいていることを確認 |
| WAFログ出力 | WAF のログが CloudWatch Logs に出力されていることを確認 |
| IAMロール | WAF Logging 用 IAM ロール等が正しく作成され、必要な権限が付与されていることを確認 |

## 学んだこと・工夫した点

- 命名規則の制約を理解  
  - MyWAFLogGroup のように、リソース名の形式や命名制約に従わないと CloudFormation でエラーになる点を学んだ。
- IAM ロールを CloudFormation で定義して使用  
  - WAF ログを CloudWatch Logs に送るための IAM ロールをテンプレート上で実装し、権限委譲の流れを理解した。
- YAML 構文の厳密さとテンプレート整合性の重要性  
  - インデントや配列のミスでエラーが発生したため、テンプレート作成時は細心の注意が必要であると実感した。
- 依存関係と参照関係の整理  
  - `!Ref` / `!GetAtt` の参照が増えると可読性が下がるため、命名ルールやコメントで整理する工夫を行った。
- 監視・通知の実践理解  
  - CloudWatch アラーム、SNS 通知、WAF ログ連携を実際に動かし、運用監視の基礎を体得した。

## 今後の改善点

- テンプレートのパラメータ化を進め、再利用性を高める（例：VPC CIDR、インスタンスタイプ、AZ 数のパラメータ化）。  
- CloudFormation の Outputs を追加して、構築後のリソース情報を自動出力する。  
- CloudWatch Logs Insights を活用したログ分析クエリを追加する。  
- SNS トピックを用途別に分割し、通知先をより細かく管理する（運用アラート／開発通知等）。  
- ALB の HTTPS 化（ACM による証明書管理）およびアクセスログの保管ポリシーを実装する。  

## 備考

- 本プロジェクトは学習目的で作成していますが、実運用を意識した監視・通知・セキュリティ設計を取り入れています。  
- テンプレート適用時は AWS アカウントのポリシーやリージョン制約に注意してください。












