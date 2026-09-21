# shop

Three-tier application: React/nginx frontend, Python backend, Postgres with
pgvector. Exposed at **http://legendbits.shop.local**.

These manifests were **extracted from the live homelab cluster on 2026-09-21**.
The app had been applied ad-hoc with `kubectl` and existed nowhere but etcd,
so it could not be rebuilt if the namespace were deleted.

## Files

| File | Contents |
|------|----------|
| `namespace.yaml` | Namespace `shop` |
| `frontend.yaml` | Deployment + Service (`frontend:80`) |
| `backend.yaml` | Deployment + Service (`backend:8000`) |
| `postgres.yaml` | PVC `db-pvc` (2Gi) + Deployment + Service (`postgres:5432`) |
| `ingress.yaml` | `legendbits.shop.local` → `/api` to backend, `/` to frontend |
| `secrets.example.yaml` | **Template only.** Real Secrets are not in Git. |

## Before first sync

The two Secrets must exist, or the backend and postgres pods will not start:

```bash
kubectl get secret -n shop db-secret backend-secret
```

If missing, create them from `secrets.example.yaml` — it lists every required
key. `backend-secret` carries an OpenAI API key, a JWT signing secret, and a
`DATABASE_URL` containing the Postgres password, which is why it stays out of
the repo. The Argo CD Application excludes `secrets.example.yaml` from sync so
the placeholder values can never overwrite the real ones.

## Deploy

With Argo CD:

```bash
kubectl apply -f ../../argocd/application-shop.yaml
```

Without Argo CD:

```bash
kubectl apply -f . --prune=false   # secrets.example.yaml must be removed first
```

Add `127.0.0.1  legendbits.shop.local` to `/etc/hosts`.

## Known gaps

- **`:latest` image tags.** Both app images use `:latest`, so Argo CD cannot
  detect drift or roll back meaningfully. The digests running at extraction
  time are recorded in each file's header. Once Jenkins publishes immutable
  tags, pin them here and let CI bump the tag.
- **No resource requests or limits.** Every container runs unbounded, which on
  a memory-capped VM means one runaway pod can starve the cluster.
- **Postgres storage is node-bound.** `db-pvc` uses the `standard`
  (local-path) StorageClass with a `Delete` reclaim policy: the data lives on
  one node's disk and is destroyed with the PVC. Back up before any cluster
  rebuild or worker-count change.
