# app-platform-gh-actions

> Integrate DigitalOcean App Platform with GitHub Actions in a true CICD fashion

https://docs.digitalocean.com/reference/doctl/reference/auth/init/

### Create an app

Build the image locally, test, then and push to Docker Hub:

```bash
## build the image
export TAG="analythium/gh-action-test:latest"
docker build -t $TAG .

## run and test
docker run -p 8080:8080 $TAG

## push the image
docker push $TAG
```

[Install `doctl`](https://docs.digitalocean.com/reference/doctl/how-to/install/). On 1st time use of `doctl` you need to validate with `doctl auth init`. Validate the `app.yaml` then create the app with `doctl`:

```bash
doctl apps spec validate app.yaml

doctl apps create --spec app.yaml
```

Take note of the app ID at the end: `baf22385-feed-4cb9-99aa-db6b15a0c75e` or something similar.

### Add secrets

Add a couple of secrets to the GitHub repo:

- `APP_ID`: the app ID from the initial deployment to the App Platform
- `DIGITALOCEAN_ACCESS_TOKEN`: an API token for `doctl` (make sure write is in the scope)
- `DOCKERHUB_ACCESS_TOKEN`: access token for Docker Hub (make sure write is in the scope)
- `DOCKERHUB_USERNAME`: your Docker Hub user name used for registry login
### Add the workflow

see the `.github/workflows/deploy.yml` file for the steps.
