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

