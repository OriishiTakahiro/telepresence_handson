# Telepresenceハンズオン的なもの

## 用意するもの

- [kubectlのインストール](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)
- [Dockerのインストール](https://docs.docker.com/get-docker/)
- [kindのインストール](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [k8sのリソースリポジトリ](https://github.com/OriishiTakahiro/k8s_handson)のクローン

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

# お片づけ
> kind delete cluster

```
