# Teleport Cluster — Local Deployment (v18)

A custom Helm chart to quickly deploy a local **Teleport** instance on a local Kubernetes cluster (e.g. `k3d`).

It runs Teleport **v18** in `standalone` mode (local persistent storage) with the native Kubernetes operator for access management.

## Deployment

### 0. Prerequisite 
Create a Kubernetes cluster. To test locally you can use kind or k3d  

```bash
```
```bash
kind cluster create my-teleport
```


### 1. Update chart dependencies

Download the official `teleport-cluster` subchart declared in `Chart.yaml`:

```bash
helm dependency update .
```

### 2. Install the chart

Deploy into a dedicated namespace:

```bash
helm install my-teleport . --namespace teleport --create-namespace
```

### 3. Create the admin user

Once the pods are running, find the auth service pod:

```bash
kubectl get pods -n teleport
```

Run `tctl` inside the auth pod to create your account and get a one-time invite link:

```bash
kubectl exec -it <my-teleport-auth-pod> -n teleport -- \
  tctl users add admin --roles=access,editor --logins=root
```

## Web UI Access

Because the cluster runs in an isolated `k3d` environment, open a tunnel to reach the proxy.

### 1. Port-forward the proxy

Forward the TLS port of the proxy service:

```bash
kubectl port-forward -n teleport svc/my-teleport-proxy 3080:443
```

### 2. First-time setup

1. Open your invite link, replacing `teleport.local:443` with the port-forward address (e.g. `localhost:3080`).
2. Follow the prompts to set your password and scan the **MFA QR code** (Google Authenticator, Bitwarden, etc.).

You are now connected to your local Teleport instance.
