# aws-eks-ingress

## ALB Ingress Controller on Amazon EKS

- Create an IAM OIDC provider and associate it with your cluster
```
$ eksctl utils associate-iam-oidc-provider --cluster=eksworkshop-eksctl --approve
```
```
[ℹ]  eksctl version 0.13.0
[ℹ]  using region us-east-2
[ℹ]  will create IAM Open ID Connect provider for cluster "eksworkshop" in "us-east-2"
[✔]  created IAM Open ID Connect provider for cluster "eksworkshop" in "us-east-2"
```

- Deploy RBAC Roles and RoleBindings needed by the AWS ALB Ingress controller:
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```

- Download the AWS ALB Ingress controller YAML into a local file:
```
$ curl -sS "https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml" > alb-ingress-controller.yaml
```

- Edit the AWS ALB Ingress controller YAML to include the clusterName of the Kubernetes (or) Amazon EKS cluster. Edit the –cluster-name flag to be the real name of our Kubernetes (or) Amazon EKS cluster in your alb-ingress-controller.yaml and Deploy the AWS ALB Ingress controller YAML:
```
$ kubectl apply -f alb-ingress-controller.yaml
```

- Verify that the deployment was successful and the controller started:
```
$ kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
```
```
-------------------------------------------------------------------------------
AWS ALB Ingress controller
  Release:    v1.1.4
  Build:      git-0db46039
  Repository: https://github.com/kubernetes-sigs/aws-alb-ingress-controller
-------------------------------------------------------------------------------
```

- Deploy Sample Application
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-namespace.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-deployment.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-service.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml
```

- Verify
```
$ kubectl get ingress/2048-ingress -n 2048-game
```

## NLB + Nginx Ingress
- Start by creating the mandatory resources for NGINX Ingress in your cluster. This also creates the NLB.
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller 0.32.0/deploy/static/provider/aws/deploy.yaml
```

- Create Services
```
$ kubectl apply -f https://raw.githubusercontent.com/cornellanthony/nlb-nginxIngress-eks/master/apple.yaml 
$ kubectl apply -f https://raw.githubusercontent.com/cornellanthony/nlb-nginxIngress-eks/master/banana.yaml
```

- Create a Self Signed Certificate
```
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout 
$ tls.key -out tls.crt -subj "/CN=example.com/O=example.com"
```

- Create the secret in the cluster
```
$ kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```

- Create the Ingress
```
$ kubectl create -f https://raw.githubusercontent.com/cornellanthony/nlb-nginxIngress-eks/master/example-ingress.yaml
```
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
  rules:
  - host: example.com
    http:
      paths:
        - path: /apple
          backend:
            serviceName: apple-service
            servicePort: 5678
        - path: /banana
          backend:
            serviceName: banana-service
            servicePort: 5678
```

- Set up Route 53 to have your domain pointed to the NLB (optional):
```
example.com.           A.    
ALIAS abf3d14967d6511e9903d12aa583c79b-e3b2965682e9fbde.elb.us-east-1.amazonaws.com
```

- Test your application:
```
$ curl  https://example.com/banana -k
Banana
 
$ curl  https://example.com/apple -k
Apple
```

## References
- https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
- https://aws.amazon.com/blogs/opensource/network-load-balancer-nginx-ingress-controller-eks/
- https://eksworkshop.com/beginner/130_exposing-service/ingress_controller_alb/
- https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/2048/2048-ingress.yaml
- https://www.edureka.co/blog/kubernetes-ingress-controller-nginx
