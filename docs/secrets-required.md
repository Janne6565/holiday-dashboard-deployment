# Required secrets

These are created **manually**, out-of-band, once per cluster. Nothing here is stored in git.

## `backend-secrets` (namespace `holidash-prod`)

Consumed wholesale by the backend via `envFrom`. Create with:

```bash
kubectl -n holidash-prod create secret generic backend-secrets \
  --from-literal=SPRING_DATASOURCE_URL='jdbc:postgresql://postgres:5432/holidash' \
  --from-literal=SPRING_DATASOURCE_USERNAME='holidash' \
  --from-literal=SPRING_DATASOURCE_PASSWORD='<same as postgres password>' \
  --from-literal=FRONTEND_URL='https://holiday.jannekeipert.de' \
  --from-literal=INGEST_KEY='<strong random string; also given to the collectors>' \
  --from-literal=RIOT_API_KEY='<riot api key>' \
  --from-literal=RIOT_PLATFORM='euw1' \
  --from-literal=RIOT_REGION='europe' \
  --from-literal=RIOT_ID_JANNE='GameName#TAG' \
  --from-literal=RIOT_ID_SIMON='GameName#TAG'
```

Riot keys are optional — without them the LoL sync is skipped and the panel stays empty.

## `postgres-secrets` (namespace `holidash-prod`)

```bash
kubectl -n holidash-prod create secret generic postgres-secrets \
  --from-literal=POSTGRES_PASSWORD='<must match SPRING_DATASOURCE_PASSWORD>'
```

## `ghcr-pull-secret` (only if the ghcr packages are private)

```bash
kubectl -n holidash-prod create secret docker-registry ghcr-pull-secret \
  --docker-server=ghcr.io \
  --docker-username=<github-user> \
  --docker-password=<github-pat-with-read:packages>
```

## GitHub Actions secrets (set on both app repos)

| Secret | Purpose |
|---|---|
| `DEPLOY_REPO_TOKEN` | PAT with `contents: write` on `holiday-dashboard-deployment` — lets CI push the image-tag bump |
