# ⚙️ Infrastructure Kubernetes / DevSecOps

> Provision, déploiement et sécurisation de la plateforme Conforméo  
> (cert-manager, Kong Gateway, Keycloak, Istio, External-Secrets, …)

---

## 🌳  Arborescence

infra/
├── cluster/ # Manifests K8s gérés par Kustomize
│ ├── base/ # Définitions génériques (cert-manager, istio…)
│ └── overlays/ # Spécifique aux env. dev / prod
├── terraform/ # Provision du cluster (EKS / GKE / AKS)
├── .github/workflows/ # Pipelines CI/CD GitHub Actions
└── helmfile.yaml # (option) orchestration Helm


| Dossier | Rôle rapide |
|---------|-------------|
| `cluster/base/` | Charts ou manifests Helm/Kustomize de chaque stack (cert-manager, kong, keycloak, istio, external-secrets, …) |
| `cluster/overlays/{dev,prod}/` | Paramètres env-spécifiques (noms de domaines, répliques, tags d’image) |
| `terraform/{dev,prod}/` | Code Terraform pour provisionner le cluster, le bucket state, VPC, etc. |

---

## 🚀  Déploiement (workflow GitHub Actions)

1. **Push sur `main`** dans `cluster/**` ➜ workflow `deploy-prod.yml`.
2. Le job se charge de :
   1. Récupérer le kubeconfig (secret GitHub `KUBE_CONFIG_PROD`).
   2. `kustomize build cluster/overlays/prod | kubectl apply -f -`.
   3. Publier le diff sur Slack (optionnel).

Pour l’environnement *dev*, un workflow similaire (`deploy-dev.yml`) utilise `cluster/overlays/dev` et un secret `KUBE_CONFIG_DEV`.

---

## 🛠️  Requirements Dev Local

| Outil | Version mini | Notes |
|-------|--------------|-------|
| kubectl | 1.30 | `brew install kubectl` |
| kustomize | 5.x | `brew install kustomize` |
| helm | 3.14 | `brew install helm` |
| terraform | 1.8 | `brew tap hashicorp/tap && brew install hashicorp/tap/terraform` |

---

## 🧩  Ajouter une nouvelle stack

```bash
# 1. Créer le dossier base
mkdir -p cluster/base/argo-cd
# 2. Ajouter un Chart via Kustomize
cat <<EOF > cluster/base/argo-cd/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

helmCharts:
  - name: argo-cd
    repo: https://argoproj.github.io/argo-helm
    version: 6.9.5
    namespace: argocd
EOF
# 3. Référencez-la dans cluster/overlays/{dev,prod}/kustomization.yaml
