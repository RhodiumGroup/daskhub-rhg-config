# daskhub-rhg-config
Daskhub-flavored Jupyterhub deployment configuration for argocd.

This is a demonstration sandbox.

## Fresh deployment

### Setup (`argocd`)
To begin, you need to have `argocd` deployed on a cluster. If it is not already deployed, you can do so with

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch deploy argocd-server -n argocd -p '[{"op": "add", "path": "/spec/template/spec/containers/0/command/-", "value": "--disable-auth"}]' --type json
```

This sets up `argocd` for minimalist and dev envrionments, to be interacted with through the `argocd` CLI.

### Deploying apps

**NOTE:** For each of the following deployment workflows, if deploying with daskhub version 2021.6.0 or earlier, you will also need to pass the parameter override `daskhub.jupyterhub.proxy.secretToken`. You will need to pass this parameter following your first upgrade to 2021.6.1 or later, but afterwards you no longer need the parameter override.

Deploy the app fresh, using automated syncing, with `argocd` from the CLI with:

```bash
export JHUB_LOADBALANCERIP="127.0.0.1"
export JHUB_HOSTS="{fakewebsite.com}"
export JHUB_CLIENTID="abc999"
export JHUB_CLIENTSECRET="cba666"
export JHUB_CALLBACKURL="https://fakewebsite.com/hub/oauth_callback"
export DASK_APITOKEN="xyz111"
export DEPLOY_NAMESPACE="daskhub"
export DEPLOY_REVISION="v1.2.3"

kubectl create namespace $DEPLOY_NAMESPACE
argocd app create daskhub \
    --repo https://github.com/RhodiumGroup/daskhub-rhg-config.git \
    --revision $DEPLOY_REVISION \
    --path daskhub-rhg \
    --values values.yaml \
    --values values-prod.yaml \
    --values values-users.yaml \
    --parameter daskhub.jupyterhub.proxy.service.loadBalancerIP=$JHUB_LOADBALANCERIP \
    --parameter daskhub.jupyterhub.proxy.https.hosts=$JHUB_HOSTS \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_id=$JHUB_CLIENTID \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_secret=$JHUB_CLIENTSECRET \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.oauth_callback_url=$JHUB_CALLBACKURL \
    --parameter daskhub.jupyterhub.hub.services.dask-gateway.apiToken=$DASK_APITOKEN \
    --parameter daskhub.dask-gateway.gateway.auth.jupyterhub.apiToken=$DASK_APITOKEN \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace $DEPLOY_NAMESPACE \
    --sync-policy automated \
    --auto-prune \
    --self-heal \
    --port-forward-namespace argocd
```

This will automatically sync to to git tag "v1.2.3". When a new tag is created, you can set the app to sync to this tag with

```bash
argocd app set daskhub --revision <new-tag> --port-forward-namespace argocd
```


Alternatively, you can deploy an app tracking the `main` branch with

```bash
export JHUB_LOADBALANCERIP="127.0.0.1"
export JHUB_HOSTS="{fakewebsite-dev.com}"
export JHUB_CLIENTID="abc999"
export JHUB_CLIENTSECRET="cba666"
export JHUB_CALLBACKURL="https://fakewebsite-dev.com/hub/oauth_callback"
export DASK_APITOKEN="xyz111"
export DEPLOY_NAMESPACE="daskhub-dev"

kubectl create namespace $DEPLOY_NAMESPACE
argocd app create daskhub-dev \
    --repo https://github.com/RhodiumGroup/daskhub-rhg-config.git \
    --revision main \
    --path daskhub-rhg \
    --values values.yaml \
    --values values-dev.yaml \
    --values values-users.yaml \
    --parameter daskhub.jupyterhub.proxy.service.loadBalancerIP=$JHUB_LOADBALANCERIP \
    --parameter daskhub.jupyterhub.proxy.https.hosts=$JHUB_HOSTS \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_id=$JHUB_CLIENTID \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_secret=$JHUB_CLIENTSECRET \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.oauth_callback_url=$JHUB_CALLBACKURL \
    --parameter daskhub.jupyterhub.hub.services.dask-gateway.apiToken=$DASK_APITOKEN \
    --parameter daskhub.dask-gateway.gateway.auth.jupyterhub.apiToken=$DASK_APITOKEN \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace $DEPLOY_NAMESPACE \
    --sync-policy automated \
    --auto-prune \
    --self-heal \
    --port-forward-namespace argocd
```

This uses `dev` environment values for the helm chart and deploys to a `daskhub-dev` namespace.
