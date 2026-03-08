# devops-argocd

Git repo for Argo CD applications targeting the homelab Kubernetes cluster (kind + Argo CD from [devops-k8s-homelab](https://github.com/YOUR_ORG/devops-k8s-homelab)).

## Repo layout

- **`homelab-apps/sample-app/`** – Sample app deployed by Argo CD (Namespace, Deployment, Service, Ingress).
- **`argocd/`** – Argo CD `Application` manifests that point at this repo and the app paths.

## sample-app

A minimal app (nginx) in the `sample-app` namespace, exposed via Ingress at **http://sample-app.local**.

### Manifests

| File            | Description                    |
|-----------------|--------------------------------|
| `namespace.yaml`| Creates namespace `sample-app`  |
| `deployment.yaml` | 1 replica nginx:1.25         |
| `service.yaml`  | ClusterIP Service on port 80   |
| `ingress.yaml`  | Host `sample-app.local`, path `/` |

### Deploy with Argo CD

1. **Prerequisites**
   - Homelab cluster running (from devops-k8s-homelab).
   - Argo CD installed and reachable (e.g. http://argocd.local).
   - This repo pushed to a Git server (GitHub, GitLab, etc.) that the cluster can reach.

2. **Configure the Application**
   - Edit `argocd/application-sample-app.yaml`.
   - Set `spec.source.repoURL` to your repo URL, e.g.:
     - `https://github.com/your-org/devops-argocd.git`
     - or `git@github.com:your-org/devops-argocd.git` (if Argo CD has SSH access).

3. **Register the app in Argo CD**
   ```bash
   kubectl apply -f argocd/application-sample-app.yaml
   ```
   Or apply from inside the repo:
   ```bash
   cd /path/to/devops-argocd
   kubectl apply -f argocd/application-sample-app.yaml
   ```

4. **Sync**
   - Argo CD will create the app and sync from `homelab-apps/sample-app`.
   - With the default `syncPolicy`, it will auto-sync and create the `sample-app` namespace and all resources.

5. **Access**
   - Add to `/etc/hosts`: `127.0.0.1  sample-app.local`
   - Open http://sample-app.local (nginx welcome page).

### Deploy without Argo CD (manual)

```bash
kubectl apply -f homelab-apps/sample-app/
```

## Adding more apps

1. Add a directory under `homelab-apps/`, e.g. `homelab-apps/my-app/`, with your manifests.
2. Add an `Application` in `argocd/`, e.g. `argocd/application-my-app.yaml`, with `spec.source.path` set to `homelab-apps/my-app` and `spec.destination.namespace` set to the app’s namespace.
3. Apply the Application: `kubectl apply -f argocd/application-my-app.yaml`.
