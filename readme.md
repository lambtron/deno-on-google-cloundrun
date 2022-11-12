




Use Artifact Registry

https://console.cloud.google.com/artifacts

Go through the options and Click "Create repository"

https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling

Configure `gcloud` with `gcloud auth`:

```
gcloud auth configure-docker us-central1-docker.pkg.dev
```

```
docker tag deno-cloudrun us-central1-docker.pkg.dev/deno-app-368305/deno-repository/deno-cloudrun-image
```

```
docker push us-central1-docker.pkg.dev/deno-app-368305/deno-repository/deno-cloudrun-image
```

Next, go to [Google Cloud Run](https://console.cloud.google.com/run) and click "Create Service".

Select "Deploy one revision from an existing container image". Use the drop down to select the image from the Artifact Registry we just pushed.

Select "allow unauthenticated requests" and then click "Create service". Make sure the port is `8000`.

When it's done, you should see it:

![]()

Awesome!

Now that it's created, we'll be able to deploy to this service from the `gcloud` CLI.

The service_name is `deno-cloudrun-image`.

```
gcloud run deploy { service_name } --image=us-central1-docker.pkg.dev/deno-app-368305/deno-repository/deno-cloudrun-image --region=us-central1 --allow-unauthenticated
```

## Automate Deployment with GitHub Actions

Now that we have done that, let's automate it.

```yml
```

WIF
https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions

Created workload identity pool [my-pool].

Created workload identity pool provider [my-provider].

create a new IAM service account on your google project.

```
gcloud iam service-accounts add-iam-policy-binding "97359446200-compute@developer.gserviceaccount.com" \
  --project="deno-app-368305" \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:97359446200-compute@developer.gserviceaccount.com"
```

https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/add-iam-policy-binding



For reference:
https://github.com/google-github-actions/example-workflows/blob/main/workflows/deploy-cloudrun/cloudrun-docker.yml

