# hashicorp-vault

| ID | Topic | Remarks |
| ----------- | ----------- | ----------- |
| A | [Install Vault](#a-install-vault) |  Installing Vault via Helm Chart ||
| B | [Understand the setup](#b-understand-the-setup) |  Vault and Vault Injector ||
| C | [Initialize Vault](#c-initialize-vault) |  Initialize Vault ||


## A. Install Vault

```
ubuntu@ip-172-31-22-219:~/linux-amd64$ helm version
version.BuildInfo{Version:"v3.8.0", GitCommit:"d14138609b01886f544b2025f5000351c9eb092e", GitTreeState:"clean", GoVersion:"go1.17.5"}

ubuntu@ip-172-31-22-219:~/linux-amd64$ helm repo add hashicorp https://helm.releases.hashicorp.com
"hashicorp" has been added to your repositories

ubuntu@ip-172-31-22-219:~/linux-amd64$ helm repo list | grep -i hashicorp
hashicorp       https://helm.releases.hashicorp.com

ubuntu@ip-172-31-22-219:~/linux-amd64$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "hashicorp" chart repository
Update Complete. ⎈Happy Helming!⎈

ubuntu@ip-172-31-22-219:~/linux-amd64$ helm install vault hashicorp/vault
NAME: vault
LAST DEPLOYED: Sat Feb 26 18:10:38 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

$ helm status vault


ubuntu@ip-172-31-22-219:~$ helm status vault
NAME: vault
LAST DEPLOYED: Sat Feb 26 18:10:38 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

ubuntu@ip-172-31-22-219:~$ kubectl exec -it vault-0 -- /bin/sh
/ $ vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        false
Sealed             true
Total Shares       0
Threshold          0
Unseal Progress    0/0
Unseal Nonce       n/a
Version            1.9.2
Storage Type       file
HA Enabled         false


ubuntu@ip-172-31-22-219:~$ kubectl get pods  | grep -i vault
vault-0                                  1/1     Running   0          16h
vault-agent-injector-58b6d499-c2qqk      1/1     Running   0          16h

$ helm get manifest vault

```

## B. Understand the setup 
---

## B.01. vault-agent-injector-cfg 
---

The configuration of the MutatingWebhookConfiguration tells - the admission controller will try to mutate any POD kubernetes resources for CREATE and UPDATE operations. Then to Mutate - it will consult the web hook - which is running as a service vault-agent-injector-svc. Agent injector is a kubernetes Mutating Admission Webhook. This controller intercepts pod CREATE and UPDATE requests looking for the metadata annotation vault.hashicorp.com/agent-inject: true in the requests. When found, the controller will inject the init container and the sidecar container. The init container is to pre-populate our secret, and the sidecar container is to keep that secret data in sync throughout our application's life cycle.

```
ubuntu@ip-172-31-22-219:~$ kubectl get MutatingWebhookConfiguration
NAME                       WEBHOOKS   AGE
vault-agent-injector-cfg   1          28d

ubuntu@ip-172-31-22-219:~$ kubectl  describe MutatingWebhookConfiguration vault-agent-injector-cfg

Webhooks:
  Admission Review Versions:
    v1beta1
    v1
  Client Config:
    Ca Bundle:  URS0tLS0tCg==
    Service:
      Name:        vault-agent-injector-svc
      Namespace:   default
      Path:        /mutate
      Port:        443
  Failure Policy:  Ignore
  Match Policy:    Equivalent
  Name:            vault.hashicorp.com
  Namespace Selector:
  Object Selector:
  Reinvocation Policy:  Never

 Rules:
    API Groups:

    API Versions:
      v1
    Operations:
      CREATE
      UPDATE
    Resources:
      pods
    Scope:          *
  Side Effects:     None
  Timeout Seconds:  10
  
```

The webhook runs as a separate service

```
ubuntu@ip-172-31-22-219:~$ kubectl get svc vault-agent-injector-svc
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
vault-agent-injector-svc   ClusterIP   10.*.*.*   <none>        443/TCP   28d
```

## B.02. vault service
---

```
ubuntu@ip-172-31-22-219:~$ kubectl get svc vault
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
vault   ClusterIP   10.*.*.*   <none>        8200/TCP,8201/TCP   28d

```



## C. Initialize Vault

```
 $ vault operator init
Unseal Key 1: lUn4Hzzd55/Mv4
Unseal Key 2: 8Taq0owMgshA
Unseal Key 3: /zELQkx3H0c+lz
Unseal Key 4: tERO9r22TnJzraIqHR
Unseal Key 5: AFuTli

Initial Root Token: s.JIQdpxqrRsM

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 keys to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.

/ $ vault status
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    0/3
Unseal Nonce       n/a
Version            1.9.2
Storage Type       file
HA Enabled         false


```


## D. Unseal Vault

```


/ $ vault operator unseal
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       d1c1f3f93668b4e
Version            1.9.2
Storage Type       file
HA Enabled         false
/ $ vault operator unseal
Unseal Key (will be hidden):
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    2/3
Unseal Nonce       d1c1f0d93668b4e
Version            1.9.2
Storage Type       file
HA Enabled         false
/ $ vault operator unseal
Unseal Key (will be hidden):
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.9.2
Storage Type    file
Cluster Name    vault-cluster-588ec97b
Cluster ID      c8fb7404-a581-65c7-d6b6-1e11e005f0b1
HA Enabled      false

```


