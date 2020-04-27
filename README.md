**AKS - configure traefik with cert-manager with letsencrypt**

**Create namespace traefik**

`k create namespace traefik`

**Install Traefik**

`helm install traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set fullnameOverride=customtraefik --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --version 1.85.0`


**Install cert manager:**
`
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml --namespace traefik 

kubectl label namespace traefik certmanager.k8s.io/disable-validation=true

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm install cert-manager --namespace traefik --version v0.12.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt --set ingressShim.defaultIssuerKind=ClusterIssuer
`

**Test cert manager pods**

`k get pods --namespace traefik`

**NOTE: AKS creates a public ip while installing traefik as ingress. Note the exernal IP.** 

`k get svc -n traefik`

**public ip alias can be used as a domain for the cert.**

**Configure the traefik ingress controller to use HTTPS**

**Create Letsencrypt cluster issuer:**
` k apply -f letsencrypt-clusterissuer.yaml --namespace traefik`

**Check cluster issuer status:**

 `k get clusterissuer -n traefik`

**Upgrade traefik to use https**

`helm upgrade traefik stable/traefik --namespace traefik --set kubernetes.ingressClass=traefik --set rbac.enabled=true --set kubernetes.ingressEndpoint.useDefaultPublishedService=true --version 1.85.0 --set ssl.enabled=true --set ssl.enforced=true --set ssl.permanentRedirect=true`

**Generate certificate**

`k apply -f cert.yml`

**View Generated Certificate status**

`k get cert`

**We will use the generated cert for our azure vote service:**

check azure-vote.yaml

**Deploy service:**

`k apply -f azure-vote-yaml`
