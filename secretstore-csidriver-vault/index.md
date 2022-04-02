# Mount Vault Secrets through Container Storage Interface (CSI) Volume using Kubernetes Secrets Store CSI Driver
---

## Install the Kubernetes Secrets Store CSI Driver
---

```
ubuntu@ip-172-31-22-219:~$ helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
"secrets-store-csi-driver" has been added to your repositories

ubuntu@ip-172-31-22-219:~$ helm install csi secrets-store-csi-driver/secrets-store-csi-driver
NAME: csi
LAST DEPLOYED: Sat Apr  2 14:57:29 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The Secrets Store CSI Driver is getting deployed to your cluster.

To verify that Secrets Store CSI Driver has started, run:

  kubectl --namespace=default get pods -l "app=secrets-store-csi-driver"

Now you can follow these steps https://secrets-store-csi-driver.sigs.k8s.io/getting-started/usage.html
to create a SecretProviderClass resource, and a deployment using the SecretProviderClass.

ubuntu@ip-172-31-22-219:~$ helm upgrade vault hashicorp/vault --set "csi.enabled=true"
Release "vault" has been upgraded. Happy Helming!
NAME: vault
LAST DEPLOYED: Sat Apr  2 15:16:47 2022
NAMESPACE: default
STATUS: deployed
REVISION: 2
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault



```

## Understand the kubernetes resources created by Secrets Store CSI Driver
---

