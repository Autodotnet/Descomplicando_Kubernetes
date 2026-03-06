## 

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl top - usado para ver utilizaçao de cpu/memória


kubectl create deployment --image nginx nginx --port 80 -o yaml --dry-run=client > nginx-hpa.yaml

kubectl expose deployment nginx

## Expondo o Locust
kubectl port-forward deployment/locust-giropops 8000:8089

nginx.default.svc.cluster.local

Criar hpa pro giropops senhas para cpu e memoria 50% pra cada

- stress test do giropops senhas


## Minikube cluster

minikube start --nodes 2 -p cluster-k8s
minikube start --nodes 2 -p cluster-k8s
/etc/sysctl.d/99-inotify.conf 

## Metrics serve no mini-kube
minikube addons enable metrics-server

## Acessar app em um minikube exposta via nodeport

Rodar comando minikube ip
- pegar o IP e jogar no browser + a porta
minikube ip --profile cluster-k8s