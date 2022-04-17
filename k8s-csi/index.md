# Amazon Elastic Block Store (EBS) CSI driver
---
| ID | Topic | Remarks |
| ----------- | ----------- | ----------- |
| A | [Create an IAM Policy and User](#a-create-an-iam-policy-user) |  IAM Policy and User |
| B | [Create Secret wrapping Access and Secret Key](#b-create-secret) || 
| C | [Deploy CSI Driver](#c-deploy-driver)||


## A. Create an IAM Policy, User  
---
https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/example-iam-policy.json

![image](https://user-images.githubusercontent.com/20844803/163722500-b7eb429a-655f-46ca-9bf2-1062db5b83ab.png)
![image](https://user-images.githubusercontent.com/20844803/163722676-98065329-c3c9-4e4d-a3ac-2ea71972ed35.png)

and keep the IAM user access and secret key handy with you

## B. Create Secret 
---

```
ubuntu@ip-172-31-22-219:~$ cat ebscsisecret.yaml
apiVersion: v1
data:
  access_key: L005cHYxCTU9taWxzeg==
  key_id: QUtJQpDN0M=
kind: Secret
metadata:
  creationTimestamp: null
  name: aws-secret
  namespace: kube-system

ubuntu@ip-172-31-22-219:~$ kubectl create -f ebscsisecret.yaml
secret/aws-secret created
ubuntu@ip-172-31-22-219:~$ kubectl get secret aws-secret -n kube-system
NAME         TYPE     DATA   AGE
aws-secret   Opaque   2      19s


```

## C. Deploy Driver
---

```
ubuntu@ip-172-31-22-219:~$ helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
"aws-ebs-csi-driver" has been added to your repositories
ubuntu@ip-172-31-22-219:~$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "secrets-store-csi-driver" chart repository
...Successfully got an update from the "sealed-secrets" chart repository
...Successfully got an update from the "aws-ebs-csi-driver" chart repository
...Successfully got an update from the "hashicorp" chart repository
...Successfully got an update from the "external-secrets" chart repository
Update Complete. ⎈Happy Helming!⎈
ubuntu@ip-172-31-22-219:~$ helm upgrade --install aws-ebs-csi-driver \
>     --namespace kube-system \
>     aws-ebs-csi-driver/aws-ebs-csi-driver
Release "aws-ebs-csi-driver" does not exist. Installing it now.
W0417 16:35:03.552472   28640 warnings.go:70] policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
W0417 16:35:03.617692   28640 warnings.go:70] policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
NAME: aws-ebs-csi-driver
LAST DEPLOYED: Sun Apr 17 16:35:03 2022
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
To verify that aws-ebs-csi-driver has started, run:

    kubectl get pod -n kube-system -l "app.kubernetes.io/name=aws-ebs-csi-driver,app.kubernetes.io/instance=aws-ebs-csi-driver"

NOTE: The [CSI Snapshotter](https://github.com/kubernetes-csi/external-snapshotter) controller and CRDs will no longer be installed as part of this chart and moving forward will be a prerequisite of using the snap shotting functionality.

ubuntu@ip-172-31-22-219:~$ kubectl get pod -n kube-system -l "app.kubernetes.io/name=aws-ebs-csi-driver,app.kubernetes.io/instance=aws-ebs-csi-driver"
NAME                                  READY   STATUS    RESTARTS   AGE
ebs-csi-controller-79f465b597-wmcmt   5/5     Running   1          63s
ebs-csi-controller-79f465b597-wnp6g   5/5     Running   1          63s
ebs-csi-node-7lk9w                    3/3     Running   1          63s

ubuntu@ip-172-31-22-219:~$ kubectl get CSIDriver
NAME                       ATTACHREQUIRED   PODINFOONMOUNT   STORAGECAPACITY   TOKENREQUESTS   REQUIRESREPUBLISH   MODES        AGE
ebs.csi.aws.com            true             false            false             <unset>         false               Persistent   2m19s



```

## D. Create storage class
---

```
ubuntu@ip-172-31-22-219:~$ cat ebscsi-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
provisioner: ebs.csi.aws.com # <-- The same name as the CSIDriver
metadata:
  name: swararoy-gp2
parameters: # <-- parameters for this CSIDriver
  encrypted: "true"
  type: gp2
allowVolumeExpansion: true
volumeBindingMode: Immediate


ubuntu@ip-172-31-22-219:~$ kubectl create -f ebscsi-storageclass.yaml
storageclass.storage.k8s.io/swararoy-gp2 created

ubuntu@ip-172-31-22-219:~$ kubectl describe storageclass swararoy-gp2
Name:                  swararoy-gp2
IsDefaultClass:        No
Annotations:           <none>
Provisioner:           ebs.csi.aws.com
Parameters:            encrypted=true,type=gp2
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>

```

## E. Provision Storage
---

```
ubuntu@ip-172-31-22-219:~$ cat swararoy-ebscsi-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: swararoy-ebscsi-pvc
spec:
  storageClassName: swararoy-gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

ubuntu@ip-172-31-22-219:~$ kubectl create -f swararoy-ebscsi-pvc.yaml
persistentvolumeclaim/swararoy-ebscsi-pvc created

ubuntu@ip-172-31-22-219:~$ kubectl get pvc swararoy-ebscsi-pvc
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
swararoy-ebscsi-pvc   Bound    pvc-0fd7583c-dce2-418f-9768-91ef09d2289e   10Gi       RWO            swararoy-gp2   32s

ubuntu@ip-172-31-22-219:~$ kubectl describe pv pvc-0fd7583c-dce2-418f-9768-91ef09d2289e
Name:              pvc-0fd7583c-dce2-418f-9768-91ef09d2289e
Labels:            <none>
Annotations:       pv.kubernetes.io/provisioned-by: ebs.csi.aws.com
Finalizers:        [kubernetes.io/pv-protection]
StorageClass:      swararoy-gp2
Status:            Bound
Claim:             default/swararoy-ebscsi-pvc
Reclaim Policy:    Delete
Access Modes:      RWO
VolumeMode:        Filesystem
Capacity:          10Gi
Node Affinity:
  Required Terms:
    Term 0:        topology.ebs.csi.aws.com/zone in [us-east-2b]
Message:
Source:
    Type:              CSI (a Container Storage Interface (CSI) volume source)
    Driver:            ebs.csi.aws.com
    FSType:            ext4
    VolumeHandle:      vol-0db7c9f23ebcf15be
    ReadOnly:          false
    VolumeAttributes:      storage.kubernetes.io/csiProvisionerIdentity=1650213342793-8081-ebs.csi.aws.com
Events:                <none>

```
![image](https://user-images.githubusercontent.com/20844803/163725901-e02b8c56-303f-460a-be0a-b784b9735518.png)
![image](https://user-images.githubusercontent.com/20844803/163726111-34915315-9ced-46f6-b627-11151e0e0534.png)


