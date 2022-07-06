# 介绍
本仓库包含以下内容
- 如何在生产环境的 `Kubernetes` 上安装 `Tyk Gateway` 和 `Tyk Pump`
- 如何使用在 `Tyk Gateway` 上创建 `API` 和 `Policy`
- 如何使用 `Tyk Gateway API` 创建 `Key`
- 如何使用 `Tyk Pump` 对 `Tyk Gateway` 进行监控

# 安装 `Tyk Gateway` 和 `Tyk Pump`
## 创建命名空间
```
kubectl create ns tyk
```

## 安装 Redis
下面的指令将会安装 Redis Cluster, 生产环境中最好不要自己管理 Redis Cluster。 而是使用云服务厂商提供的 Redis Cluster 会更加稳妥
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install redis bitnami/redis-cluster --namespace tyk
```

## 安装 Tyk Gateway + Pump

- `tyk-helm/tyk-headless` 中包含了 Tyk Gateway 和 Tyk Pump 两个组件
- `tyk-pump` 不支持 Mongo 5.x,  需要使用 4.x 或者使用其他的后端数据引擎

```
export REDIS_PASSWORD=$(kubectl get secret -n tyk redis -o jsonpath="{.data.redis-password}" | base64 -d)

helm install tyk ./tyk-headless -n tyk \
    --set secrets.APISecret=management-api-secret \
    --set "redis.cluster.enabled=true" \
    --set "redis.cluster.addrs[0]=redis-cluster.tyk.svc.cluster.local:6379" \
    --set redis.password=$REDIS_PASSWORD \
```