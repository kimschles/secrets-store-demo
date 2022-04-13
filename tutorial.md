# Step-by-Step Instructions 

## Prepare Your Cluster
Check that you can connect to your cluster 

```sh
kubectl get nodes
```

Create a `demo` namespace 

```sh
kubectl create ns demo
```

## Install HashiCorp Vault

Add Hashicorp to your Helm Repositories 

``` sh
helm repo add hashicorp https://helm.releases.hashicorp.com
```

Update the Hashicorp repo 

``` sh 
helm repo update hashicorp 
``` 

Install Vault in the `demo` namespace. Ensure that the following options are set.  

Note: this is an internal instance of vault. You can also setup an external instance outside of a kubernetes cluster

``` sh
helm install vault hashicorp/vault -n demo \
--set "server.dev.enabled=true" \
--set "injector.enabled=false" \
--set "csi.enabled=true"
```     

## Store a password using the `kv` engine (change this to API key or something else)

Exec in to the `vault-0` pod 

```sh
kubectl exec vault-0 -n demo -it -- /bin/sh 
```

Create a secret at the path xxx/xxx and with a key-value pair (change these specifics)

```sh
vault kv put secret/db-pass password="db-secret-password"
```

Check that the secret has been saved 

```sh
vault kv get secret/db-pass
```

## Something, something Kubernetes Auth 

Enable Kubernetes Auth in Vault

```sh
vault auth enable kubernetes
```

From the tutorial: 
> Configure the Kubernetes authentication method to use the service account token, the location of the Kubernetes host, and its certificate.

```sh
vault write auth/kubernetes/config \
issuer="https://kubernetes.default.svc.cluster.local" \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

Create a policy called 'internal-app' that will allow the SSCD to read the mounts and the secrets itself. 

```sh 
vault policy write internal-app - <<EOF
path "secret/data/db-pass" {
  capabilities = ["read"]
}
EOF
```

> Finally, create a Kubernetes authentication role named database that binds this policy with a Kubernetes service account named webapp-sa.

```sh
vault write auth/kubernetes/role/database \
    bound_service_account_names=webapp-sa \
    bound_service_account_namespaces=default \
    policies=internal-app \
    ttl=20m
Success! Data written to: auth/kubernetes/role/database
``` 
Exit out of the `vault-0` pod
```
exit
```

## Install the Secrets Store CSI Driver 

Add Secrets Store CSI Driver to your Helm Repositories 

```sh 
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
``` 

Update the repo 

```sh
helm repo update secrets-store-csi-driver
```

Install the Secrets Store CSI Driver in the `demo` namespace. 

```sh
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --namespace demo
``` 

## Update Your App to 






Create a SecretProviderClass Object

Update your Deployment so it references the SSCD


Secret Content is Mounted on Pod Start



     


1. Application 
    - env var 
    - db password 


Notes: 
    - Define Container Storage Interface (CSI) volume    
    - Define ServiceAccount 