# feathersforall-gitops

Estado deseado de **FeathersForAll** en Kubernetes. **Argo CD** lo lee desde el clúster y lo aplica: lo que está en `main` es lo que corre.

El código de la app, el pipeline y la guía completa están en [Erolayer-Cl/FeathersForAll](https://github.com/Erolayer-Cl/FeathersForAll):
- Guía: `FeathersForAll/Documentos/infra/ci-cd.md`
- Decisión: `FeathersForAll/Documentos/adr/0009-ci-cd-y-gitops.md`

```text
base/                 manifiestos comunes (Deployments, Services, Ingress, ConfigMap, HPA, PDB, CronJob)
overlays/dev/         clúster kind del laboratorio: HPA hasta 3 réplicas; digest escritos por CI
overlays/prod/        sin banderas de desarrollo; digest copiados desde dev con aprobación
argocd/project.yaml   AppProject: solo este repo, solo el namespace feathersforall, nunca Secrets
argocd/application-dev.yaml    sync automático (prune + selfHeal)
argocd/application-prod.yaml   sync manual (todavía no hay clúster de producción)
```

## Reglas

- **No se edita a mano** el digest de `overlays/dev`: lo escribe el job `promote` del CI en un PR `deploy(dev): ...`.
- **Prod** solo recibe lo que ya corre en dev: workflow `promote-prod` del repo app (requiere aprobación del Environment `production`) → PR con 2 revisores.
- **Imágenes siempre por digest** (`@sha256:...`), nunca por tag.
- **Sin Secrets en este repo.** `ffa-api-secrets` y `ffa-jobs-secrets` los crea Terraform (`terraform/` del repo app); el AppProject además prohíbe que Argo CD gestione Secrets.
- **Rollback = `git revert`** del PR de despliegue. Nadie corre `kubectl apply` contra el clúster.

## Probar en local

```bash
kubectl kustomize overlays/dev      # o: kustomize build overlays/dev
kubectl kustomize overlays/prod
```
