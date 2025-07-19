# cdevops-gitea

k8s gitea lab to take dev (sqlite based) to prod (mysql based)

Implementation of three requirements for transitioning Gitea from development to production:

## 1. Persistent Repository Data

**Implementation** (`prod/values.yaml`):

```yaml
persistence:
  enabled: true # Changed from false (dev) to true (prod)
  size: 10Gi # 10GB storage allocation
  accessModes:
    - ReadWriteOnce # Single node read-write access
```

## 2. External Database

**PostgreSQL Deployment** (`prod/up.yml`):

```yaml
- name: Deploy external PostgreSQL database
  kubernetes.core.helm:
    name: gitea-postgresql
    chart_ref: bitnami/postgresql
    values:
      auth:
        postgresPassword: 'gitea123'
        database: 'gitea'
      primary:
        persistence:
          enabled: true
          size: 8Gi
```

**Gitea Configuration** (`prod/values.yaml`):

```yaml
postgresql:
  enabled: false # Disable bundled database

gitea:
  config:
    database:
      DB_TYPE: postgres # Changed from sqlite3
      HOST: gitea-postgresql-postgresql:5432 # External service endpoint
      NAME: gitea
      USER: postgres
      PASSWD: gitea123
```

## 3. Public Exposure

**Setup**:

1. Create account at https://ngrok.com
2. Get API Key, Domain, and Auth Token from dashboard

**Deploy Ngrok Operator**:

```bash
export NGROK_DOMAIN="your-domain.ngrok-free.app"
export NGROK_AUTHTOKEN="your_auth_token"
export NGROK_API_KEY="your_api_key"

helm repo add ngrok https://charts.ngrok.com
helm install ngrok-operator ngrok/ngrok-operator \
--namespace ngrok-ingress-controller \
--create-namespace \
--set credentials.apiKey=$NGROK_API_KEY \
--set credentials.authtoken=$NGROK_AUTHTOKEN
```

**Create Ingress**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: gitea-public-ingress
spec:
  ingressClassName: ngrok
  rules:
    - host: your-domain.ngrok-free.app
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gitea-http
                port:
                  number: 3000
```

## Deployment Process

**Development vs Production**:

| Component | Development          | Production            |
| --------- | -------------------- | --------------------- |
| Storage   | Ephemeral            | Persistent (10GB)     |
| Database  | SQLite3              | PostgreSQL (external) |
| Access    | Port-forward (local) | Ngrok (public)        |

**Commands**:

```bash
# Deploy production Gitea
ansible-playbook prod-up.yml

# Setup Ngrok
helm install ngrok-operator ngrok/ngrok-operator \
--namespace ngrok-ingress-controller \
--create-namespace \
--set credentials.apiKey=$NGROK_API_KEY \
--set credentials.authtoken=$NGROK_AUTHTOKEN

# Create ingress
kubectl apply -f ingress.yaml
```

TLDR;

```bash
pip install ansible kubernetes
git submodule update --init --recursive
ansible-playbook up.yml
```

Wait until `kubectl get pod` shows all pods running and:

```bash
kubectl port-forward svc/gitea-http 3000:3000
```

Now you should be able to access gitea in development mode.

The challenge is to run this in production mode.

### Points to Cover

## Marking

| Item                                                                                                                   | Out Of |
| ---------------------------------------------------------------------------------------------------------------------- | -----: |
| use [the gitea helm](https://gitea.com/gitea/helm-gitea) to make the repository data persistent                        |      3 |
| make gitea use external database                                                                                       |      3 |
| Use [this article](https://blog.techiescamp.com/using-ngrok-with-kubernetes/) to expose your gitea instance publically |      2 |
| make the README easy to use and ACCURATE                                                                               |      2 |
|                                                                                                                        |        |
| total                                                                                                                  |     10 |
