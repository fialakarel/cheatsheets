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
