# ECRリポジトリにDockerイメージをpushする
このページはCFnとは関係ありません。いま作成したECRリポジトリにDockerイメージをpushしてみようというものです。

方法は[Docker イメージをプッシュする - Amazon ECR](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/docker-push-ecr-image.html)に従います。

このページの内容は、どこかに新しくディレクトリを作り、その中で行ってください。(`docker build` のときにエラーになるかもしれないため。)

## Dockerfile
`Dockerfile` というファイルを作成し、以下の内容を書きます。

```Dockerfile
FROM debian:buster-slim
ENV TZ=Asia/Tokyo
CMD ["date"]
```

## Dockerイメージにつけるタグの確認
以下のコマンドを実行してください。

```sh
aws ecr describe-repositories
```

するとECRリポジトリの情報が表示されます。この中から、今回作成したリポジトリを見つけ、 `repositoryUri` の値を見てください。  
それがDockerイメージにつけるタグです。

## プライベートレジストリの認証
[プライベートレジストリの認証 - Amazon ECR](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/registry_auth.html)に書いてある通りやります。

以下のコマンドの `<REGION>` と `<AWS_ACCOUNT_ID>` は自分の環境に置き換えて実行します。  
(上で調べたタグを見れば同じような文字列があるので、それを参考にできます。)

```sh
aws ecr get-login-password --region <REGION> | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com
```
"Login Succeeded" が出ればOKです。

もしAWS CLIとdockerコマンドが同じ環境にないときは、上のようにパイプでコマンドをつなぐ代わりに
```sh
aws ecr get-login-password --region <REGION> > /tmp/ecr_auth.txt
cat /tmp/ecr_auth.txt | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com
```
とすればいいです。  
(ただし実際に業務などで使うときは、このようにファイルに書き出すのはやめた方がいいと思います。パイプでつなげば認証文字列はどこにも残りませんが、ファイルに残すと、そこから漏れる恐れが生じてしまうからです。)

## Dockerイメージのビルド
```sh
docker build -f ./Dockerfile -t <上で調べたタグ>
```

## ECRリポジトリへのpush
```sh
docker push <上で調べたタグ>
```

## 確認
AWSコンソールでECRの、先ほど作成したリポジトリを見ます。するとイメージが `latest` という名前で登録されています。
