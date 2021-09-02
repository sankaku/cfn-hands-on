# デプロイ

[aws cloudformation deploy](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/deploy/index.html)というコマンドを使ってデプロイします。

```sh
aws cloudformation deploy \
  --template-file /path_to_template/template.yaml \
  --stack-name my-new-stack \
  --parameter-overrides \
    Key1=Value1 \
    Key2=Value2 \
  --tags Key1=Value1 Key2=Value2
```