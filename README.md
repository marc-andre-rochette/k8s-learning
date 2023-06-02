eksctl create cluster \
     --name eksdemo1 \
      --region us-east-1 \
       --without-nodegroup \
       --zones us-east-1a,us-east-1b \
       --profile sandbox-marochette


eksctl delete cluster \
     --name eksdemo1 \
      --region us-east-1 \
       --profile sandbox-marochette

eksctl --profile sandbox-marochette \
    --region us-east-1 \
    get clusters

eksctl --profile sandbox-marochette \
    --region us-east-1 \
    get nodegroup --cluster eksdemo1

eksctl --profile sandbox-marochette \
    --region us-east-1 \
    delete cluster eksdemo1


eksctl utils associate-iam-oidc-provider --region us-east-1 --profile sandbox-marochette --cluster eksdemo1 --approve

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://exemples/keycloak/iam_policy.json \
    --profile sandbox-marochette



eksctl create iamserviceaccount \
  --cluster=eksdemo1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::091632995049:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve \
  --profile sandbox-marochette \
  --region us-east-1


helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
   -n kube-system \
   --set clusterName=eksdemo1 \
   --set serviceAccount.create=false \
   --set serviceAccount.name=aws-load-balancer-controller \
   --set region=us-east-1 \
   --set vpcId=vpc-0da1845abc6553907 \
   --set image.repository=602401143452.dkr.ecr.us-east-1.amazonaws.com/amazon/aws-load-balancer-controller


eksctl create nodegroup --cluster=eksdemo1 \
                       --region=us-east-1 \
                       --name=eksdemo1-ng-public1 \
                       --node-type=t3.micro \
                       --nodes=3 \
                       --nodes-min=1 \
                       --nodes-max=4 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=kube-ec2-key-pair \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access \
                       --profile sandbox-marochette

kubectl apply -f keycloak-ingress-class.yaml
kubectl apply -f keycloak-namespace.yaml
kubectl apply -f keycloak-config.yaml
kubectl apply -f keycloak-deployment.yaml
kubectl apply -f keycloak-ingress.yaml
kubectl apply -f keycloak-service.yaml 