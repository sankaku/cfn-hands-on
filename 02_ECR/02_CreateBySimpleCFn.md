# CloudFormationでECRリポジトリを作成する
ではCFnではどうなるか見てみましょう。  
できる限り単純なテンプレートから始めます。このテンプレートを書き、デプロイ・削除をする中で、CFnに慣れていきましょう。

ECRリポジトリのリソースに対応するCFnのドキュメントはこれです。これを参照しながら書いていきます。
[AWS::ECR::Repository - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecr-repository.html)

## 単純なCFnテンプレートを作成
以下のようなテンプレートのファイルを作成します。  
必須のセクションはResourcesだけですが、わかりやすいようにAWSTemplateFormatVersionとDescriptionも書いておきます。(なおDescriptionには日本語も書けますが、AWSコンソールで見た場合に文字化けするようです。)

名前は例として ecr.yaml` とします。

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Very Simple CFn Template for ECR Repository"

Resources:
  Type: AWS::ECR::Repository
  Properties:
    RepositoryName: very_simple_repository
```

リポジトリ名だけ指定して、ECRリポジトリを作るテンプレート…のはずです。

ではこのテンプレートをチェックしてみましょう。

```sh
$ cfn-lint ecr.yaml
E3001 Resource not properly configured at Type
ecr.yaml:6:3

E3001 Type not defined for resource Properties
ecr.yaml:7:3

E3001 Invalid resource attribute RepositoryName for resource Properties
ecr.yaml:8:5
```

エラーが出ましたね。Resourcesセクションのフォーマットを思い出すと、各リソースにLogical IDを設定しないといけないのでした。Logical IDは自分で好きに決められるので、適当に決めて `VerySimpleRepository` とします。

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Very Simple CFn Template for ECR Repository"

Resources:
  VerySimpleRepository:
    Type: AWS::ECR::Repository
    Properties:
      RepositoryName: very_simple_repository
```

再びcfn-lintにかけてみます。

```sh
$ cfn-lint ecr.yaml
```

何も出力されません。これは(少なくとも構文的には)OKだということです。  
これでCFnテンプレートができました。

## AWSコンソールでデプロイ・削除
ではこのCFnテンプレートで、スタック(この場合はECRリポジトリ1つのみ)をデプロイしましょう。

まずAWSコンソールでデプロイする方法です。  
[CloudFormationのページ](https://ap-northeast-1.console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks)へ行き、 "Create Stack" から、テンプレートをアップロードしてスタックを作成します。途中でスタック名を入れる箇所がありますが、適当に入力すればいいです。他はデフォルトのままでいいです。

デプロイが始まると、CFnの画面で進行状況が見れます。うまくリソースが作成できると、ECRの画面で、上記のリソースができているのが確認できます。

ではいま作成したスタックを削除しましょう。  
**CFnの画面で、スタックを削除します** 。ECRの画面で削除するのではないので、注意してください。CFnで作成したリソースを変更するときは、基本的にCFnで変更するようにしてください。  
先ほどのスタックを選択し、削除すればいいです。スタックが消えているのを確認したあと、ECRの画面を見て、リポジトリが消去されているのを確認します。

## AWS CLIでデプロイ
これでCFnのデプロイまで行いましたが、どうせCFnでIaCをやっているので、デプロイもAWSコンソールなしで行いましょう。

```sh
aws cloudformation deploy \
  --template-file ./ecr.yaml \
  --stack-name very-simple-ecr-stack
```

これを実行したあと、AWSコンソールを見ると、CFnの画面でスタックが作成されています。そしてECRの画面ではリポジトリが作成されています。

## テンプレートを変更し、AWS CLIで再デプロイ
### テンプレートを変更
リポジトリ名を指定するだけでは面白くないので、もう少し設定をしてみましょう。  
[ドキュメント](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecr-repository.html#cfn-ecr-repository-imagescanningconfiguration)を見ると、ECRリポジトリではDockerイメージの脆弱性チェックをしてくれるようです。デフォルトでは無効になっており、実際ECRリポジトリ一覧を見ると、いま作成したリポジトリは "Scan on push" が "Disabled" です。これを有効にしましょう。

先ほどのテンプレート ecr.yaml を以下のように書き換えます。(`ImageScanningConfiguration` の項目を追加しました。)

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Very Simple CFn Template for ECR Repository"

Resources:
  VerySimpleRepository:
    Type: AWS::ECR::Repository
    Properties:
      RepositoryName: very_simple_repository
      ImageScanningConfiguration:
        ScanOnPush: true
```

これは問題なくチェックを通ります。
```sh
$ cfn-lint ecr.yaml
```

### 再デプロイ
では再デプロイしましょう。  
このとき **スタック名は先ほどと変えないでください** 。

```sh
aws cloudformation deploy \
  --template-file ./ecr.yaml \
  --stack-name very-simple-ecr-stack
```

成功すると、ECRリポジトリの "Scan on push" が "Disabled" だったのが "Enabled" に変わっています。


## AWS CLIで削除
[aws cloudformation delete-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/delete-stack.html)を使います。

```sh
aws cloudformation delete-stack \
  --stack-name very-simple-ecr-stack
```

