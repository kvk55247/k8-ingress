### OIDC provider
`````````````````

REGION_CODE=us-east-1
CLUSTER_NAME=roboshop

````
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve

```

### download IAM policy
````
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/install/iam_policy.json


### Create IAM policy

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

Create an IAM role and Kubernetes ServiceAccount for the LBC.Mapping between IAM role and k8 service account
````
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::784585544641:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve


### Installing Loadbalancer driver via helm
```
helm repo add eks https://aws.github.io/eks-charts


helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller