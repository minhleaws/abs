

# Installation Guide

## Install Nginx controller

Reference: [Nginx ingress controller](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/)

### Deploy ingress controller

Pull nginx ingress controller code from github repo:

```sh
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v2.4.1
cd kubernetes-ingress/deployments
```

Apply manifests to create ingress controller

```sh
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f rbac/rbac.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f common/crds/k8s.nginx.org_transportservers.yaml
kubectl apply -f common/crds/k8s.nginx.org_policies.yaml
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml
```

For default nginx controller will create an ingress class name nginx. To change ingres class name please update file below

```sh
vi common/ingress-class.yaml
```

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: nginx # Change this to new ingress class name
spec:
  controller: nginx.org/ingress-controller
```

Apply ingress class

```sh
kubectl apply -f common/ingress-class.yaml
```

Deploy nginx deployment

```sh
kubectl apply -f deployment/nginx-ingress.yaml
```

Deploy service

For deploy on cloud use service type loadbalancer

```sh
kubectl apply -f service/loadbalancer.yaml
```

### Get ingress controller IP

```sh
kubectl get service -n nginx-ingress
```

Get external ip for nginx ingress load balancer in EXTERNAL-IP column for using it later

## Install Cert Manager


Create a namespace for cert-manager and test namespace:

```sh
kubectl create ns cert-manager
kubectl create ns test
```

Install cert-manager:
```sh
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.1/cert-manager.yaml
```

## Create ClusterIssuer

Reference: [Issuer Documentation](https://cert-manager.io/docs/configuration/acme/)

Create a file `cluster-issuer.yaml` with the following content:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # Replace this email address with your own.
    email: minh.le@gmail.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-issuer-key
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx # Change it if using diffrent nginx class
```

Apply the ClusterIssuer:
```sh
kubectl apply -f cluster-issuer.yaml
```

## Create Certificate

Create a file `certificate.yaml` with the following content:
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: letsencrypt-cert
  namespace: test
spec:
  secretName: letsencrypt-tls
  dnsNames:
  - 42.116.7.14.nip.io # For testing purpose using nip.io with nginx ingress load balancer external ip
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

Apply the Certificate:
```sh
kubectl apply -f certificate.yaml
```

## Test Application

### Deployment

Create a file `nginx-deployment.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: test
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.23.2
        ports:
        - containerPort: 80
```

Apply the Deployment:
```sh
kubectl apply -f nginx-deployment.yaml
```

### Service

Create a file `nginx-service.yaml` with the following content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: test
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

Apply the Service:
```sh
kubectl apply -f nginx-service.yaml
```

### Ingress

Create a file `nginx-ingress.yaml` with the following content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: test
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  tls:
  - hosts:
      - 42.116.7.14.nip.io # For testing purpose using nip.io with nginx ingress load balancer external ip
    secretName: letsencrypt-tls
  rules:
  - host: 42.116.7.14.nip.io # For testing purpose using nip.io with nginx ingress load balancer external ip
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 8080
```

Apply the Ingress:
```sh
kubectl apply -f nginx-ingress.yaml
```

## Conclusion

This guide has demonstrated how to install cert-manager, create a ClusterIssuer, Certificate, deploy a test application with Nginx and 42.116.7.14 is my IP Address. If you have any questions or issues, please contact me.

