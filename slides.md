# Getting Started with the Kubernetes Secrets Store CSI Driver

--- 
## Kim Schlesinger

![inline left](https://res.cloudinary.com/kimschlesinger/image/upload/c_scale,w_550/v1606362929/00044_DVLP2019.jpg)


---
## Agenda 

1. Key Terms 
1. Demo
1. Q & A

---
## Key Terms 
**Ephemeral Volumes**

- Not PVs or PVCs 
- Volumes have the same lifecycle as a pod 
- Good for storing read-only data
- SIG-CSI 
- GA in Kubernetes v1.19    

---

|  | CSI Ephemeral Volumes | Generic Ephemeral Volumes |
| ---- | ---- | ---- |
| Follows pod lifecycle | ✅ | ✅ |
| Inline definition  | ✅ | ✅ |
| Specify storage class | ❌ | ✅ |
| 3rd Party Driver  | ✅ | ❌ |

---
## Kubernetes Secrets Store CSI Driver with HashiCorp Vault

--- 
## Demo 

--- 
## Q & A 

--- 
## Resources
- [Secrets Store CSI Driver Documentation](https://secrets-store-csi-driver.sigs.k8s.io/introduction.html)
- [Kubernetes CSI Developer Documentation](https://kubernetes-csi.github.io/docs/ephemeral-local-volumes.html)
- [HashiCorp Learn: Mount Vault Secrets through Container Storage Interface (CSI) Volume](https://learn.hashicorp.com/tutorials/vault/kubernetes-secret-store-driver)
- [Kubernetes Auth Method](https://www.vaultproject.io/docs/auth/kubernetes)


---
## Thank you! 

![inline left](https://media.giphy.com/media/po3NDGWuAE33qmWqe3/giphy.gif)

[@kimschles](https://twitter.com/kimschles)

[digitalocean.com](https://www.digitalocean.com)
[digitalocean.com/community](https://www.digitalocean.com/community)