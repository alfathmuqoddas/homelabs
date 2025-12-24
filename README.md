# Homelabs

A collection of self-hosted services and infrastructure configurations for a personal home lab environment, primarily using Docker Compose and K3s.

## Project Structure

The repository is organized into several directories, each containing the configuration for a specific service or infrastructure component:

- **`gitea/`**: Self-hosted Git service (Gitea).
- **`jenkins/`**: Automation server (Jenkins) with a Docker-enabled agent.
- **`k3s/`**: Kubernetes configuration, including ArgoCD ingress and local Kubeconfig.
- **`portainer-ce/`**: Docker and Kubernetes management UI.
- **`postgres/`**: Shared PostgreSQL database instance.
- **`registry/`**: Local Docker registry.
- **`traefik/`**: Reverse proxy and edge router with Docker integration.

## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) (for K3s/Kubernetes interactions)

### Network Setup

Many services in this lab use an external Docker network named `proxy` to communicate and for Traefik to route traffic. Create this network before starting the services:

```bash
docker network create proxy
```

### Deploying Services

To deploy a service, navigate to its directory and use `docker-compose`:

```bash
cd <service-directory>
docker-compose up -d
```

For example, to start Traefik:

```bash
cd traefik
docker-compose up -d
```

## Service Details

### Traefik
- **Role**: Edge router and reverse proxy.
- **Dashboard**: Enabled by default (requires configuration for external access).
- **Entrypoints**: Web (Port 80).

### Gitea
- **URL**: `http://gitea.local:3000`
- **SSH Port**: 2222
- **Network**: Connected to the `proxy` network.

### Jenkins
- **URL**: `http://localhost:8080`
- **Components**:
  - **Controller**: Jenkins LTS.
  - **Docker Agent**: A custom agent built to run Docker commands (via `/var/run/docker.sock`).
- **Setup Notes**:
  - Update `min-api-version` to `1.43` in `/etc/docker/daemon.json` if required.
  - Check the Docker group GID using `grep docker /etc/group` and ensure it matches the `JENKINS_AGENT_GID` in `docker-compose.yaml`.
  - Fix permissions on `./jenkins_home` if needed: `sudo chown -R 1000:1000 ./jenkins_home`.

### Portainer
- **URL**: `https://localhost:9443`
- **Role**: Visual management for Docker containers and images.

### PostgreSQL
- **Host**: `shared_postgres` (internal network) or `localhost:5432`
- **Credentials**: Managed via environment variables in `postgres/docker-compose.yaml`.

### Local Registry
- **URL**: `localhost:5000`
- **Role**: Stores locally built Docker images.

### K3s / Kubernetes
- **Config**: A `config` file is provided in `k3s/` for local `kubectl` access (configured for `https://127.0.0.1:6443`).
- **ArgoCD**: An ingress definition for ArgoCD is provided in `k3s/argocd-ingress.yaml`, using Traefik as the ingress class.

## CI/CD Pipeline

A sample `Jenkinsfile` is provided in `jenkins/jenkins-pipeline/` which demonstrates a complete CI/CD flow:

1.  **Agent**: Runs on the `docker-agent`.
2.  **Checkout**: Clones the repository from the local Gitea instance.
3.  **Build**: Builds a Docker image using Docker BuildKit.
4.  **Push**: Pushes the resulting image to the local Docker registry.

**Required Jenkins Credentials**:
- `gitea-creds`: Username/password for Gitea access.

**Expected Environment Variables**:
- `REPO_NAME`: Name of the repository.
- `COMMIT_ID`: ID of the commit being built.

## Troubleshooting

- **Permissions**: If Jenkins or other services fail to start due to permission issues on mounted volumes, ensure the host directories have the correct ownership (usually UID `1000`).
- **Docker in Docker**: The Jenkins agent requires access to `/var/run/docker.sock`. Ensure the user running the container has permissions to access this socket.
