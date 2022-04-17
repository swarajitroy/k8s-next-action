# Amazon Elastic Block Store (EBS) CSI driver
---
| ID | Topic | Remarks |
| ----------- | ----------- | ----------- |
| A | [Create an IAM Policy and User](#a-create-an-iam-policy-user) |  IAM Policy and User |
| B | [Create Secret wrapping Access and Secret Key]() || 


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
