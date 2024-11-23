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
## EKS
- update kubeconfig
```bash
# rm -rf ~/.kube/config
aws eks update-kubeconfig --region ca-central-1 --name lab-dev-k8s
```
- add root user or any other user to EKS
> example of user.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: 
  mapUsers: |
    - userarn: arn:aws:iam::123456789:root
      username: eks-admin
      groups:
        - system:masters
```
> create or modify user yaml
```
# kubectl apply -f ./user.yaml
kubectl edit configmap aws-auth -n kube-system
```
