# How to Deploy to Google Cloudrun

Google Cloudrun is a ... With Docker, it's simple to deploy your Deno app to
Google Cloudrun.

First, we'll show you how to deploy manually, then we'll show you how to
automate it with GitHub Actions.

Pre-requisites:

- [Google Cloud Platform account](https://cloud.google.com/gcp)
- [`docker` CLI](https://docs.docker.com/engine/reference/commandline/cli/)
  installed
- [`gcloud`](https://cloud.google.com/sdk/gcloud) installed

## Manual Deployment

### Set up Artifact Registry

Artifact Registry is GCP's private registry of Docker images.

Before we can use it, go to GCP's
[Artifacts](https://console.cloud.google.com/artifacts) and "Create repository".

### Build, Tag, and Push to Artifact Registry

First, we'll have to register the registry's address to `gcloud`:

```
gcloud auth configure-docker us-central1-docker.pkg.dev
```

Next, build your Docker image and give it a name.

```
```

Then, tag it with the new Google Artifact Registry address, repository, and
name. The image name should follow this structure:
`{{ location }}-docker.pkg.dev/{{ google_cloudrun_project_name }}/{{ repository }}/{{ image }}`:

```
docker tag deno-cloudrun us-central1-docker.pkg.dev/deno-app-368305/deno-repository/deno-cloudrun-image
```

If you don't specify a tag, it'll use `:latest` by default.

Next, push the image:

```
docker push us-central1-docker.pkg.dev/deno-app-368305/deno-repository/deno-cloudrun-image
```

_[More info on how to push and pull images to Google Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling)._

Your image is now in Google Artifact Registry!

![Image in Google Artifact Registry](/static/image-in-google-artifact-registry.png)

### Create a Google Cloud Run Service

We need an instance where we can build these images, so let's go to
[Google Cloud Run](https://console.cloud.google.com/run) and click "Create
Service".

Select "Deploy one revision from an existing container image". Use the drop down
to select the image from the Artifact Registry we just pushed.

Select "allow unauthenticated requests" and then click "Create service". Make
sure the port is `8000`.

When it's done, you should see it:

![Hello from Google Cloud Run](/static/hello-from-google-cloud-run.png)

Awesome!

### Deploy with `gcloud`

Now that it's created, we'll be able to deploy to this service from the `gcloud`
CLI.

The service_name is `deno-cloudrun-image`.

```
gcloud run deploy { service_name } --image=us-central1-docker.pkg.dev/deno-app-368305/deno-repository/deno-cloudrun-image --region=us-central1 --allow-unauthenticated
```

![]()

Success!

## Automate Deployment with GitHub Actions

In order for automation to work, we first need to make sure that these both have
been created:

- the Google Cloud Run instance
- the Google Artifact Registry

(If you haven't done that, please see the section before.)

Now that we have done that, we can automate it with a GitHub workflow. Here's
the yaml file:

```yml
name: Build and Deploy to Cloud Run

on:
  push:
    branches:
      - main

env:
  PROJECT_ID: {{ PROJECT_ID }}
  GAR_LOCATION: {{ GAR_LOCATION }}
  REPOSITORY: {{ GAR_REPOSITORY }}
  SERVICE: {{ SERVICE }}
  REGION: {{ REGION }}

jobs:
  deploy:
    name: Deploy
    permissions:
      contents: 'read'
      id-token: 'write'

    runs-on: ubuntu-latest
    steps:
    - name: CHeckout
      uses: actions/checkout@v3

    - name: Google Auth
      id: auth
      uses: 'google-github-actions/auth@v0'
      with:
        credentials_json: '${{ secrets.GCP_CREDENTIALS }}'

    - name: Login to GAR
      uses: docker/login-action@v2.1.0
      with:
        registry: ${{ env.GAR_LOCATION }}-docker.pkg.dev
        username: _json_key
        password: ${{ secrets.GCP_CREDENTIALS }}

    - name: Build and Push Container
      run: |-
        docker build -t "${{ env.GAR_LOCATION }}-docker.pkg.dev/${{ env.PROJECT_ID }}/${{ env.REPOSITORY }}/${{ env.SERVICE }}:${{ github.sha }}" ./
        docker push "${{ env.GAR_LOCATION }}-docker.pkg.dev/${{ env.PROJECT_ID }}/${{ env.REPOSITORY }}/${{ env.SERVICE }}:${{ github.sha }}"

    - name: Deploy to Cloud Run
      id: deploy
      uses: google-github-actions/deploy-cloudrun@v0
      with:
        service: ${{ env.SERVICE }}
        region: ${{ env.REGION }}
        image: ${{ env.GAR_LOCATION }}-docker.pkg.dev/${{ env.PROJECT_ID }}/${{ env.REPOSITORY }}/${{ env.SERVICE }}:${{ github.sha }}

    - name: Show Output
      run: echo ${{ steps.deploy.outputs.url }}
```

The environment variables that we need to set are (the examples in parenthesis
are the ones for this repository)

- `PROJECT_ID`: your project id (`deno-app-368305`)
- `GAR_LOCATION`: the location your Google Artifact Registry is set
  (`us-central1`)
- `GAR_REPOSITORY`: the name you gave your Google Artifact Registry
  (`deno-repository`)
- `SERVICE`: the name of the Google Cloud Run service (`deno-cloudrun-image`)
- `REGION`: the region of your Google Cloud Run service (`us-central1`)

The secret variables that we need to set are:

- `GCP_CREDENTIALS`: this is the
  [service account](https://cloud.google.com/iam/docs/service-accounts) json
  key. When you create the service account, be sure to
  [include the roles and permissions necessary](https://cloud.google.com/iam/docs/granting-changing-revoking-access#granting_access_to_a_user_for_a_service_account)
  for Artifact Registry and Google Cloud Run.

[Check out more details and examples of deploying to Cloud Run from GitHub Actions.](https://github.com/google-github-actions/deploy-cloudrun)

For reference:
https://github.com/google-github-actions/example-workflows/blob/main/workflows/deploy-cloudrun/cloudrun-docker.yml
