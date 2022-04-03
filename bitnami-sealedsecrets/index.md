# Sealed Secrets with Kubernetes
---

## Install the sealed secret operator
---

```
ubuntu@ip-172-31-22-219:~$ helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
"sealed-secrets" has been added to your repositories

ubuntu@ip-172-31-22-219:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "sealed-secrets" chart repository
...Successfully got an update from the "secrets-store-csi-driver" chart repository
...Successfully got an update from the "hashicorp" chart repository
...Successfully got an update from the "external-secrets" chart repository
Update Complete. ⎈Happy Helming!⎈

ubuntu@ip-172-31-22-219:~$ helm install sealed-secrets-controller -n kube-system sealed-secrets/sealed-secrets
NAME: sealed-secrets-controller
LAST DEPLOYED: Sun Apr  3 14:30:36 2022
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

You should now be able to create sealed secrets.

1. Install the client-side tool (kubeseal) as explained in the docs below:

    https://github.com/bitnami-labs/sealed-secrets#installation-from-source

2. Create a sealed secret file running the command below:

    kubectl create secret generic secret-name --dry-run=client --from-literal=foo=bar -o [json|yaml] | \
    kubeseal \
      --controller-name=sealed-secrets-controller \
      --controller-namespace=kube-system \
      --format yaml > mysealedsecret.[json|yaml]

The file mysealedsecret.[json|yaml] is a commitable file.

If you would rather not need access to the cluster to generate the sealed secret you can run:

    kubeseal \
      --controller-name=sealed-secrets-controller \
      --controller-namespace=kube-system \
      --fetch-cert > mycert.pem

to retrieve the public cert used for encryption and store it locally. You can then run 'kubeseal --cert mycert.pem' instead to use the local cert e.g.

    kubectl create secret generic secret-name --dry-run=client --from-literal=foo=bar -o [json|yaml] | \
    kubeseal \
      --controller-name=sealed-secrets-controller \
      --controller-namespace=kube-system \
      --format [json|yaml] --cert mycert.pem > mysealedsecret.[json|yaml]

3. Apply the sealed secret

    kubectl create -f mysealedsecret.[json|yaml]

Running 'kubectl get secret secret-name -o [json|yaml]' will show the decrypted secret that was generated from the sealed secret.

Both the SealedSecret and generated Secret must have the same name and namespace.


```
