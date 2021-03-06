# Getting Started with the Kubernetes Secrets Store CSI Driver

In Kubernetes, it can be difficult to keep application API keys, access tokens and passwords safe. There are several different approaches to solving this problem, and in this talk Kim will demonstrate how to install Hashicorp Vault and the Secrets Store CSI Driver so that your applications can access secrets stored in ephemeral volumes.

This tutorial was written for a Data on Kubernetes Talk given on 21 April 2022. [Getting Started with the Kubernetes Secrets Store CSI Driver](https://www.meetup.com/Data-on-Kubernetes-community/events/285208289/)

## Prerequisites 
- A Kubernetes Cluster using version 1.19+
- `kubectl` 
- Helm 3 

## Resources 
- [Secrets Store CSI Driver Documentation](https://secrets-store-csi-driver.sigs.k8s.io/introduction.html)
- [Kubernetes CSI Developer Documentation](https://kubernetes-csi.github.io/docs/ephemeral-local-volumes.html)
- [HashiCorp Learn: Mount Vault Secrets through Container Storage Interface (CSI) Volume](https://learn.hashicorp.com/tutorials/vault/kubernetes-secret-store-driver)
- [Kubernetes Auth Method](https://www.vaultproject.io/docs/auth/kubernetes)




