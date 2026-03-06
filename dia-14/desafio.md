# Desafio HPA

Fui desafiado pelo Jeferson a criar um hpa para a aplicação giropops-senhas usando as métricas de CPU e Memória e executar o stress test utilizando a ferramenta Locust para entender o escalonamento de pods quando uma aplicação recebe muitas requisições.

## Requisitos

1 - Criar cluster Kubernetes com 2 nodes no mínimo (metrics server não aceita clusters de 1 node).

2 - Instalar o Metrics server para o hpa se basear na hora de escalonar os pods.
3 - Subir a aplicação do Giropops-senhas com service clusterIP.
4 - Subir o Locust com service NodePort ou LB.
5 - Rodar o stress test e ver os pods aumentarem e depois diminuirem.

## Deploy

1 - Subi um cluster multi-node no Minibuke 
``` bash
minikube start --nodes 2 -p cluster-k8s

```
2 - Instalar o metrics server no minikube

``` bash
minikube addons enable metrics-server --profile cluster-k8s

```
3 - Aplicar os yaml do giropops senhas e redis com seus respectivos services. Conforme yamls ao lado.

``` bash
cd dk8s/dia-14
kubect apply -f giropops-senhas.yaml
kubect apply -f app-svc.yaml
kubect apply -f redis.yaml
kubect apply -f redis-svc.yaml
ou
kubect apply -f . <<< aplica todos os yamls

```

4 - Aplicar os yamls do Locust, aplicação responsável pelo stress test

``` bash
kubect apply -f locust.yaml
kubect apply -f locust-cm.yaml
kubect apply -f locust-svc.yaml
``` 

5 - Aplicar o HPA

``` bash
kubectl apply -f hpa.yaml 

```

6 - Acessar o Locust usando IP do Node

Como utilizei o service NodePort para expor o Locust fora do Cluster na porta 30085 precisei pegar o IP com o comando:

```bash 
minikube ip
``` 
E acessei dessa forma:
http://192.168.67.2:300085

![alt text](<Screenshot from 2026-03-06 20-05-21.png>)

Para rodar o teste usei a url do service do giropops-senhas

http://giropops-senhas.default.svc.cluster.local 

7 - Pod escalando depois de rodar o Locust

![alt text](<Screenshot from 2026-03-06 20-15-20.png>)