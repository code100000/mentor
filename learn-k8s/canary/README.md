# 카나리 구성

카나리 적용 방법은 여러가지
1. 동일 k8s service, pod 버전을 여러개 배포하고 canary 개수 차등 적용
2. istio traffic routing
  - 장점 : 적용 쉬움
  - 단점 : 네트워크 agent 오버헤드 (envoy proxy), 장애 발생시 proxy 구간 확인 필요함
3. ingress canary annotation 사용

이하는 3번 방법에 대한 가이드

## Prerequisite

ingress controller 구성

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/cloud/deploy.yaml
```

## 1. v1 서비스 띄우기


```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: app
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: app
  replicas: 1
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: human537/cicdtest:5
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: app
  labels:
    app: app
spec:
  ports:
  - name: http
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: app
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app
            port:
              number: 8080
```


## 2. v2 서비스 띄우기

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: app-v2
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: app-v2
  replicas: 1
  template:
    metadata:
      labels:
        app: app-v2
    spec:
      containers:
      - name: app-v2
        image: human537/cicdtest:v2
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: app-v2
  labels:
    app: app-v2
spec:
  ports:
  - name: http
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: app-v2
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress-v2
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-v2
            port:
              number: 8080
```


## 3. 테스트

```
for i in {1..10}; do curl -H "Host: canary.example.com" \
http://<IP_ADDRESS>:80/version; done;
```

```sh
for i in {1..10}; do curl http://a14d6e16aaaec448bbac25cc5769316c-1825264213.us-east-1.elb.amazonaws.com:80/version; done;
1.0.0
1.0.0
1.0.0
2.0.0
1.0.0
1.0.0
2.0.0
1.0.0
1.0.0
1.0.0
```

> 참고 :  https://docs.mirantis.com/mke/3.5/ops/deploy-apps-k8s/nginx-ingress/configure-canary-deployment.html