# AWSコンソールでECRリポジトリを作成する
ここからはECRリポジトリを作成する作業を通じて、CFnを説明します。  
ECRリポジトリは設定項目が少なく比較的簡単なので、最初にこれを選びました。

ここではCFnでリソースを作成する前に、まずAWSリソースコンソールで作成してみます。

## ECRリポジトリとは
Dockerイメージの置き場所です。Docker Hubみたいなものです。

## AWSコンソールで作成する
1. AWSコンソールにログイン
1. [ECR Repositories](https://ap-northeast-1.console.aws.amazon.com/ecr/repositories?region=ap-northeast-1)のページに行く  
   サービス名の検索窓に "ECR" か入れれば行けます。
1. "Create Repository" を押す
1. 必要な項目に入力  
   "Repository name" だけ書けば、他はデフォルトでOKです。アルファベットは小文字しか使えないので注意してください。  
   他には以下のような項目があるはずですが、そのままでいいです。
   - Visibility settings
   - Tag immutability
   - Scan on push
   - KMS encryption
1. "Create Repository" を押し、作成
1. リポジトリ一覧に、入力した名前のリポジトリがあることを確認

## AWSコンソールで削除する
ではいま作成したECRリポジトリを削除してみましょう。

1. いま作成したリポジトリを選択
1. "Delete" を押す
1. 確認画面が出てくるので、表示に従って実行
1. リポジトリ一覧に、先ほどのリポジトリがないことを確認