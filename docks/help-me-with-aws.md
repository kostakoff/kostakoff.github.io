# Instruction

## IAM pass role for deploy-sa
- iam policy json
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": [
                "arn:aws:iam::123456789:role/eks-sa",
                "arn:aws:iam::123456789:instance-profile/eksNodeGroup-sa"
            ],
            "Condition": {
                "StringEquals": {
                    "iam:PassedToService": "eks.amazonaws.com"
                }
            }
        }
    ]
}
```
> Where: arn:aws:iam::123456789:role/eks-sa - role that we wanna grant to EKS cluster
