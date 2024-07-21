Chắc chắn rồi! Dưới đây là phiên bản tiếng Anh của tệp README.md của bạn:

```markdown
# Installation Guide

## Install Cert Manager


Create a namespace for cert-manager:

```sh
kubectl create ns cert-manager
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
          ingressClassName: nginx
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
  - 42.116.7.14.nip.io
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
        env:
        - name: NGINX_PORT
          value: "8080"
        ports:
        - containerPort: 8080
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
      targetPort: 8080
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
spec:
  tls:
  - hosts:
      - 42.116.7.14.nip.io
    secretName: letsencrypt-tls
  rules:
  - host: 42.116.7.14.nip.io
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
```
