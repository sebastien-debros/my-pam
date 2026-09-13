# Teleport Database Service Agent

Wraps the official `teleport-kube-agent` chart to run a Teleport Database
Service (`roles: db`) that joins the existing `teleport.local` cluster
deployed by `../my-teleport` and proxies access to the PostgreSQL instance
deployed by `../my-postgres`.

## Deployment

### 0. Prerequisites

Deploy `my-teleport` and `my-postgres` first.

### 1. Update chart dependencies

```bash
helm dependency update .
```

### 2. Allow this agent to join the cluster

Find the Teleport auth pod:

```bash
kubectl get pods -n teleport
```

Create a static join token for the `Db` role, restricted to this agent's
service account (mirrors how the proxy already joins — see the
`my-teleport-proxy` token in the `my-teleport-auth` ConfigMap):

```bash
kubectl exec -it <my-teleport-auth-pod> -n teleport -- tctl create -f - <<'EOF'
kind: token
version: v2
metadata:
  name: my-teleport-db
  expires: "2050-01-01T00:00:00Z"
spec:
  roles: [Db]
  join_method: kubernetes
  kubernetes:
    allow:
      - service_account: "teleport:my-teleport-db"
EOF
```

### 3. Make Postgres's CA available to this agent

Copy the `server.cas` file generated for `my-postgres` (see its README,
`postgres-certs/tmp/postgres.cas`) into a secret in the `teleport` namespace:

```bash
kubectl create secret generic postgres-ca -n teleport \
  --from-file=server.cas=postgres-certs/tmp/postgres.cas
```

### 4. Install the chart

```bash
helm install my-teleport-db-agent . --namespace teleport
```

### 5. Verify

```bash
kubectl get pods -n teleport
kubectl logs -n teleport deploy/my-teleport-db -f
```

The agent should reach `Running` and its logs should show it joined the
cluster and registered the `postgres` database. Confirm from the auth pod:

```bash
kubectl exec -it <my-teleport-auth-pod> -n teleport -- tctl get db
```
