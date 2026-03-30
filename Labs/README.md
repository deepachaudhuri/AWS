# EKS CloudFormation Learning Guide

This workspace includes a CloudFormation template to deploy:
- 1 VPC with /24 CIDR (10.0.0.0/24)
- 2 public subnets (/26 each)
- 1 EKS cluster (Kubernetes 1.34)
- 1 managed EKS node group (1 Linux node, t3.micro, AL2023_x86_64)

## Files
- `eks-cluster-cf-template.yaml`: CloudFormation template
- `README.md`: this file

## Prerequisites
- AWS CLI configured with credentials and region
- `aws` CLI v2 (recommended)
- `kubectl` installed
- `eksctl` optional, not used in this template

## Deploy
1. From terminal, navigate to this folder:
   ```bash
   cd "$HOME/OneDrive/Desktop/AWS/Labs"
   ```

2. Set region and kubeconfig variables:
   ```bash
   export AWS_REGION=us-east-2
   export KUBECONFIG="$HOME/.kube/config"
   ```

3. Create a CloudFormation stack:
   ```bash
   aws cloudformation deploy --region "$AWS_REGION" --template-file eks-cluster-cf-template.yaml --stack-name interview-eks-stack --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
   ```

4. Wait until stack creation completes:
   ```bash
   aws cloudformation describe-stacks --region "$AWS_REGION" --stack-name interview-eks-stack --query 'Stacks[0].StackStatus' --output text
   ```

## Configure kubectl
1. Get cluster name from output or use default `interview-eks-cluster`.
2. Update kubeconfig:
   ```bash
   aws eks update-kubeconfig --name interview-eks-cluster --region "$AWS_REGION" --kubeconfig "$KUBECONFIG"
   ```

3. Verify nodes:
   ```powershell
   kubectl get nodes
   ```

## Test deployment
1. Create a test namespace:
   ```powershell
   kubectl create ns demo
   ```
2. Deploy nginx:
   ```powershell
   kubectl apply -n demo -f https://k8s.io/examples/application/deployment.yaml
   kubectl wait --for=condition=available deploy/nginx-deployment -n demo --timeout=120s
   kubectl get pods -n demo
   ```

3. Cleanup test resources:
   ```powershell
   kubectl delete ns demo
   ```

## Delete stack
```powershell
aws cloudformation delete-stack --stack-name interview-eks-stack
```

## Notes
- The template uses one managed node group `t3.micro` with a single instance.
- The node group AMI type is `AL2023_x86_64` to support current Kubernetes versions.
- The VPC CIDR is `10.0.0.0/24` and subnets are `.0/26` and `.64/26`.
- Keep an eye on costs and cleanup after interview practice.
