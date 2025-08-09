# Ingress Controller
  An Ingress Controller in Kubernetes is like a smart traffic cop 🚦 for your cluster—it manages how external users access services running inside the cluster, especially HTTP and HTTPS traffic.

Follow the below steps in terminal and do it

### OIDC provider
trust and authentication provider

REGION_CODE=us-east-1
CLUSTER_NAME=roboshop-dev
ACC_ID=315069654700
```
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```

### Download IAM Policy

```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.4/docs/install/iam_policy.json
```

### Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

### Create IAM Role and K8 ServiceAccount. 
Mapping between IAM ROle and K8 Service account

 k8 is independent platform as a service. K8 has its own RBAC. Why should we integrate K8 RBAC and IAM. Because AWS services should have IAM permissions to create/manage other services. to create AWS services from K8 resources K8 SA should integrate with AWS IAM Role.
```
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve
```

### Install Load balancer controller drivers

```
helm repo add eks https://aws.github.io/eks-charts
```

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```



![alt text](k8-Ingress/Screenshot 2025-08-09 164647)
