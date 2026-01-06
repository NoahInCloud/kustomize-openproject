# Kustomize OpenProject PoC

This repo shows how to manage environment differences (test vs prod) for OpenProject, Postgres, and Memcached using **pre-rendered Helm output** plus Kustomize overlays. It is structured to mirror the companion Tanka/Jsonnet PoC for easy A/B comparison.

## Layout

```
openproject-project/
  memcached/
    base/helm-rendered/memcached.yaml
    overlays/{test,prod}/kustomization.yaml
  openproject/
    base/helm-rendered/openproject.yaml
    overlays/{test,prod}/kustomization.yaml
  postgres/
    base/helm-rendered/{postgres.yaml,postgres-backup-*.yaml}
    overlays/{test,prod}/kustomization.yaml
```

Base directories hold pre-rendered Helm YAML. Overlays set the namespace, per-environment replica counts, and Vault Agent Injector annotations for POD_INJECTION.

## ArgoCD app mapping (Kustomize)

| App | Path | Revision | Namespace |
| --- | --- | --- | --- |
| lhw-openproject-memcached-test | `openproject-project/memcached/overlays/test` | `Test` | `lhw-openproject-test` |
| lhw-openproject-memcached-prod | `openproject-project/memcached/overlays/prod` | `Production` | `lhw-openproject-prod` |
| lhw-openproject-openproject-test | `openproject-project/openproject/overlays/test` | `Test` | `lhw-openproject-test` |
| lhw-openproject-openproject-prod | `openproject-project/openproject/overlays/prod` | `Production` | `lhw-openproject-prod` |
| lhw-openproject-postgres-test | `openproject-project/postgres/overlays/test` | `Test` | `lhw-openproject-test` |
| lhw-openproject-postgres-prod | `openproject-project/postgres/overlays/prod` | `Production` | `lhw-openproject-prod` |

## How to build locally

```bash
# Test overlay
cd openproject-project/memcached/overlays/test
kustomize build

# Prod overlay
cd ../prod
kustomize build
```

Run the same pattern for `openproject` and `postgres`.

## Vault Agent Injector

Overlays add `vault.hashicorp.com/*` annotations so the Vault Agent sidecar renders secrets to `/vault/secrets/db` at runtime. Update the `vaultRole` and `vaultSecretPath` values in patches if your Vault config differs.

## Branches

- `main`: source of truth for the PoC
- `Test`: pinned to test settings for ArgoCD
- `Production`: pinned to production settings for ArgoCD
