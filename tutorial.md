# Step-by-Step Instructions for Setting Up the Kubernetes Secrets Store CSI Driver with HashiCorp Vault

## 1. Prepare Your Cluster
Check that you can connect to your cluster 

```sh
kubectl get nodes
```

Create a `demo` namespace 

```sh
kubectl create ns demo
```

## 2. Set up HashiCorp Vault

Add Hashicorp to your Helm Repositories 

``` sh
helm repo add hashicorp https://helm.releases.hashicorp.com
```

Update the Hashicorp repo 

``` sh 
helm repo update hashicorp 
``` 

Install a [dev-server](https://www.vaultproject.io/docs/concepts/dev-server) instance of Hashicorp Vault in the `demo` namespace. 

Note: In this tutorial, Vault is running inside the Kubernetes cluster. It is possible to setup an external instance outside of cluster. If you'd like to connect an External Vault to your cluster, check out HashiCorp Learn's Guide [How Integrate a Kubernetes Cluster with an External Vault](https://learn.hashicorp.com/tutorials/vault/kubernetes-external-vault).

``` sh
helm install vault hashicorp/vault -n demo \
    --set "server.dev.enabled=true" \
    --set "injector.enabled=false" \
    --set "csi.enabled=true"
```     

## 3. Store a credential using Vault's Key-Value Storage

Exec in to the `vault-0` pod 

```sh
kubectl exec vault-0 -n demo -it -- /bin/sh 
```

Create a secret at the path secret/token and with a key-value pair 

```sh
vault kv put secret/token token="a9ndf6f64ac1er"
```

Check that the secret has been saved 

```sh
vault kv get secret/token
```

## 4. Setup Kubernetes Auth in Vault

Enable Kubernetes Auth in Vault

```sh
vault auth enable kubernetes
```

Setup the configuration for Vault to authenticate to Kubernetes by pointing to the issuer, jwt, address of the Kubernetes host and the location of the certificate authority cert. 

```sh
vault write auth/kubernetes/config \
issuer="https://kubernetes.default.svc.cluster.local" \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

Create a policy called 'app-permissions' that will allow the SSCD to read the mounts and the secret itself. 

```sh 
vault policy write app-permissions - <<EOF
path "secret/data/token" {
  capabilities = ["read"]
}
EOF
```
Create a Kubernetes authentication role called `database` that binds the app-permissions policy with a service account called `app-sa`  


```sh
vault write auth/kubernetes/role/database \
    bound_service_account_names=app-sa \
    bound_service_account_namespaces=demo \
    policies=app-permissions \
    ttl=20m
``` 
Exit out of the `vault-0` pod

```sh
exit
```

## 5. Install the Secrets Store CSI Driver 

Add Secrets Store CSI Driver to your Helm Repositories 

```sh 
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
``` 

Update the repo 

```sh
helm repo update secrets-store-csi-driver
```

Install the Secrets Store CSI Driver in the `demo` namespace and enable secret sync

```sh
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --namespace demo \
--set syncSecret.enabled=true \
``` 

## 6. Create new Kubernetes Objects so Your App Can Use the CSI Driver to Create an Ephemeral Volume 

Create a `SecretProviderClass` Object

```sh
kubectl apply -f manifests/secret-provider-class.yaml 
```

Create a Service Account called `app-sa`

```sh
kubectl apply -f manifests/service-account.yaml
```

Create a Deployment that mounts a CSI Ephemeral Volume to your pod
 
```sh
kubectl apply -f manifests/deployment.yaml
```
The secret content is mounted on pod start. You can check by exec'ing into the busybox pod and looking at the path `/mnt/secrets-store`


## 7. Sync the as a Kubernetes Secret and set as an `ENV` var in your application pod

Configure the `secretObjects` block in the `SecretProviderClass` by removing the comments the `secretObjects` block in manifests/secret-provider-class.yaml. Then run

```sh
kubectl apply -f manifests/secret-provider-class.yaml
```

Add the secret as an `ENV` var in your application pod by removing the comments from `env` block in manifests/deployment.yaml and then run 

```sh
kubectl apply -f manifests/deployment.yaml
```
Check that the `ENV` var is set. 

Exec into the busybox pod 

```sh
kubectl exec busybox -demo -it -- /bin/sh
```

Check that the `API_TOKEN` `ENV` Var exists

```sh
echo $API_TOKEN
```

Exit out of the `busybox` pod

```sh
exit
```

Pat your self on the back. You're done! 🎉