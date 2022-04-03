
# AWS Secret Manager
---

## Create a Secret
---

```
C:\Users\SWARAJITROY>aws --version
aws-cli/2.1.28 Python/3.8.8 Windows/10 exe/AMD64 prompt/off

C:\Users\SWARAJITROY>aws secretsmanager get-secret-value --secret-id=arn:aws:secretsmanager:us-east-2:xyx:secret:prod/apps/myapp-kPZ8Er
{
    "ARN": "arn:aws:secretsmanager:us-east-2:xyz:secret:prod/apps/myapp-kPZ8Er",
    "Name": "prod/apps/myapp",
    "VersionId": "eso",
    "SecretString": "{\"password\":\"welcome123\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2022-04-03T15:41:59.485000+05:30"
}



```


## Create an IAM policy to read secrets
-- 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:ListSecrets",
                "secretsmanager:GetSecretValue"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}


```

## Create an IAM User to attach policy
-- 

