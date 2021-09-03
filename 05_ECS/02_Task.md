# ECSタスクの作成
ECSタスクは設定項目が多いです。また、ECRリポジトリやIAMといった他のリソースを参照することになります。

## テンプレート
`task.yaml` として作成します。

### TaskDefinition
ECSタスク定義です。  
[AWS::ECS::TaskDefinition - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-taskdefinition.html)を参考に書きます。  
また、この中のContainerDefinitionを書くときは[AWS::ECS::TaskDefinition ContainerDefinition - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ecs-taskdefinition-containerdefinitions.html)

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: "Creates ECS Task"

Parameters:
  ECRImage:
    Type: String
    Description: "ECS image"

  ECSTaskName:
    Type: String
    Default: DefaultECSTask
    Description: "ECS task name"

  ECSTaskCommand:
    Type: String
    Description: "ECS task command"

  ECSTaskExecutionRoleArn:
    Type: String
    Description: "ExecutionRoleArn for ECS Task"

  ECSTaskRoleArn:
    Type: String
    Description: "TaskRoleArn for ECS Task"

  CpuConfig:
    Type: String
    Description: "CPU Setting for ECS Task"

  MemoryConfig:
    Type: String
    Description: "Memory Setting for ECS Task"

  LogRetentionInDays:
    Type: Number
    Description: "Log retention days"


Resources:
  ECSTaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      ContainerDefinitions:
        -
          Command:
            - !Ref ECSTaskCommand
          Environment:
            -
              Name: HELLO
              Value: "Hello, World!"
          Image: !Ref ECRImage
          LogConfiguration:
            LogDriver: awslogs
            Options:
              awslogs-group: !Ref ECSLogGroup
              awslogs-region: !Ref "AWS::Region"
              awslogs-stream-prefix: !Ref ECSTaskName
          Name: !Ref ECSTaskName
      Cpu: !Ref CpuConfig
      ExecutionRoleArn: !Ref ECSTaskExecutionRoleArn
      Memory: !Ref MemoryConfig
      NetworkMode: awsvpc
      RequiresCompatibilities:
        - FARGATE
      TaskRoleArn: !Ref ECSTaskRoleArn

  ECSLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub "/ecs/logs/${ECSTaskName}-log-group"
      RetentionInDays: !Ref LogRetentionInDays
```

チェックOKです。
```sh
$ cfn-lint task.yaml
```

## デプロイ
`<...>` は置き換えてください。

```sh
aws cloudformation deploy \
  --template-file ./task.yaml \
  --stack-name simple-ecs-task-stack \
  --parameter-overrides \
    ECRImage=<上で調べたタグ> \
    ECSTaskName=SampleTask \
    ECSTaskCommand=date \
    ECSTaskExecutionRoleArn=<ExecutionRoleArn> \
    ECSTaskRoleArn=<ECSTaskRoleArn> \
    CpuConfig=256 \
    MemoryConfig=512 \
    LogRetentionInDays=30
```