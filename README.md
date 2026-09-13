My-PAM is a home-lab to test Teleport - a privileged access management solution.

The repository contains:
- `my-teleport`: an helm chart to deploy teleport in a cluster
- `my-postgres`: an helm chart to deploy a PostgreSQL instance as a target resource for Teleport Database Access
- `my-teleport-db-agent`: an helm chart to deploy the Teleport Database Service agent that proxies access to `my-postgres`
