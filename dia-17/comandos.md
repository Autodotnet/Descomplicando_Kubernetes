# Configurando cluster Eks

eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name=eks-cluster --version 1.35  --node-type t3.medium --nodes=2 \
--nodes-min=1 --nodes-max=3 --managed

## Instalando addon AWS VPC CNI

eksctl create addon --name vpc-cni --version v1.21.1-eksbuild.5 --cluster eks-cluster --force

## Habilitar Network Policies no EKS
"enableNetworkPolicy": "true"

## Ingress nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml

aplicar yamls da aula-9 e ingress-3


## container para teste de netpol
k run -ti alpine --image alpine -- sh
k exec -ti alpine -n giropops -- sh

## criar deployment na namespace giropops

## comandos uteis

k create ns giropops

k get svc -n ingress-nginx

curl giropops-senhas.giropops.svc.cluster.local:5000

apk add redis
redis-cli -h redis-service.giropops.svc ping

eksctl delete cluster my-cluster --region=us-east-1