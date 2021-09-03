# ECSクラスタの作成
ECSクラスタは設定が少なく、簡単です。

## CFnテンプレート
[AWS::ECS::Cluster - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-cluster.html)を参考に、ECSクラスタを作成するCFnテンプレートを書きます。ファイル名は `cluster.yaml` とします。  
クラスタ名だけ指定しておけば作成できます。

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Creates ECS Cluster"

Parameters:
  MyECSClusterName:
    Type: String
    Default: DefaultCluster
    Description: "ECS cluster name"

Resources:
  ECSCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: !Ref MyECSClusterName
```

チェックOKです。
```sh
$ cfn-lint cluster.yaml
```

## デプロイ
```sh
aws cloudformation deploy \
  --template-file ./cluster.yaml \
  --stack-name ecs-cluster-stack \
  --parameter-overrides \
    MyECSClusterName=SampleECSCluster
```

成功するとECSの画面のクラスタ一覧に、作成したクラスタが表示されています。
