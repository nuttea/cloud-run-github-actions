# cloud-run-github-actions
cicd pipeline with github action to deploy cloud run

https://github.com/marketplace/actions/authenticate-to-google-cloud

https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions

## Environment Variables and gcloud setup

Update gcloud components

```bash
gcloud components update
gcloud components install --quiet \
    alpha \
    beta \
    log-streaming \
    cloud-run-proxy \
    skaffold

gcloud auth login
gcloud auth application-default login
```

Manual export env vars

```bash
export CICD_PROJECT_ID=nuttee-lab-tf
export CICD_PROJECT_NUMBER=$(gcloud projects describe $CICD_PROJECT_ID --format="value(projectNumber)")
export SERVICE_PROJECT_ID=nuttee-lab-02
export SERVICE_PROJECT_NUMBER=$(gcloud projects describe $SERVICE_PROJECT_ID --format="value(projectNumber)")
export WORKLOAD_IDENTITY_POOL_NAME=gh-actions-pool
export WORKLOAD_IDENTITY_PROVIDER_NAME=gh-provider
export GH_ACTIONS_SA=nuttea-gh-actions
export GH_ORG=nuttea
export GH_REPO=cloud-run-github-actions

gcloud config set project $CICD_PROJECT_ID
```

or copy and edit from .env.example

```bash
cp .env.example .env
vim .env

source .env

gcloud config set project $CICD_PROJECT_ID
```

## Setup CICD Project

Create a service account

Setting up Identity Federation for GitHub Actions

```bash
gcloud iam workload-identity-pools create "${WORKLOAD_IDENTITY_POOL_NAME}" \
  --project="${CICD_PROJECT_ID}" \
  --location="global" \
  --display-name="Github Actions Identity pool"

gcloud iam workload-identity-pools providers create-oidc "${WORKLOAD_IDENTITY_PROVIDER_NAME}" \
  --project="${CICD_PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="${WORKLOAD_IDENTITY_POOL_NAME}" \
  --display-name="Github Actions identity provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.aud=assertion.aud,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```

Get the full ID of the Workload Identity Pool ID and Provider ID

```bash
WORKLOAD_IDENTITY_POOL_ID=$(gcloud iam workload-identity-pools describe "${WORKLOAD_IDENTITY_POOL_NAME}" --project="${CICD_PROJECT_ID}" --location="global" --format="value(name)")

WORKLOAD_IDENTITY_PROVIDER_ID=$(gcloud iam workload-identity-pools providers describe "${WORKLOAD_IDENTITY_PROVIDER_NAME}" --workload-identity-pool "${WORKLOAD_IDENTITY_POOL_NAME}"  --project="${CICD_PROJECT_ID}" --location="global" --format="value(name)")
```

```bash
gcloud iam service-accounts add-iam-policy-binding "${GH_ACTIONS_SA}@${CICD_PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${CICD_PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${GH_ORG}/${GH_REPO}"
```

## Setup Workload Project
PROJECT_ID=nuttee-lab-02

Add permission for Github Actions Service Account

```bash

```
