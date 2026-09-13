# PostgreSQL — Target Resource for Teleport Database Access

A minimal PostgreSQL deployment used as a target resource to exercise
**Teleport Database Access**. It requires a valid client certificate signed
by Teleport's Database CA for every connection (mutual TLS), so it cannot be
reached directly — only through Teleport's Database Service.

## Deployment

### 0. Prerequisite

Deploy `my-teleport` first (see `../my-teleport/README.md`) — its auth pod is
needed to issue the TLS certificate below.

### 1. Create the namespace

```bash
kubectl create namespace postgres
```

### 2. Generate the TLS material

Find the Teleport auth pod:

```bash
kubectl get pods -n teleport
```

Sign a server certificate for Postgres from Teleport's Database CA. The auth
image is distroless (no `tar`/`cat`/shell), so `kubectl cp` won't work —
instead use `tctl auth sign --tar` to stream a tarball straight to stdout:

```bash
kubectl exec <my-teleport-auth-pod> -n teleport -- \
  tctl auth sign --format=db --host=postgres.postgres.svc.cluster.local --out=/tmp/postgres --ttl=2190h --tar \
  > postgres-certs.tar

mkdir -p postgres-certs && tar -xf postgres-certs.tar -C postgres-certs
# extracts to postgres-certs/tmp/postgres.{crt,key,cas}
```

Create the `postgres-tls` secret from them:

```bash
kubectl create secret generic postgres-tls -n postgres \
  --from-file=server.crt=postgres-certs/tmp/postgres.crt \
  --from-file=server.key=postgres-certs/tmp/postgres.key \
  --from-file=server.cas=postgres-certs/tmp/postgres.cas
```

### 3. Create the database credentials

```bash
kubectl create secret generic postgres-credentials -n postgres \
  --from-literal=password=<choose-a-password>
```

### 4. Install the chart

```bash
helm install my-postgres . --namespace postgres
```

Postgres is now reachable only from inside the cluster, at
`postgres.postgres.svc.cluster.local:5432`, and only with a client
certificate signed by Teleport's Database CA — i.e. only through Teleport.

Next, deploy `../my-teleport-db-agent` to register this database with
Teleport.
