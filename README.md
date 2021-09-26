# app-platform-gh-actions

> Integrate DigitalOcean App Platform with GitHub Actions in a true CICD fashion

https://docs.digitalocean.com/reference/doctl/reference/auth/init/

### Create an app

Build the image locally, test, then and push to Docker Hub:

```bash
## export few variables
export REGISTRY="analythium"
export IMAGE_NAME="github-action-test"
export GITHUB_SHA="latest"

## build the image
docker build -t $REGISTRY/$IMAGE_NAME:$GITHUB_SHA .

## run and test
docker run -p 8080:8080 $REGISTRY/$IMAGE_NAME:$GITHUB_SHA

## push the image
docker push $REGISTRY/$IMAGE_NAME:$GITHUB_SHA
```

[Install `doctl`](https://docs.digitalocean.com/reference/doctl/how-to/install/). On 1st time use of `doctl` you need to validate with `doctl auth init`. 

Now use text `app.yaml` as a template for our app spec and substitute the environmental variables in it to create a new file called `update.yaml`. We set the image tag to the git SHA in GitHub actions, but for now just tag with latest:

```bash
envsubst < app.yaml > update.yaml
```

Validate the `update.yaml` then create the app with `doctl`:

```bash
doctl apps spec validate update.yaml

doctl apps create --spec update.yaml
```

Take note of the app ID at the end: `ff95990f-5235-43dd-baba-74980e30740f` or something similar.

### Add secrets

Add a couple of secrets to the GitHub repo:

- `APP_ID`: the app ID from the initial deployment to the App Platform
- `DIGITALOCEAN_ACCESS_TOKEN`: an API token for `doctl` (make sure write is in the scope)
- `DOCKERHUB_ACCESS_TOKEN`: access token for Docker Hub (make sure write is in the scope)
- `DOCKERHUB_USERNAME`: your Docker Hub user name used for registry login

### Add the workflow

This workflow will test, build, bush, deploy the app. See the `.github/workflows/deploy.yml` file for the steps.

#### Check out

Pull the latest state of the repo.

#### Add your tests

This is where you can add any tests before build/push/deploy.

#### Install doctl

We will need the `doctl` utility initialized with your `$DIGITALOCEAN_ACCESS_TOKEN`.

#### Docker build and push with cache

This is an important step. It handles the following:

- log into Docker Hub using the username and access token
- builds the new image using a previous build cache (no effect on 1st build)
- tags it with the git commit SHA
- pushed the new image to Docker Hub

As it turns out, the image tag needs to be unique to trigger a pull on the App Platform. So it is best to use the git commit SHA.

Using the build cache cuts down the build time if the dependencies have not changed (in this case from 9 minutes to 1 minute).

#### Update app spec

This is where we use environment variables and substitute these into the multi-line text, and save it into the `update.yaml` file.

#### Update the app

Finally, update app in App Platform using the new `update.yaml` app spec file. The new file has the updated tag based on the commit SHA, so the image pull will happen as expected.
