# パラメータを使う
前のページではテンプレートに値をベタ書きしました。今回はパラメータを使って、デプロイ時に値を渡してみましょう。

## テンプレートを書く
`ecr2.yaml` を以下のように書き変えます。

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Template With Parametes for ECR Repository"

Parameters:
  MyRepositoryName:
    Type: String
    Default: default_repository_name
    Description: "This is repository name parameter."

Resources:
  RepositoryWithParameters:
    Type: AWS::ECR::Repository
    Properties:
      RepositoryName: !Ref MyRepositoryName
      ImageScanningConfiguration:
        ScanOnPush: true
```

Parametersセクションを追加し、さらに `RepositoryName` で `MyRepositoryName` を参照させています。

チェックも通ります。
```sh
$ cfn-lint ecr2.yaml
```

## デプロイ
パラメータがありますが、まずはそのままデプロイしましょう。

```sh
aws cloudformation deploy \
  --template-file ./ecr2.yaml \
  --stack-name with-parameters-ecr-stack
```

成功すると `default_repository_name` という名前でECRリポジトリが作成されています。

## パラメータを指定してリソースを変更

### 間違った方法: Defaultを変える
ではこの値を変えてみます。さっきのテンプレートでParametesセクションにある `MyRepositoryName` の値を変えます。

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Template With Parametes for ECR Repository"

Parameters:
  MyRepositoryName:
    Type: String
    Default: new_repository_name
    Description: "This is repository name parameter."

Resources:
  RepositoryWithParameters:
    Type: AWS::ECR::Repository
    Properties:
      RepositoryName: !Ref MyRepositoryName
      ImageScanningConfiguration:
        ScanOnPush: true
```

もちろんチェックも通ります。
```sh
$ cfn-lint ecr2.yaml
```

デプロイします。

```sh
aws cloudformation deploy \
  --template-file ./ecr2.yaml \
  --stack-name with-parameters-ecr-stack
```

が、失敗します。
```sh
$ aws cloudformation deploy \
  --template-file ./ecr2.yaml \
  --stack-name with-parameters-ecr-stack

Waiting for changeset to be created..

No changes to deploy. Stack with-parameters-ecr-stack is up to date
```

変更が何もないそうです。ちゃんとパラメータの値を変えたのに…？

これはどういうことかというと、[Parametersのドキュメント](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)の中の `Default` の説明に書いてあります。

> スタックの作成時に値を指定しなかった場合に、テンプレートで使用される適切な型の値。パラメーターの制約を定義する場合は、これらの制約に従う値を指定する必要があります。

つまり **Defaultの値はスタックの作成時にしか参照されません** 。  
現在は既に一度デプロイしているため、スタックは作成済みです。再びデプロイをすると更新になるので、Defaultの値は使われなくなり、デプロイすべき変更もなくなってしまうというわけです。

### 正しい方法: デプロイ時にパラメータを指定する
実は、テンプレートは変える必要はありません。  
以下のようにデプロイ時に `parameter-overrides` でパラメータの値を指定すればいいです。

(ecr2.yamlは、このページの最初のバージョンでも、2番目のバージョンでもいいです。どちらにせよ、Defaultはスタック更新時は参照されないので。)

```sh
aws cloudformation deploy \
  --template-file ./ecr2.yaml \
  --stack-name with-parameters-ecr-stack \
  --parameter-overrides \
    MyRepositoryName=new_repository_name
```

成功すると `new_repository_name` という名前でECRリポジトリが作成されます。  
(なおこのとき、最初に作った `default_repository_name` という名前のECRリポジトリは削除され、新しく `new_repository_name` が作成されています。ですがスタック自体は「更新」という扱いです。)
