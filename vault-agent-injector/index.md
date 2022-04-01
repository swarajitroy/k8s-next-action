# Vault Agent Sidecar Injector
---


## A. vault-agent-injector-cfg 
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

## B. Adding annotations to demo pod
---

```
ubuntu@ip-172-31-22-219:~$ cat demo-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
     vault.hashicorp.com/agent-inject: 'true'
     vault.hashicorp.com/role: 'demo-auth'
     vault.hashicorp.com/agent-inject-secret-database-config.txt: 'kv-v2/data/database/config'
  creationTimestamp: null
  labels:
    run: demo-pod
  name: demo-pod
spec:
  serviceAccountName: demo-user
  containers:
  - image: nginx
    name: demo-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
```
ubuntu@ip-172-31-22-219:~$ kubectl exec -it demo-pod -- /bin/sh
Defaulted container "demo-pod" out of: demo-pod, vault-agent, vault-agent-init (init)
# cd /vault/secrets
# ls
database-config.txt

# cat database-config.txt
data: map[password:pass123 username:postgres]
metadata: map[created_time:2022-03-29T17:57:35.985186799Z custom_metadata:<nil> deletion_time: destroyed:false version:2]

```

Inject one init container named vault-agent-init and one sidecar container named vault-agent, as well as an emptyDir type volume named vault-secrets . Also, the vault-secrets volume is mounted in the init container, the sidecar container, and the app container with the /vault/secrets/ directory. The secret is stored in the vault-secrets volume.

## C. Apply a template to the injected secrets
---
The structure of the injected secrets may need to be structured in a way for an application to use. Before writing the secrets to the file system a template can structure the data. To apply this template a new set of annotations need to be applied.
