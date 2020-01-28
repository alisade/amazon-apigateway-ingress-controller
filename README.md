# Amazon API Gateway Ingress Controller

## Getting Started

The default configuration assumes you are using kube2iam to manage pod permissions.
To set up a role for this controller use the following command

```sh
export INSTANCE_ROLE_ARNS=`comma delimited list of k8s worker instance ARNs`
make iam
```

To build and deploy the controller

```sh
export IMG=`some ecr repository`
export IAMROLEARN=`the iam role arn created above`

mkdir -p ~/go
export GOPATH=~/go
export GOFLAGS="-mod=vendor"
go get github.com/awslabs/amazon-apigateway-ingress-controller
cd ~/go/src/github.com/awslabs/amazon-apigateway-ingress-controller
curl --silent --location -o /tmp/kb.tgz https://github.com/kubernetes-sigs/kubebuilder/releases/download/v2.2.0/kubebuilder_2.2.0_linux_amd64.tar.gz
tar xf /tmp/kb.tgz -C /tmp
sudo mkdir /usr/local/kubebuilder
sudo mv /tmp/kubebuilder*/bin /usr/local/kubebuilder

make docker-build
make docker-push
make deploy
```



## Example

```yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: foobar-ingress
  annotations:
    kubernetes.io/ingress.class: apigateway
    apigateway.ingress.kubernetes.io/stage-name: prod
    apigateway.ingress.kubernetes.io/client-arns: arn::foo,arn::bar
    apigateway.ingress.kubernetes.io/nginx-replicas: "3"
    apigateway.ingress.kubernetes.io/nginx-image: nginx:latest
    apigateway.ingress.kubernetes.io/nginx-service-port: "9090"
spec:
  rules:
    - http:
        paths:
        - backend:
            serviceName: foo-service
            servicePort: 8080
          path: /api/v1/foo
        - backend:
            serviceName: bar-service
            servicePort: 8080
          path: /api/v1/bar
```
