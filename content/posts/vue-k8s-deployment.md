---
title: VUE前端k8s部署
description: 将 Vue 前端应用构建并部署到 Kubernetes 的实践。
publishDate: 2021-03-04 09:59:56
tags:
  - k8s
---

pipeline

```
pipeline {
  agent {
    node {
      label 'nodejs'
    }

  }
  stages {

    stage('代码获取') {
      steps {
        git(url: 'http://git.eden.io/standardization/fe-cloud.git', credentialsId: 'gitlab-id', branch: '${BRANCH}', changelog: true, poll: false)
      }
    }

    stage('编译构建') {
      steps {
        container('nodejs') {
          sh 'npm config set proxy http://squid:squid@10.15.15.15:3128'
          sh 'npm config set registry http://registry.npm.taobao.org/'
          sh 'npm install --ignore-scripts'
          sh 'npm install node-sass --save'
          sh 'npm install '
          sh 'npm run build'
          withCredentials([usernamePassword(credentialsId : 'harbor-id' ,passwordVariable : 'PASSWORD' ,usernameVariable : 'USERNAME' ,)]) {
            sh 'docker login -u $USERNAME -p $PASSWORD image.onecode.ict.cmcc:31104'
            sh 'docker build -t eden.io/standardization/fe-cloud-$BRANCH:$BUILD_NUMBER .'
            sh 'docker push eden.io/standardization/fe-cloud-$BRANCH:$BUILD_NUMBER'
          }

        }

      }
    }

    stage('部署') {
      steps {
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: 'kubeconfig', configs: 'deploy/**')
      }
    }

  }
}
```

Dockerfile

```
FROM eden.io/library/nginx:1.14-alpine
ADD ./dist /usr/share/nginx/html/web
CMD ["nginx","-g","daemon off;"]
```

nginx.conf

```
user root;
worker_processes 1;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;

  sendfile on;
  #tcp_nopush     on;

  keepalive_timeout 65;

  #gzip  on;
  server {

    listen 80;
    server_name fe-cloud;

    location / {
      root /usr/share/nginx/html/web;
      index index.html index.htm;
      # try_files $uri $uri/ /index.html;
    }

    location ^~/api/ {
      proxy_pass http://gateway:8080/;
      proxy_redirect default;
    }

    location ^~/web {
      alias /usr/share/nginx/html/web/;
      index index.html;
    }

  }
}
```

deployment.yaml

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: fe-cloud-v1
  namespace: standardization
  labels:
    app: fe-cloud
    app.kubernetes.io/name: standardization
    app.kubernetes.io/version: v1
    version: v1
  annotations:
    deployment.kubernetes.io/revision: "12"
    kubesphere.io/creator: Eden
    servicemesh.kubesphere.io/enabled: "false"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fe-cloud
      app.kubernetes.io/name: standardization
      app.kubernetes.io/version: v1
      version: v1
  template:
    metadata:
      labels:
        app: fe-cloud
        app.kubernetes.io/name: standardization
        app.kubernetes.io/version: v1
        version: v1
      annotations:
        kubesphere.io/containerSecrets: '{"container-uxt0my":"harbor-secret"}'
        logging.kubesphere.io/logsidecar-config: "{}"
        sidecar.istio.io/inject: "false"
    spec:
      volumes:
        - name: volume-n2kn8y
          configMap:
            name: fe-cloud-conf
            # items 这里是重点
            items:
              - key: nginx.conf
                path: nginx.conf
            defaultMode: 420
      containers:
        - name: container-uxt0my
          image: "eden.io/standardization/fe-cloud-$BRANCH:$BUILD_NUMBER"
          ports:
            - name: http-80
              containerPort: 80
              protocol: TCP
          resources: {}
          # volumeMounts 这里是重点
          volumeMounts:
            - name: volume-n2kn8y
              readOnly: true
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: default
      serviceAccount: default
      securityContext: {}
      imagePullSecrets:
        - name: harbor-secret
      affinity: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```
