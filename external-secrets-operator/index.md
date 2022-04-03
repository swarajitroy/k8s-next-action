# External Secrets Operator
---

## Install the external secrets operator (ESO)
---

https://external-secrets.io/v0.4.4/

```
ubuntu@ip-172-31-22-219:~$ helm repo add external-secrets https://charts.external-secrets.io
"external-secrets" has been added to your repositories

ubuntu@ip-172-31-22-219:~$ helm install external-secrets external-secrets/external-secrets -n external-secrets  --set installCRDs=true
NAME: external-secrets
LAST DEPLOYED: Sun Apr  3 09:46:27 2022
NAMESPACE: external-secrets
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
external-secrets has been deployed successfully!

In order to begin using ExternalSecrets, you will need to set up a SecretStore
or ClusterSecretStore resource (for example, by creating a 'vault' SecretStore).

More information on the different types of SecretStores and how to configure them
can be found in our Github: https://github.com/external-secrets/external-secrets

ubuntu@ip-172-31-22-219:~$ kubectl get crd | grep -i external
clustersecretstores.external-secrets.io                     2022-04-03T09:46:30Z
externalsecrets.external-secrets.io                         2022-04-03T09:46:30Z
secretstores.external-secrets.io                            2022-04-03T09:46:30Z


```

## Create an instance of SecretStore CRD
---

```
ubuntu@ip-172-31-22-219:~$ echo -n '1234' > ./access-key
ubuntu@ip-172-31-22-219:~$ echo -n '4567' > ./secret-access-key
ubuntu@ip-172-31-22-219:~$ kubectl create secret generic awssm-secret --from-file=./access-key  --from-file=./secret-access-key
secret/awssm-secret created

ubuntu@ip-172-31-22-219:~$ cat secret-store.yaml
apiVersion: external-secrets.io/v1alpha1
kind: SecretStore
metadata:
  name: my-secret-store
spec:
  provider:
    aws:  # set secretStore provider to AWS.
      service: SecretsManager # Configure service to be Secrets Manager
      region: us-east-2   # Region where the secret is.
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: awssm-secret # References the secret we created
            key: access-key
          secretAccessKeySecretRef:
            name: awssm-secret
            key: secret-access-key

ubuntu@ip-172-31-22-219:~$ kubectl create -f secret-store.yaml
secretstore.external-secrets.io/my-secret-store created

ubuntu@ip-172-31-22-219:~$ kubectl get SecretStore
NAME              AGE     STATUS
my-secret-store   3m58s   Valid



```

## Create an instance of ExternalSecret CRD
---

```
ubuntu@ip-172-31-22-219:~$ cat external-secret.yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: awssm-to-k8s-es
spec:
  refreshInterval: 1m
  secretStoreRef:
    name: my-secret-store
    kind: SecretStore
  target:
    name: secret-to-be-created
    creationPolicy: Owner
  data:
  - secretKey: firstname
    remoteRef:
      key: prod/apps/myapp
      property: password

ubuntu@ip-172-31-22-219:~$ kubectl get ExternalSecret
NAME              STORE             REFRESH INTERVAL   STATUS
awssm-to-k8s-es   my-secret-store   1m                 SecretSynced

ubuntu@ip-172-31-22-219:~$ kubectl get secrets secret-to-be-created
NAME                   TYPE     DATA   AGE
secret-to-be-created   Opaque   1      18m

ubuntu@ip-172-31-22-219:~$ kubectl get secrets secret-to-be-created -o yaml | grep firstname
  firstname: d2VsY29tZTEyMw==

ubuntu@ip-172-31-22-219:~$ echo 'd2VsY29tZTEyMw==' | base64 -d
welcome123

```
