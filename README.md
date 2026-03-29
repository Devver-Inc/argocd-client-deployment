# argocd-client-deployment

Ce dépôt contient les **values Helm** déposées par les clients et consommées par un **ArgoCD ApplicationSet** pour déployer automatiquement des applications sur Kubernetes via le chart `kumernetes.mel-chart-template`.

---

## Fonctionnement

```
Client dépose values.yaml
         │
         ▼
argocd-client-deployment/
  └── <organisation>/
        └── <projet>/
              └── values.yaml
                     │
                     ▼
      ArgoCD ApplicationSet (Git generator)
                     │
                     ▼
      Helm chart: kumernetes.mel-chart-template
                     │
                     ▼
      Application déployée sur Kubernetes
```

1. Le client crée un fichier `values.yaml` dans le chemin `<organisation>/<projet>/values.yaml`.
2. L'**ApplicationSet ArgoCD** surveille ce dépôt via un **Git file generator** et détecte chaque nouveau fichier `values.yaml`.
3. Pour chaque fichier détecté, ArgoCD crée automatiquement une **Application ArgoCD** qui déploie le chart Helm `kumernetes.mel-chart-template` en injectant les values du fichier correspondant.
4. L'application est déployée dans un namespace dédié sur le cluster Kubernetes.

---

## Structure du dépôt

```
argocd-client-deployment/
├── values.yaml                          # Values par défaut (référence)
├── <organisation>/
│   └── <projet>/
│       └── values.yaml                  # Values spécifiques au projet
└── ...
```

Chaque dossier `<organisation>/<projet>/` correspond à un déploiement indépendant.

---

## Format du fichier `values.yaml`

```yaml
# Identifiants
organization:
  name: "mon-org"
  domain: "devver.app"

project:
  name: "mon-projet"

# Image du conteneur
container:
  image: "ghcr.io/devver-inc/deploy-agent:latest"
  port: 80
  type: "app"
  imagePullSecrets:
    - name: ghcr-secret

  # Variables d'environnement (optionnel)
  env:
    - name: NODE_ENV
      value: "production"
    - name: DEVVER_SECRET
      value: "mon-secret"

# Ressources Kubernetes
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Stockage persistant
persistence:
  enabled: true
  app:
    size: "10Gi"
    mountPath: "/app"
    storageClass: longhorn
  root:
    size: "5Gi"
    mountPath: "/root"
    storageClass: longhorn

# Nombre de replicas
replicaCount: 1

# Ports
ports:
  http: 80
  https: 443
```

---

## Ajouter un nouveau déploiement

1. Créer le dossier `<organisation>/<projet>/`
2. Y déposer un fichier `values.yaml` en suivant le format ci-dessus
3. Committer et pousser sur la branche principale
4. ArgoCD détecte automatiquement le nouveau fichier et crée l'Application correspondante
