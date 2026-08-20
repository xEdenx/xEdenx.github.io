---
title: Minikube
description: 使用 Minikube 搭建本地 Kubernetes 开发环境。
publishDate: 2019-07-09 15:46:11
tags:
  - docker
---

## 安装virtual box

## 安装 minikube

访问 https://github.com/AliyunContainerService/minikube

访问 https://yq.aliyun.com/articles/221687

## 创建 Secret（用于添加私有仓库）

```shell
kubectl create secret docker-registry regsecret --docker-server=ccr.ccs.tencentyun.com --docker-username=10000 --docker-password=T439439439 --docker-email=tn@gmail.com
```

## 创建 Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rbac-default-namespace
subjects:
  - kind: ServiceAccount
    name: default
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

## 创建 ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: register
  labels:
    app: register
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: backend
    matchExpressions:
      - { key: tier, operator: In, values: [backend] }
  template:
    metadata:
      labels:
        app: register
        tier: backend
    spec:
      containers:
        - name: register
          image: ccr.ccs.tencentyun.com/demo/register-k8s:2.0.3
          ports:
            - containerPort: 8080
      imagePullSecrets:
        - name: regsecret
```
