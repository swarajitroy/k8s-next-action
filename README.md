# hashicorp-vault

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

$ helm get manifest vault


 


```
