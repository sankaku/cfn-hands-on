# CloudFormationテンプレートの検証
CFnテンプレートを書いたら、実際にデプロイする前に、文法チェックなどができます。

### cfn-lintでチェック
[GitHub - aws-cloudformation/cfn-lint: CloudFormation Linter](https://github.com/aws-cloudformation/cfn-lint)を使う方法です。

```sh
pip install cfn-lint
```
でインストールし、
```sh
cfn-lint sample_template.yaml
```
で構文チェックや、使われていないパラメータがないかなどをチェックします。
