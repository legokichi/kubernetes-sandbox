* Skaffoldとhelmとの比較は？ - https://github.com/GoogleContainerTools/skaffold/issues/174
    * Skaffoldは、画像を構築/プッシュし、Kubernetesに展開するためのワークフローをラップするツール
    * `skaffold dev` - ローカルでの multistage dockerfile を使った docker image 開発
    * `skaffold run` - kubectl | helm を使ったデプロイタスク
    * helm ~= npm + package.json
    * skaffold ~= webpack
    *  より詳しい説明 - https://blog.hasura.io/draft-vs-gitkube-vs-helm-vs-ksonnet-vs-metaparticle-vs-skaffold-f5aa9561f948

skaffold で複数の helm リリースを展開する。


```
+ containers
|   + api-server - 今回開発する helm サービスで使う コンテナ
|       + Dockerfile - 今回開発する docker image
|       + web.go - ソースコード
+ charts
|   + api-server - 今回開発する helm サービスの 1 リリース
|       + Chart.yaml
|       + values.yaml
|       + templates
|       + charts
|       + helmignore
+ readme.md - このファイル
+ skaffold.yaml - skaffold を使った開発での成果物(Dockerfile + Docker Image) とデプロイ方法(kubectl|helm) を設定する
```


## ローカルレジストリの用意

* Kubernetesの開発環境で困っているならskaffoldを使え - https://qiita.com/tomoyamachi/items/660bd7bb3afff8340307


```sh
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

* ローカルレジストリの確認 - https://qiita.com/rsakao/items/617f54579278173d3c20#ブラウザでレジストリを確認したい場合docker-registry-frontend
* https://github.com/kwk/docker-registry-frontend

```sh
docker run \
  --link registry \
  -d \
  -e ENV_DOCKER_REGISTRY_HOST=registry \
  -e ENV_DOCKER_REGISTRY_PORT=5000 \
  -p 8080:80 \
  --name registry-frontend \
  konradkleine/docker-registry-frontend:v2
open http://localhost:8080/home
```