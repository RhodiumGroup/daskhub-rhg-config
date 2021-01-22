# daskhub-rhg-config
Daskhub-flavored Jupyterhub deployment configuration for argocd.

This is a demonstration sandbox.

## Fresh deployment
Deploy the app fresh, using automated syncing, with `argocd` from the CLI with:

```
export JHUB_LOADBALANCERIP="127.0.0.1"
export JHUB_HOSTS="{fakewebsite.com}"
export JHUB_SECRETTOKEN="fakenumbers"
export JHUB_CLIENTID="abc999"
export JHUB_CLIENTSECRET="cba666"
export JHUB_CALLBACKURL="https://fakewebsite.com/hub/oauth_callback"
export DASK_APITOKEN="xyz111"
export JHUB_APITOKEN="zyx222"
export DEPLOY_NAMESPACE="daskhub"
export DEPLOY_REVISION="v1.2.3"

kubectl create namespace $DEPLOY_NAMESPACE
argocd app create daskhub \
    --repo https://github.com/RhodiumGroup/daskhub-rhg-config.git \
    --revision $DEPLOY_REVISION \
    --path daskhub-rhg \
    --values values.yaml \
    --values values-prod.yaml \
    --parameter daskhub.jupyterhub.proxy.service.loadBalancerIP=$JHUB_LOADBALANCERIP \
    --parameter daskhub.jupyterhub.proxy.https.hosts=$JHUB_HOSTS \
    --parameter daskhub.jupyterhub.proxy.secretToken=$JHUB_SECRETTOKEN \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_id=$JHUB_CLIENTID \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_secret=$JHUB_CLIENTSECRET \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.oauth_callback_url=$JHUB_CALLBACKURL \
    --parameter daskhub.jupyterhub.hub.services.dask-gateway.apiToken=$DASK_APITOKEN \
    --parameter daskhub.dask-gateway.gateway.auth.jupyterhub.apiToken=$JHUB_APITOKEN \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace $DEPLOY_NAMESPACE \
    --sync-policy automated \
    --auto-prune \
    --port-forward-namespace argocd
```

This will automatically sync to to git tag "v1.2.3". When a new tag is created, you can set the app to sync to this tag with

```
argocd app set daskhub --revision <new-tag> --port-forward-namespace argocd
```


Alternatively, you can deploy an app tracking the `main` branch with

```
export JHUB_LOADBALANCERIP="127.0.0.1"
export JHUB_HOSTS="{fakewebsite-dev.com}"
export JHUB_SECRETTOKEN="otherfakenumbers"
export JHUB_CLIENTID="abc999"
export JHUB_CLIENTSECRET="cba666"
export JHUB_CALLBACKURL="https://fakewebsite-dev.com/hub/oauth_callback"
export DASK_APITOKEN="xyz111"
export JHUB_APITOKEN="zyx222"
export DEPLOY_NAMESPACE="daskhub-dev"

kubectl create namespace $DEPLOY_NAMESPACE
argocd app create daskhub-dev \
    --repo https://github.com/RhodiumGroup/daskhub-rhg-config.git \
    --revision main \
    --path daskhub-rhg \
    --values values.yaml \
    --values values-dev.yaml \
    --parameter daskhub.jupyterhub.proxy.service.loadBalancerIP=$JHUB_LOADBALANCERIP \
    --parameter daskhub.jupyterhub.proxy.https.hosts=$JHUB_HOSTS \
    --parameter daskhub.jupyterhub.proxy.secretToken=$JHUB_SECRETTOKEN \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_id=$JHUB_CLIENTID \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.client_secret=$JHUB_CLIENTSECRET \
    --parameter daskhub.jupyterhub.hub.config.GitHubOAuthenticator.oauth_callback_url=$JHUB_CALLBACKURL \
    --parameter daskhub.jupyterhub.hub.services.dask-gateway.apiToken=$DASK_APITOKEN \
    --parameter daskhub.dask-gateway.gateway.auth.jupyterhub.apiToken=$JHUB_APITOKEN \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace $DEPLOY_NAMESPACE \
    --sync-policy automated \
    --auto-prune \
    --port-forward-namespace argocd
```

This uses `dev` environment values for the helm chart and deploys to a `daskhub-dev` namespace.
