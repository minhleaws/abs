# Installation Guide

## Install kasten-io

Register helm repository

```sh
helm repo add kasten https://charts.kasten.io/
helm repo update
```

Install kasten-io

```sh
kubectl create namespace kasten-io
helm install k10 kasten/k10 --namespace=kasten-io
```

Check status pod

```sh
kubectl get pods -n kasten-io
```

> [!IMPORTANT]  
> In case using NFS storageclass can got error permission denied in some pods. In that case change the permission of pv folder 

## Access kasten-io from UI

### Access directly using kubectl

```sh
kubectl port-forward service/gateway 8080:80 -n kasten-io
```

Access using broswer: [Kasten-io](http://127.0.0.1:8080/k10/).

### Access using ingress

For access using ingress kasten required authen. We are using authen for this case

Generate basic authen string 

```sh
htpasswd -bn kasten kasten
```

Update helm to setting ingress 

```sh
helm upgrade k10 kasten/k10 --namespace=kasten-io \
--set ingress.create=true \
--set ingress.class=<ingressclass> \
--set ingress.host=<domain> \
--set auth.basicAuth.enabled=true \
--set auth.basicAuth.htpasswd='<httpasswd string>' 
```
Access using broswer: domain/k10/
