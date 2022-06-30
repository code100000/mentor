# helm

## 1. helm client install
- k8s 호환성 버전 확인 : [Helm Version Support Policy](https://helm.sh/docs/topics/version_skew/)

[Helm Install](https://helm.sh/docs/intro/install/)

```sh
curl -O https://get.helm.sh/helm-v3.6.0-linux-amd64.tar.gz
tar -zxvf helm-v3.6.0-linux-amd64.tar.gz
mv linux-amd64/helm /usr/bin/helm
```

### 1.1 환경 구성

```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

$ helm search repo bitnami
```

## 2. tomcat 설치

### 2.1. 기본 설치

```sh
$ helm install bitnami/tomcat --generate-name
```

```sh
$ helm ls

NAME             	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
tomcat-1656579749	default  	1       	2022-06-30 09:02:31.752293571 +0000 UTC	deployed	tomcat-10.3.9	10.0.22
```

### 2.2. tar 설치

```sh
$ helm pull bitnami/tomcat
$ tar xvf tomcat-10.3.9.tgz
$ vi values.yaml
$ helm template -f values.yaml .
$ helm install hybrid -f values.yaml .
$ helm upgrade -f values.yaml hybrid .
export TOMCAT_PASSWORD=$(kubectl get secret --namespace "default" hybrid-tomcat -o jsonpath="{.data.tomcat-password}" | base64 -d)
$ helm upgrade -f values.yaml --set tomcatPassword=$TOMCAT_PASSWORD hybrid .
$ kubectl get svc
```

```sh
$ helm status hybrid
$ helm ls
$ helm rollback hybrid 1
$ kubectl get svc
$ helm uninstall hybrid
$ helm ls
```

## 3. clean up 

$ helm uninstall tomcat-1656579749
