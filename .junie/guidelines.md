### Project Guidelines

#### Tech Stack
- **Infrastructure as Code (IaC):** Kustomize
- **Target Platform:** Kubernetes
- **Applications:** n8n, PostgreSQL 11

#### Repository Structure
- `base/`: Contains the base Kubernetes manifests for all components.
- `production/`: Overlay for the production environment.

#### Deployment Conventions
- All resources are deployed in the `n8n` namespace (defined in `base/kustomization.yaml`).
- **PostgreSQL:**
    - Uses `postgres:11` image.
    - Persistence is handled by a PVC named `n8n-db-data`.
    - Credentials and database configuration are managed via `secret_n8n-db-secrets.yaml`.
- **n8n:**
    - Uses `n8nio/n8n` image.
    - Persistence is handled by a PVC named `n8n-app-data`.
    - Includes an init container `volume-permissions` to ensure the `/data` directory has the correct ownership (1000:1000).
    - Database connection is configured via environment variables, pointing to the `postgres` service.
    - Startup includes a `sleep 5` delay to allow the database to become ready.

#### Configuration Management
- Secrets should be managed securely. The current `base/secret_n8n-db-secrets.yaml` contains default values with descriptive keys. Ensure these are properly managed for production environments.
- Resource limits and requests are defined in the deployment manifests. Monitor these and adjust based on actual usage.

#### Development Workflow
- When adding new resources, add them to `base/kustomization.yaml`.
- For environment-specific changes, create or update overlays in the respective environment directory (e.g., `production/`).

#### Secrets
To create a sealedsecret from a secret:
`kubeseal -f secret_[name].yaml -w sealedsecret_[name].yaml -n n8n`
secrets and configs are stored in the production folder (or rather once per enviroment)