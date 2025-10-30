# Infrastructure README - infra-repo

This README supplements the [main project](https://github.com/saurav-22/food-delivery-app-repo) documentation with focused notes for the infrastructure Helm chart and the exact configuration/secret keys used by the deployments.

## Location

- Helm chart: `helm-chart/` (templates, `values.yaml`)
- App config template: `helm-chart/templates/app-config.yaml` (used to populate ConfigMap / env)
- Secrets file reference (example): `app-secret.yaml`

## Important: Secrets & Config keys

The repository includes two different Kubernetes artifacts that supply environment variables to pods. Please ensure these keys are populated before deploying.

1) `app-secret.yaml` (Kubernetes Secret)

- Contains only the database credentials below (Base64-encoded when applied as a Secret):
  - `DB_USER`
  - `DB_PASS`

2) `app-config.yaml` (ConfigMap / templated values)

- Contains the remaining runtime configuration values (example keys expected by services):
  - `DB_NAME`
  - `DB_HOST`
  - `DB_PORT`
  - `DB_SSLMODE`

- Service-to-service URLs (used by frontends/backends to call other microservices):
  - `CART_SERVICE_URL`
  - `ORDER_SERVICE_URL`
  - `MENU_SERVICE_URL`

Notes:
- The Helm chart consumes these keys either via `envFrom: secretRef` for `app-secret.yaml` and via a ConfigMap for `app-config.yaml` (or `env` entries populated from values).
- For production use, replace the static `app-secret.yaml` with a secret manager (AWS Secrets Manager, SSM, or External Secrets Operator) to avoid storing secrets in Git.

## Quick deploy notes (high-level)

1. Ensure the database credentials (`DB_USER`, `DB_PASS`) are created as a Kubernetes Secret in the target namespace before Helm installs the deployments.

2. Populate `DB_NAME`, `DB_HOST`, `DB_PORT`, `DB_SSLMODE`, and the service URLs in `values.yaml` or via a ConfigMap so the chart templates render the correct values.

3. Example Helm install (adjust `values.yaml` or pass `--set` overrides):

```powershell
helm upgrade --install food-delivery ./helm-chart -n food-delivery --create-namespace --values ./helm-chart/values.yaml
```

4. Verify: check pods and logs, and confirm environment variables are set inside running containers.

```powershell
kubectl get pods -n food-delivery
kubectl describe secret app-secret -n food-delivery
kubectl describe configmap app-config -n food-delivery
kubectl logs deploy/restaurant-service -n food-delivery
```

## Recommended improvements

- Move DB credentials to AWS Secrets Manager and use External Secrets Operator to populate Kubernetes Secrets at runtime.
- Add a small `values.example.yaml` with placeholders for all required keys (image repositories, tags, DB host, and the service URLs) so new deployers can quickly bootstrap.

If you want, I can create `values.example.yaml` and a small `scripts/create-secrets.ps1` to automate secret creation in the cluster. Which would you prefer next?