# Installation Guide

## Install postgres-operator

Pull code from github

```sh
git clone https://github.com/zalando/postgres-operator.git
cd postgres-operator
```

Edit namespace for manifests for custom namespace installation

```sh
vi manifests/configmap.yaml
vi manifests/operator-service-account-rbac.yaml
vi manifests/postgres-operator.yaml
vi manifests/api-service.yaml
```

Apply manifests for following order

```sh
kubectl create -f manifests/configmap.yaml  
kubectl create -f manifests/operator-service-account-rbac.yaml  
kubectl create -f manifests/postgres-operator.yaml  
kubectl create -f manifests/api-service.yaml  
```

Verify pod are running

```sh
kubectl get pod -l name=postgres-operator
```

## Install postgres-operator-ui

Edit namespace for manifests for custom namespace installation

```sh
vi ui/manifests/ui-service-account-rbac.yaml
vi ui/manifests/deployment.yaml
vi ui/manifests/service.yaml
vi ui/manifests/ingress.yaml # optional for expose to internet
```

Apply manifests for following order

```sh
kubectl create -f ui/manifests/ui-service-account-rbac.yaml
kubectl create -f ui/manifests/deployment.yaml
kubectl create -f ui/manifests/service.yaml
kubectl create -f ui/manifests/ingress.yaml # optional for expose to internet
```

Verify pod are running

```sh
kubectl get pod -l name=postgres-operator-ui
```

Quick access to ui service

```sh
kubectl port-forward 0n <namespace> svc/postgres-operator-ui 8081:80 
```

Using localhost:8081 for access to UI

## Create postgres cluster

Create yaml for postgre cluster postgres.yaml

```yaml
apiVersion: "acid.zalan.do/v1"
kind: postgresql
metadata:
  name: postgresql-cluster
  namespace: default # change for diffrent namespace
spec:
  teamId: "acid"
  volume:
    size: 10Gi
    storageClass: nfs # change storage class
  numberOfInstances: 2 # number of instance in cluster
  users:
    admin:  # database owner
    - superuser
    - createdb
    user: []  
  databases:
    database: admin  # dbname: owner
  postgresql:
    version: "14"
```

Apply manifest postgres.yaml

```sh
kubectl create -f postgres.yaml
```