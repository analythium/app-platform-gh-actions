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

- the app ID ($APP_ID) that you will know once the app is initially created,
- an API token for doctl to work ($DIGITALOCEAN_ACCESS_TOKEN) (make sure write is in the scope),
- and the access token to your Docker registry ($DOCKERHUB_ACCESS_TOKEN) (make sure write is in the scope).

### Add the workflow

Then follow these steps following the doctl action as one of the steps:

1. build the Docker image with docker build -t $TAG .,
2. log in to a registry using  echo $REGISTRY_TOKEN | docker login --username $USERNAME --password-stdin $REGISTRY_NAME where $REGISTRY_NAME can be omitted if you are using Docker Hub ($USERNAME and $REGISTRY_NAME can be added as secrets or as environment variables)
3. push the image to the registry with docker push $TAG,
4. update the app using doctl apps update $APP_ID (your access token will be known from the doctl action step).

This way, you can add any tests before the build/push/update steps to prevent a new release when some tests fail.
