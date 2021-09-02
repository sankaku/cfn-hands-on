# CloudFormationテンプレートファイルの書き方

(CFnのテンプレートの書き方にはYAMLの他にJSONがあります。が、JSONは今回は扱いません。YAMLの方が可読性に優れているのでYAMLだけ覚えればいいと思います。)

## CloudFormationテンプレートとは
CFnでは、どんなAWSリソースを、どんな設定でデプロイするかをコードで記述します。そのコードがCFnテンプレートです。つまりCFnの核心と言っていいものです。

CFnテンプレートの見た目はこんな感じです。([ドキュメント](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html)からのコピペ。)

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template
Resources:
  MyEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties: 
      ImageId: "ami-0ff8a91507f77f867"
      InstanceType: t2.micro
      KeyName: testkey
      BlockDeviceMappings:
        -
          DeviceName: /dev/sdm
          Ebs:
            VolumeType: io1
            Iops: 200
            DeleteOnTermination: false
            VolumeSize: 20
```

## スタックとは
CFnではテンプレート単位でデプロイしたり、削除したりします。このとき、1つのテンプレートから作成される一連のリソースのことをスタックといいます。

## CloudFormationテンプレートの基本
[テンプレートの分析 - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/template-anatomy.html)

CFnテンプレートはいくつかのセクションで構成されます。  
以下では、基本的な使い方をする上で必要になるセクションだけを説明します。

以下やや長いですが、必須のセクションは `Resources` だけなので、とりあえずそこだけ読めばいいです。

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description:
  ...

Parameters:
  ...

Resources:
  ...

Outputs:
  ...
```

### AWSTemplateFormatVersionセクション(任意)
CFnテンプレートでは書き方のフォーマットが決まっており、これのバージョンを指定できます。現在は `2010-09-09` しかないので、これを指定します。

### Descriptionセクション(任意)
そのCFnテンプレートの説明を書きます。

```yaml
Description: This is a sample templete.
```

### Parametersセクション(任意)
テンプレート内の変数のようなものです。  
テンプレート内で何度も使う値をパラメータにすると、その値を何度も書かずに済みます。  
また、テンプレート自体は変えずに、デプロイのときにパラメータの値だけを指定してデプロイできます。

型(`Type`)やデフォルト値(`Default`)、許容される値のリスト(`AllowedValues`)、パラメータの説明(`Description`)を指定できます。

```yaml
Parameters:
  SampleParameter:
    Type: String
    Default: foo
    AllowedValues:
      - foo
      - bar
      - baz
    Description: This is a sample parameter.
```

### Resourcesテンプレート(必須)
作成するリソースです。CFnテンプレートで唯一必須なセクションであり、おそらく最も重要なセクションです。

テンプレートを一から書くときは、以下のページを見て、自分の作りたいリソースでどのようにResourcesを書けばいいのかを頑張って調べることになります。  
[AWS resource and property types reference - AWS CloudFormation](https://docs.aws.amazon.com/en_us/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

大雑把な構成は以下のようになります。

```yaml
Resources:
  <Logical ID>:
    Type: <Resource Type>
    Properties: 
      ...
```

`<Logical ID>` は自分の好きな名前を英数字でつけます。これはテンプレート内で、他のリソースから参照するために使われます。そのため、テンプレート内で一意にする必要があります。

`<Resource Type>` はリソースタイプを書きます。これはリソースのドキュメントを見て書きます。  
Propertiesの中身もドキュメントを見て書きます。この部分はリソースの設定です。

### Outputsセクション(任意)
作成したリソースの情報などを出力したり、別のCFnテンプレートに公開する値を設定します。

作成するリソースによっては、テンプレートにはないランダム文字列がリソースの名前に使われることがあります。そうすると、そのテンプレートによって何のリソースが作成されたかがわからなくなるおそれがあります。このようなときに、作成したリソース名(やARN)を出力し、記録しておくことができます。  
また、 **クロススタック参照** でこのセクションが使われます。CFnテンプレートによってAWSリソースを作成したとき、そのテンプレートのリソース情報などを別のテンプレートでも使いたい場合があります。そのようなときにこのセクションを書きます。

```yaml
Outputs:
  <Logical ID>:
    Description: This is a sample output.
    Value: foobar
    Export:
      Name: Sample-Output
```

`<Logical ID>` は自分の好きな名前を英数字でつけます。これはテンプレート内で、他のリソースから参照するために使われます。そのため、テンプレート内で一意にする必要があります。  
`Value` は出力したい値を書きます。  
`Export` は別のテンプレートから参照できるよう公開するものを書きます。 `Name` は別のテンプレートから参照するときのキーとなります。

## テンプレートについてその他のトピック
### 組み込み関数
[組み込み関数リファレンス - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

CFnテンプレートで使える関数がいくつか用意されています。パラメータを使う場合などは必須です。  
よく使うものを書きます。

- Ref  
  パラメータの値を参照したりします。
- Fn::Sub  
  文字列に挿入したパラメータの値を展開したりします。

### 擬似パラメータ
[擬似パラメータ参照 - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html)

Parametersセクションには自分で定義したパラメータを書きますが、CFnに元々定義されているパラメータがあります。これをテンプレートの中で使うことがあります。  
これもよく使うものを書きます。

- AWS::AccountId  
  AWSアカウントIDを返します。
- AWS::Region  
  AWSリージョンを返します。
