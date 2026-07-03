# raapid-azure-marketplace-ocr-manifest

GitOps source of truth for the raapid OCR managed application. Flux v2
(installed as an AKS cluster extension by [raapid-ocr-marketplace-repo](https://github.com/raapidorg/raapid-ocr-marketplace-repo)'s
`modules/gitops.bicep`) watches the `main` branch of this repo and
reconciles everything under `clusters/ocr-managed-app/` onto every
customer's AKS cluster automatically — no manual `helm`/`kubectl` step,
during deployment or after.

## Layout

```
clusters/ocr-managed-app/
  kustomization.yaml   # lists the manifests Flux applies
  kafka-source.yaml    # HelmRepository: the bitnami chart repo
  kafka-release.yaml   # HelmRelease: installs Kafka (PoC service)
```

## Adding a service

Each OCR microservice becomes one `HelmRelease` (pointing at a
`HelmRepository` or `OCIRepository` source) added to this folder and
listed in `kustomization.yaml`. Once merged to `main`, Flux picks it up
on its next sync (every 5 minutes, per `syncIntervalInSeconds` in the
Bicep module) on every customer cluster — no redeploy of the managed
app needed.

## Why this repo is public

Flux needs a credential to clone a private repo, and there's currently
no way to hand that credential into a Marketplace managed-app template
without it being visible to whoever deploys the template. So for now
this repo carries no secrets — real secrets (DB passwords, service
connection strings) are handled via Key Vault + AKS Workload Identity
in the main infra template, never in Helm values here. If/when we need
a private repo, the plan is a publisher-side webhook that configures
Flux using raapid's own Key Vault, rather than embedding a credential
in the ARM template.
