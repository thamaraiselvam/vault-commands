# Vault in Kubernetes

## Install vault in kubernetes on dev mode

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo list
helm repo update
helm install vault hashicorp/vault --set "server.dev.enabled=true"
```

## Components

> vault-0 - Vault server
> vault-agent-injector - Inject vault agents to pods

## Check vault server logs
```bash
kubectl logs vault-0
```

## Communicate to vault from local machine

```bash
kubectl port-forward vault-0 8200:8200
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=root
vault status
```

## Enable Kubernetes Authentication on Vault
Vault <-> K8S (Api server)

```bash
vault auth enable kubernetes
vault auth list
```

## Allow Vault to communicate to K8S
Vault -> K8S (Api server)

```bash
kubectl exec -it vault-0 sh

vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://${KUBERNETES_PORT_443_TCP_ADDR}:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
    issuer="https://kubernetes.default.svc.cluster.local"
```

## Allow Pod to communicate to Vault
K8S Pods -> Vault (To Access secret)

## Create Secret

```bash
vault kv put secret/my-application apiKey=xxx
vault kv get secret/my-application
```

## Create Policy
> Authorization

```bash
vault policy write my-application-read-policy ./resources/my-application-read.hcl
```

## Create Service Account and associate to a policy
> Authentication

```bash
kubectl apply -f ./resources/service-account.yml
```

```bash
vault write auth/kubernetes/role/my-application \
    bound_service_account_names=my-application \
    bound_service_account_namespaces=default \
    policies=my-application-read \
    ttl=1h
```

## Run deployment

```bash
kubectl apply -f ./resources/dobby.yml
```

> Two containers will be running
> How sidecar injected? - Mutation webhook -> Hijack
> Injector will keep looking into pods that it is interested in it and mutate the pods (figure out using annotiation)
    > Inject
    > What to inject
    > How to inject
    > Who (Permission role)
    > use this service accoint - authentication

## Dobby Deployment describe

```bash
kubectl describe pod <dobby-deploy>
```
There should be 3 containers

> vault-init  - Authenticate to vault server
> dobby  - Primary container
> vault-agent - Communicate to Vault and Fetch Secrets

## Check secret inside pod

```bash
kubectl exec -it <POD> bash
cat /vault/secret/my-application.json
```

## Update values and see if pod can sync new data

```bash
vault kv put secret/my-application apiKey=yyy
vault kv get secret/my-application
```

```bash
kubectl logs -f pod vault-agent
```
Check secret inside pod again.