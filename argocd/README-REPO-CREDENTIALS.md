# Argo CD repository credentials

Argo CD needs credentials to clone **private** Git repos. If you see:

`ComparisonError: authentication required`

add the repo in Argo CD with credentials (UI or the Secret below).

---

## Option A: Argo CD UI

1. Open Argo CD → **Settings** → **Repositories** → **Connect Repo**.
2. **Repository URL:** your repo (e.g. `https://github.com/kaunglin/devops-argocd.git` or `git@github.com:kaunglin/devops-argocd.git`).
3. **HTTPS:** set **Username** (e.g. `git` or your GitHub username) and **Password** = a [GitHub Personal Access Token](https://github.com/settings/tokens) (repo scope).
4. **SSH:** leave username/password empty and paste your **SSH private key** (contents of `~/.ssh/id_ed25519` or `~/.ssh/id_rsa`).
5. Save. The app should sync on the next refresh.

---

## Option B: kubectl (Secret)

Create a Secret in the `argocd` namespace. Argo CD will use it for the repo URL you configured in the Application.

### HTTPS (GitHub token)

Replace `YOUR_GITHUB_TOKEN` and the repo URL, then apply:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-devops-argocd
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  url: https://github.com/kaunglin/devops-argocd.git
  username: git
  password: YOUR_GITHUB_TOKEN
```

```bash
kubectl apply -f argocd/repo-credentials-secret.yaml
```

(Use a token with `repo` scope; create at https://github.com/settings/tokens .)

### SSH key

Replace the key and URL, then apply:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-devops-argocd
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  url: git@github.com:kaunglin/devops-argocd.git
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ... paste contents of ~/.ssh/id_ed25519 ...
    -----END OPENSSH PRIVATE KEY-----
```

```bash
kubectl apply -f argocd/repo-credentials-secret.yaml
```

---

After the Secret exists, refresh the app in Argo CD (or wait for auto-sync). The "authentication required" error should go away.
