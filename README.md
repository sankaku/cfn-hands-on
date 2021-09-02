# cfn-hands-on
[AWS CloudFormation](https://aws.amazon.com/jp/cloudformation/)のハンズオン資料。  
最終的にはECS Fargateによってバッチを実行できるようになる。

- この資料内では **CFn** や **cfn** と書いた場合、AWS CloudFormationを指す
- 「インフラ」と「リソース」の区別がちゃんとできていないかもしれない…インフラはリソースより広い意味で使っているつもり

## TODO
- cfn-lint以外のチェック方法
- デプロイ方法
- クロススタック参照
- ECR以外
- ECRの02でもっといろいろな場合を書く(スタック名を変えてしまった場合、別のスタックで同じリポジトリ名をデプロイしようとして失敗した場合)
- デプロイ失敗した場合の処理
- 組み込み関数、擬似パラメータの詳細
- 必要な権限の表記
- AWS CLIを使う方法

## このハンズオンの目標
- CloudFormationテンプレートの基本的な構成を把握する
- CloudFormationによるデプロイができるようになる(AWSコンソールから)
- CloudFormationによるデプロイができるようになる(AWS CLIから)
- CloudFormationのlinterの使い方を知る
- Parameterの使い方を知り、同じテンプレートから複数環境のデプロイができるようになる
- クロススタック参照を使えるようになる
- CloudFormationの主要な組み込み関数(Sub, Refなど)を把握する

### 扱うリソース
- ネットワーク関連
  - VPC
  - サブネット
  - セキュリティグループ
  - ゲートウェイ
  - ルート
  - ルートテーブル
  - VPCGatewayAttachment
  - SubnetRouteTableAssociation
- ECRリポジトリ
- ECS
  - ECSクラスタ
  - ECSタスク
