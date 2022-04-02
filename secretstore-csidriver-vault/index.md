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

```
ubuntu@ip-172-31-22-219:~$ kubectl get ds
NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
csi-secrets-store-csi-driver   1         1         1       1            1           kubernetes.io/os=linux   26m
vault-csi-provider             1         1         1       1            1           <none>                   7m52s

ubuntu@ip-172-31-22-219:~$ kubectl get SecretProviderClass
No resources found in default namespace.


```

## Craete a SecretProviderClass
---

```
ubuntu@ip-172-31-22-219:~$ cat secretproviderclass.yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: vault-csi-example
spec:
  provider: vault
  parameters:
    vaultAddress: "http://vault.default:8200"
    roleName: "demo-auth"
    objects: |
      - objectName: "db-credential.txt"
        secretPath: "kv-v2/data/database/config"
        secretKey: "password"

ubuntu@ip-172-31-22-219:~$ kubectl create -f secretproviderclass.yaml
Warning: secrets-store.csi.x-k8s.io/v1alpha1 is deprecated. Use secrets-store.csi.x-k8s.io/v1 instead.
secretproviderclass.secrets-store.csi.x-k8s.io/vault-csi-example created

ubuntu@ip-172-31-22-219:~$ kubectl get secretproviderclass
NAME                AGE
vault-csi-example   24s


```
