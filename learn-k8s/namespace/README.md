# 네임스페이스 구성


## 1. 네임스페이스 생성

논리적 테넌트 구성을 위해 네임스페이스를 구성

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: $NS
  labels:
    kubernetes.io/namespace: $NS
```

## 2. 리소스쿼터 생성

네임스페이스 내에서 사용할 리소스 쿼터 생성

```yaml
kind: ResourceQuota
metadata:
  name: $NS
  namespace: $NS
spec:
  hard:
    limits.cpu: "2"
    limits.memory: 6Gi
```

## 3. 네트워크 폴리시 생성

- default deny
- namespace 내에서의 통신은 허용
- next ? : https://github.com/ahmetb/kubernetes-network-policy-recipes


### 네트워크 폴리시 초단간 실습

#### access=granted 레이블이 붙은 파드만 nginx 파드와 통신이 가능하도록 네트워크 폴리시 구성해보기

nginx 디플로이먼트 및 서비스 생성
`kubectl run nginx --image=nginx`      
`kubectl expose deployment nginx --port=80`

접근 되는것 확인
```
$ kubectl run busybox --rm -ti --image=busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.152.183.242:80)
/ #
```

네트워크 폴리시 구성후 접근 안되는 것 확인
```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "granted"
```

###  busybox 파드 생성후 nginx 접근 시도해보기 - 결과 block 처리됨

```
$ kubectl run busybox --rm -ti --image=busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.10.10.123:80)
wget: download timed out  <------ this is what we want
```

###  busybox 파드에 레이블(access=granted)을 붙인후 다시 시도 - 결과 : accecss 성공

```
$ kubectl run busybox --rm -ti --labels="access=granted" --image=busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget --spider --timeout=1 nginx
Connecting to nginx (10.152.183.242:80)
/ #
```

### 4. 실제 구성

```sh
export NS=test
envsubst < namespace.yaml | kubectl apply -f -
```
