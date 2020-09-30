# dotNet Core on Kubernetes

Application ASP.NET Core déployé en local dans un cluster Kubernetes à node unique


# 3. Deploiement par Helm

Se mettre dans le dossier contenant le chart metadata, pour deployer:

```bash
helm upgrade --install [releaseName] . \
  --set api.image.tag="nightly" \
  --set site.image.tag="firsttry" \
  --debug \
````

Pour annuler un deployment:

```bash
helm rollback [releaseName]
````

Pour lister les releases:

```bash
helm list
````