# 介绍
本仓库是关于如何在 K8s 上安装 Tyk Quick Start. 可根据需求自行修改.


# 创建命名空间
```
kubectl create ns tyk
```

# 安装 Redis
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install redis bitnami/redis-cluster --namespace tyk
```

# 安装 Mongodb
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mongo bitnami/mongodb --namespace tyk
```

# 安装 Tyk Gateway + Pump

- `tyk-helm/tyk-headless` 中包含了 Tyk Gateway 和 Tyk Pump 两个组件
- `tyk-pump` 可能会不支持 Mongo 5.x,  需要使用 4.x 或者使用其他的后端数据引擎

```
helm repo add tyk-helm https://helm.tyk.io/public/helm/charts/
helm repo update

export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace tyk mongo-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d)
export REDIS_PASSWORD=$(kubectl get secret -n tyk redis -o jsonpath="{.data.redis-password}" | base64 -d)

helm install tyk ./tyk-headless -n tyk \
    --set secrets.APISecret=management-api-secret \
    --set "redis.cluster.enabled=true" \
    --set "redis.cluster.addrs[0]=redis-cluster.tyk.svc.cluster.local:6379" \
    --set redis.password=$REDIS_PASSWORD \
    --set mongo.enabled=true \
    --set "mongo.mongoURL=mongodb://root:$MONGODB_ROOT_PASSWORD@mongo-mongodb.tyk.svc.cluster.local:27017/tyk_analytics?authSource=admin" \
    --set "pump.enabled=true"
```