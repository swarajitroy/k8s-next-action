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

## Install client side Kubeseal tool
---

```
ubuntu@ip-172-31-22-219:~$ sudo wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.16.0/kubeseal-linux-amd64 -O /usr/local/bin/kubeseal
--2022-04-03 14:42:36--  https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.16.0/kubeseal-linux-amd64
Resolving github.com (github.com)... 140.82.114.3
Connecting to github.com (github.com)|140.82.114.3|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/92702519/38955200-b1e1-11eb-9a84-0e13336d831e?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220403%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220403T144236Z&X-Amz-Expires=300&X-Amz-Signature=ab0941687fdea8e708cb3e3f42093b48419dbe07f43060963a92c3a2b8136171&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=92702519&response-content-disposition=attachment%3B%20filename%3Dkubeseal-linux-amd64&response-content-type=application%2Foctet-stream [following]
--2022-04-03 14:42:36--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/92702519/38955200-b1e1-11eb-9a84-0e13336d831e?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20220403%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220403T144236Z&X-Amz-Expires=300&X-Amz-Signature=ab0941687fdea8e708cb3e3f42093b48419dbe07f43060963a92c3a2b8136171&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=92702519&response-content-disposition=attachment%3B%20filename%3Dkubeseal-linux-amd64&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.109.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 35626806 (34M) [application/octet-stream]
Saving to: ‘/usr/local/bin/kubeseal’

/usr/local/bin/kubeseal                         100%[====================================================================================================>]  33.98M  69.4MB/s    in 0.5s

2022-04-03 14:42:37 (69.4 MB/s) - ‘/usr/local/bin/kubeseal’ saved [35626806/35626806]

ubuntu@ip-172-31-22-219:~$ sudo chmod 755 /usr/local/bin/kubeseal
ubuntu@ip-172-31-22-219:~$ kubeseal --version
kubeseal version: v0.16.0

ubuntu@ip-172-31-22-219:~$ kubectl -n kube-system patch svc sealed-secrets-controller --type='json' -p='[{"op": "remove", "path": "/spec/ports/0/name"}, {"op": "replace", "path": "/spec/ports/0/targetPort", "value":8080}]'
service/sealed-secrets-controller patched

ubuntu@ip-172-31-22-219:~$ kubectl create secret generic secret-name --dry-run=client --from-literal=password=welcome123 -o yaml | kubeseal --controller-name=sealed-secrets-controller --controller-namespace=kube-system  --format yaml > mysealedsecret.yaml

ubuntu@ip-172-31-22-219:~$ cat mysealedsecret.yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: secret-name
  namespace: default
spec:
  encryptedData:
    password: AgAZcTaJmGMFfxalJy9NMBIHrw0q9gKFH18GnfbcMQs2JKc179SmIi1XyG+qJ82U+mnLJV/dohKg7gyeh4EButqayNCsXQGK0JpxkKjh/xsuLE+0sKT7uWBLp6PhOpYm9kt4VCDD7btUgWwByVTq5MHrMS+D95hTqWgxS7q0FFeKB+VVtK/IaolDC4xUBl3yIEpCmWAayuKvk0kCXhNnWlP7Oi0LeOYAHRnFZiGmQzae75opZrZUt1VDZ4qIuQh90s+uobv3f7EnywOu2EI59a13OLuij4Fo2qll1O9Ys+6liSWJIMaWPMLt7jhcoBukdLrphH5JDZbgrGQ9jvgupLAITv6fukn/ZNinLlF+Hd6bK8Cwco80Z8fIGiitmfVeSb4pCYyAuiXda7fiIg7uPlSnqQl+hoFEP2m0BwBrHD7u199fabZgD2ELNqcnHamtp551qT+W4/xs81gnUIyyi+TppOdaNhlrQvSKYCJ+D5fk/nP9Tgo6UpDSreuKlcF0iRhmQ8eq3hsEj74FfAp1pPgsB22bD+8tagKtnNIqsB9e5k87C+AWxM79mukx5QgpWIpBjtxTBKP1oeIrnaQcdSPuT6ybyzKEeFRQnBj2YBm1BWjDxBfe9lvfZtWjrsIvIkXaEIKeZ60HYEp2PnCeddp9rOMjv3TRwV3Ccn/N2xgnI6aU216xq7MbUJgRUYg58Zd8O5nTin5ALjd3
  template:
    data: null
    metadata:
      creationTimestamp: null
      name: secret-name
      namespace: default

ubuntu@ip-172-31-22-219:~$ kubectl create -f mysealedsecret.yaml
sealedsecret.bitnami.com/secret-name created

ubuntu@ip-172-31-22-219:~$ kubectl get SealedSecret
NAME          AGE
secret-name   27s

ubuntu@ip-172-31-22-219:~$ kubectl get secret secret-name
NAME          TYPE     DATA   AGE
secret-name   Opaque   1      56s


ubuntu@ip-172-31-22-219:~$ kubectl get secret secret-name -o yaml | grep password
  password: d2VsY29tZTEyMw==
  
ubuntu@ip-172-31-22-219:~$ echo 'd2VsY29tZTEyMw==' | base64 -d
welcome123

```
