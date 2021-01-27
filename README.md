# Telepresenceハンズオン的なもの

## Telepresenceとは？

ローカルでコンテナを起動し、ポートフォワード を使ってリモートk8sクラスタのDeploymentと置き換えるツール  
commit, pushのフローを経ずに実際のクラスタを使った挙動の確認ができるので、開発サイクルの高速化が期待できる  

ソースコードをマウントしてのホットリロードと組み合わせると便利そう  
(他の人も使うクラスタなら独占しすぎに注意)  

## 用意するもの

- [kubectlのインストール](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)
- [Dockerのインストール](https://docs.docker.com/get-docker/)
- [kindのインストール](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [telepresenceのインストール](https://www.telepresence.io/reference/install) (実行に `python3` が必要なので、python3が動いていることを確認してください)

## 実施手順

### 下準備

```
# kindでクラスタ構築
> kind create cluster --config cluster.yaml
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂

# deployを作成してサンプルアプリケーションを立てる
> kubectl apply -f deploy-2.yaml
deployment.apps/deploy-sample created

# serviceの作成
> kubectl apply -f svc.yaml
service/svc-sample created

# nginx-ingress-controllerを適用
> kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created

# nginx-ingress-controllerが適用されるまで待機
> kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
pod/ingress-nginx-controller-55bc59c885-7jcmn condition met

# Ingressの作成
> kubectl apply -f ingress.yaml
ingress.networking.k8s.io/ingress-sample created

# ingressを通してリクエストが到達することを確認
> curl http://localhost/ip
10.244.1.7
```

### Teleperesenceを使ってみる

```
> cd app_container
> vim main.go
```

`/ip` の返す内容を変更してみる

```
> docker build -t demo .
Sending build context to Docker daemon  13.82kB
Step 1/8 : FROM golang:1.15.6 as builder
 ---> 5f9d35ce5cfe
Step 2/8 : WORKDIR /go/src/app
 ---> Using cache
 ---> 330c59a08725
Step 3/8 : COPY . .
 ---> Using cache
 ---> abdfb695110d
Step 4/8 : RUN set -x &&     go mod download &&     CGO_ENABLED=0 GOOS=linux go build -o app .
 ---> Using cache
 ---> f6eb1fc3bc1b
Step 5/8 : FROM alpine:latest
 ---> d6e46aa2470d
Step 6/8 : EXPOSE 9200
 ---> Using cache
 ---> 3f53b79d86db
Step 7/8 : COPY --from=builder /go/src/app/app /app
 ---> Using cache
 ---> d46c9ad6c331
Step 8/8 : CMD ["/app"]
 ---> Using cache
 ---> c490f42a1d80
Successfully built c490f42a1d80
Successfully tagged demo:latest

# クラスタのデプロイメントが削除されて置き換えられる過程を楽しめる
> kubetl get deploy -w

# teleprecenseを使ってdeploy-sampleと置換
> telepresence --swap-deployment deploy-sample --expose 9200 --docker-run --rm -ti demo

# 別のシェルから投げて置き換えられていることを確認
> curl http://localhost/ip
modified: 10.244.2.12

# お片づけ
> kind delete cluster

```
