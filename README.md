'''
The idea if this repo is practice and be familiar with kubernetes, deployments, config files etc.

We are going to deploy 3 pods in minikube (local cluster), one for PgAdmin, one for keycloak and one for a PostgreSQL Db using Bitnami package solution (in reality are deployed 2 pods for postgres, one for the Db and another one for postgres PgPool.) 

Read about bitnami PostgreSQL in this link: https://bitnami.com/stack/postgresql

All this pods we are goint to inject Envoy Istio proxy.

Note: consider that inject istio consume a lot of resources, take into account that we are going to work using minikube (menas locally) so we are going to use the resoruces of our local computer. 

In case that your computer is not powerfull enough or you feel that the resoruces available are not enough, just skip the Istio step.
'''

# Prerequisites.
Install minikube

Install Helm charts

Install and setup kubectl.

'''
Note: If you don't know how to install them, feel free to google it or in youtube how to do it. There are tons of tutirials and is pretty straigthforward.
'''

------------------ Comands to execute -----------------------------

# start minikube
minikube start --cpus 6 --memory 8192 (to include istio)

or

minikube start (no Istio)


minikube addons enable ingress (first install nginx ingress)
# prerequisites
helm repo add bitnami https://charts.bitnami.com/bitnami
# when using minikube ingress addon ingress-nginx should be installed
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx -n ingress-nginx --create-namespace ingress-nginx/ingress-nginx

------------------

# Step 1: Deploy keycloak, PgAdmin and PostgreSQL Bitnami:

# create dedicated namespace for our deployments
kubectl create ns test
# create TLS cert
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout auth-tls.key -out auth-tls.crt -subj "/CN=keycloak.localtest.me/O=test"
# Create a secret
kubectl create secret -n test tls auth-tls-secret --key auth-tls.key --cert auth-tls.crt
# deploy PostgreSQL cluster - in dev we will use 1 replica.
helm install -n test keycloak-db bitnami/postgresql-ha --set postgresql.replicaCount=1
# deploy Keycloak cluster
kubectl apply -n test -f keycloak-config.yaml
# deploy PgAdmin
kubectl apply -n test -f pgadmin-config.yaml
# create HTTPS ingress for Keycloak
kubectl apply -n test -f keycloak-ingress.yaml

# Veryfi that pods are deployed:
kubectl -n test get pods


---------------------

# Step 2: Download and Install Istio (optional, in case that you have enough resources in your local machine)

# Dowonload Istio

```For mac or Linux you can use the next command:```

curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.17.1 sh - (this version can vary, to download the latest remove ISTIO_VERSION)

``` Or go to this link and download the latest version : https://github.com/istio/istio/releases ```

# Move to the istio package directory
cd istio-1.17.1
# Add the istioctl client to your PATH environment variable
export PATH=$PWD/bin:$PATH (for windows ussing the application to do that.)
# Move to bin folder and execute install istioctl
istioctl install
# Evony Proxy Injection
kubectl label ns test istio-injection=enabled
# Verify that your pods has been labeled
kubectl get ns test --show-labels
# Restart the pods, after the restart the Evony proxy will be injected in the pods that living in ns test
kubectl rollout restart deployment -n test

```Note: after restart the pods you will see that the pods now have an extra container, that extra container is Istio Envoy proxy injected as side card```

# Verify the pods with Istio
kubectl -n test get pods

----------------------

Start the tunnel:

# create tunnel
minikube tunnel

``` After start the tunnel you can go to https://auth.localtest.me and see the keycloak page.```

# Delete Minikube

minikube stop
minikube delete



