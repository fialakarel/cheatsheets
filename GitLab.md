# GitLab

## How to add existing k3s into GitLab

1. Get API URL.
```bash
kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'
```

2. Get CA certificate from secret called `default-token-xxxxx`.
```bash
kubectl get secret $(kubectl get secrets | grep -o "default-token-.[a-z0-9]*") -o jsonpath="{['data']['ca\.crt']}" | base64 --decode
```

3. Create service account for you GitLab.

```bash
cat <<EOF | kubectl apply -f -
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitlab-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: gitlab-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: gitlab-admin
  namespace: kube-system
EOF
```

4. Get the authentication token.
```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep gitlab-admin | awk '{print $1}')
```


## How to pull Docker image from private GitLab registry

If you need to pull Docker image from a private Docker registry you need to instruct your Kubernetes cluster on how to do that and provide the proper credentials.

There are two options of granting permissions `cluster-wide` and `per-deployment`.

* The first step is to create the Kubernetes secret with your credentials/tokens.

```bash
kubectl create secret \
    docker-registry gitlabdockersecret \
    --docker-server=registry.gitlab.com \
    --docker-username=<USERNAME> \
    --docker-password=<YOUR_TOKEN> \
    --docker-email=<EMAIL>
```

The `docker-password` can be your real GitLab password or your **personal access token (preferred)** with `read_registry` scope.

### cluster-wide

You need to "patch" your service account to pass your credentials into each deployment.

```bash
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gitlabdockersecret"}]}'
```

### per-deployment

You just need to add `imagePullSecrets` into your deployment.

```yaml
imagePullSecrets:
       - name: gitlabdockersecret
```

### References

* https://gitlab.com/profile/personal_access_tokens
* https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account
* https://docs.gitlab.com/ee/user/project/deploy_tokens/index.html
