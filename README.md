# daskhub-rhg-config
Daskhub-flavored Jupyterhub deployment configuration.

This is a demonstration sandbox.


Deploy the app fresh, using automated syncing, with `argocd` from the CLI with:

```
export JHUB_LOADBALANCERIP="127.0. 0.1"
export JHUB_HOSTS="{fakewebsite.com}"
export JHUB_SECRETTOKEN="fakenumbers"
export JHUB_CLIENTID="abc999"
export JHUB_CLIENTSECRET="cba666"
export JHUB_CALLBACKURL="https://fakewebsite.com/hub/oauth_callback"
export DASK_APITOKEN="xyz111"
export JHUB_APITOKEN="zyx222"

kubectl create namespace daskhub
argocd app create daskhub \
    --repo https://github.com/RhodiumGroup/daskhub-rhg-config.git \
    --revision main \
    --path daskhub-rhg \
    --values values.yaml \
    --values values-prod.yaml \
    --parameter jupyterhub.proxy.service.loadBalancerIP=$JHUB_LOADBALANCERIP \
    --parameter jupyterhub.proxy.https.hosts=$JHUB_HOSTS \
    --parameter jupyterhub.proxy.secretToken=$JHUB_SECRETTOKEN \
    --parameter jupyterhub.auth.github.clientId=$JHUB_CLIENTID \
    --parameter jupyterhub.auth.github.clientSecret=$JHUB_CLIENTSECRET \
    --parameter jupyterhub.auth.github.callbackUrl=$JHUB_CALLBACKURL \
    --parameter jupyterhub.hub.services.dask-gateway.apiToken=$DASK_APITOKEN \
    --parameter dask-gateway.gateway.auth.jupyterhub.apiToken=$JHUB_APITOKEN\ 
    --dest-server https://kubernetes.default.svc \
    --dest-namespace daskhub \
    --sync-policy automated \
    --auto-prune \
    --port-forward-namespace argocd
```