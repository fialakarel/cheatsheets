# Kubernetes

## Helm

### Deploy nginx ingress controller

```bash
# Add helm repository with stable nginx
helm repo add stable https://kubernetes-charts.storage.googleapis.com

# Update helm repository
helm repo update

# Install nginx-ingress controller
helm install nginx-ingress stable/nginx-ingress \
    --set controller.replicaCount=2 \
    --namespace kube-system
```

### Deploy cert-manager

```bash
# Add jetstack repo
helm repo add jetstack https://charts.jetstack.io

# Update helm repository
helm repo update

# Create namespace for cert-manager
kubectl create namespace cert-manager

# Install cert-manager
helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --version v0.15.1 \
    --set installCRDs=true \
    --set ingressShim.defaultIssuerName=letsencrypt-prod \
    --set ingressShim.defaultIssuerKind=ClusterIssuer

# Create ClusterIssuer
cat <<EOF | kubectl apply -f -
# Issuer issue the new certificate request
apiVersion: cert-manager.io/v1alpha2
#kind: Issuer
kind: ClusterIssuer
metadata:
  #name: letsencrypt-staging
  name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL -- staging or production
    #server: https://acme-staging-v02.api.letsencrypt.org/directory
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: <YOUR EMAIL>
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      #name: letsencrypt-staging
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

## AKS -- Azure Kubernetes Service

### Init kubectl

First, you need to run `az login` then the commands below should work.

```bash
subscription="<AZURE_SUBSCRIPTION>"
resource_group="<AZURE_RESOURCE_GROUP>"
aks_name="<AZURE_AKS_NAME>"

# List subscriptions
az account list --output table

# Set Agrifac subscription
az account set --subscription $subscription

# Get kubectl credentials
az aks get-credentials --resource-group $resource_group --name $aks_name

# Set kubectl context
kubectl config use-context $aks_name

# Get cluster-info to see if it is working
kubectl cluster-info

# List nodes to see if it is working
kubectl get nodes
```

### Dashboard

```bash
# Enable Kubernetes dashboard
kubectl create clusterrolebinding kubernetes-dashboard \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:kubernetes-dashboard

# To open dashboard use
az aks browse --resource-group $resource_group --name $aks_name
```

## Kubernetes resources

### Deployment example

```yaml
---
kind: Deployment
apiVersion: apps/v1 
metadata:
  name: testapp-dpl
spec:
  replicas: 2
  selector:
    matchLabels:
      app: testapp
  template:
    metadata:
      labels:
        app: testapp
    spec:
      containers:
      - image: nginx:alpine
        name: testapp-http
        ports:
        - name: http
          containerPort: 80
        volumeMounts:
          - name: volv
            mountPath: /data
      volumes:
        - name: volv
          persistentVolumeClaim:
            claimName: dataclaimname
      restartPolicy: Always
```


### Service example

```yaml
kind: Service
apiVersion: v1
metadata:
  name: testapp-svc
spec:
  selector:
    app: testapp
  ports:
  - port: 80
    targetPort: http
```


### Ingress example

```yaml
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: testapp-ing
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
    nginx.ingress.kubernetes.io/tls-acme: "true"
    nginx.ingress.kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
    certmanager.k8s.io/acme-http01-edit-in-place: "true"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
#    nginx.ingress.kubernetes.io/rewrite-target: "/$1"
spec:
  rules:
  - host: <YOUR_DOMAIN>
    http:
      paths:
      - path: /
        backend:
          serviceName: testapp-svc
          servicePort: 80
  tls:
  - secretName: tlssecret
    hosts:
    - <YOUR_DOMAIN>
```

### Persistent Volume Claim example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dataclaimname
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 1G
```

### Secret example

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tcs-api-token
type: Opaque
data:
  TCS_API_TOKEN: <put-here-base64-of-your-TCS_API_TOKEN>
```

### CronJob example

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cjname
spec:
  schedule: "0 2 * * 2,3,4,5,6"
  jobTemplate:
    spec:
      activeDeadlineSeconds: 7200
      template:
        spec:
          containers:
          - name: containername
            image: registry.gitlab.com/foobar:latest
            imagePullPolicy: Always
            env:
            - name: TCS_API_TOKEN
              valueFrom:
                secretKeyRef:
                  name: tcs-api-token
                  key: TCS_API_TOKEN
            - name: TCS_WORKERS
              value: "4"
            - name: HC_URL
              value: "<PUT-HERE-healthchecks.io-URL>"
            volumeMounts:
              - name: volv
                mountPath: /app/data
          volumes:
            - name: volv
              persistentVolumeClaim:
                claimName: dataclaimname
          restartPolicy: OnFailure
```

### Create Job from CronJob

```bash
kubectl create job --from=cronjob/cjname cjname-manual-1
```
