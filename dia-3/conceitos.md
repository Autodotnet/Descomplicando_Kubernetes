### Deployment

- Possibilita a criação de replicas dos Pods para distribuir nos nós do nosso cluster.
- Se acontecer algo com uma das réplicas o ReplicaSet que faz parte do Deployment recria o pod.

Depois de criar o Deployment pode usar 
``` bash
kubectl get deployments -l app=nginx-deployment
```

k create deployment --image nginx --replicas 3 nginx-deployment

k exec -ti nginx-deployment-54789fb4df-kbbsg -- bash --nginx -v

k create namespace giropopos --dry-run=client -o yaml > namespace.yaml

### Atualizar o Deployment

Quando atualizamos um imagem o K8s vai recriando os container de forma gradual para não impactar a aplicação, conforme a saída abaixo.


allan@ALLAN-Z:~/Descomplicando_Kubernetes/dia-3$ k get pods -n strigus -w
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-6c4849c57d-754sm   1/1     Running             0          34s
nginx-deployment-6c4849c57d-87xlq   0/1     ContainerCreating   0          3s
nginx-deployment-6c4849c57d-bpwss   0/1     ContainerCreating   0          3s
nginx-deployment-6c4849c57d-bswj5   0/1     ContainerCreating   0          3s
nginx-deployment-6c4849c57d-djvjn   0/1     ContainerCreating   0          3s
nginx-deployment-6c4849c57d-dzvsz   0/1     ContainerCreating   0          3s
nginx-deployment-6c4849c57d-knkqc   1/1     Running             0          3s
nginx-deployment-6c4849c57d-kxcdt   0/1     ContainerCreating   0          3s
nginx-deployment-6c4849c57d-mjdwn   1/1     Running             0          3s
nginx-deployment-6c4849c57d-mlltx   1/1     Running             0          3s
nginx-deployment-6c4849c57d-mrlvl   1/1     Running             0          3s
nginx-deployment-6c4849c57d-q88tb   1/1     Running             0          3s
nginx-deployment-6c4849c57d-rwgqx   1/1     Running             0          35s
nginx-deployment-6c4849c57d-sqntt   1/1     Running             0          37s
nginx-deployment-6c4849c57d-swjxs   1/1     Running             0          37s
nginx-deployment-6c4849c57d-bswj5   1/1     Running             0          4s
nginx-deployment-6c4849c57d-87xlq   1/1     Running             0          4s
nginx-deployment-6c4849c57d-kxcdt   1/1     Running             0          5s
nginx-deployment-6c4849c57d-dzvsz   1/1     Running             0          5s
nginx-deployment-6c4849c57d-bpwss   1/1     Running             0          6s
nginx-deployment-6c4849c57d-djvjn   1/1     Running             0          6s

## Estratégia de deployment

1. Rolling Update
- Nessa estratégia você define quantos pods você vai recriar por vez e quantos pod a mais vai ter além do definida em Replica.



``` bash
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 3
```
``` bash
allan@ALLAN-Z:~/Descomplicando_Kubernetes/dia-3$ k get pods -n strigus 
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-54789fb4df-4wlls   0/1     ContainerCreating   0          1s
nginx-deployment-54789fb4df-522d2   0/1     ContainerCreating   0          1s
nginx-deployment-54789fb4df-8mh8w   0/1     ContainerCreating   0          3s
nginx-deployment-54789fb4df-8whdd   1/1     Running             0          3s
nginx-deployment-54789fb4df-drjdf   1/1     Running             0          6s
nginx-deployment-54789fb4df-g7g6h   1/1     Running             0          6s
nginx-deployment-54789fb4df-mfdwg   1/1     Running             0          6s
nginx-deployment-54789fb4df-mvwx8   1/1     Running             0          4s
nginx-deployment-54789fb4df-s7s7z   1/1     Running             0          4s
nginx-deployment-54789fb4df-vc697   0/1     ContainerCreating   0          1s
nginx-deployment-54789fb4df-xnfkv   1/1     Running             0          6s
nginx-deployment-6c4849c57d-bswj5   1/1     Running             0          26m
nginx-deployment-6c4849c57d-djvjn   1/1     Running             0          26m
nginx-deployment-6c4849c57d-dzvsz   1/1     Running             0          26m
nginx-deployment-6c4849c57d-knkqc   0/1     Terminating         0          26m
nginx-deployment-6c4849c57d-mjdwn   1/1     Running             0          26m
nginx-deployment-6c4849c57d-mlltx   1/1     Terminating         0          26m
nginx-deployment-6c4849c57d-mrlvl   0/1     Terminating         0          26m
nginx-deployment-6c4849c57d-rwgqx   1/1     Running             0          27m

```

2. Recreate
- Recria todos os pods em uma tacada só.

``` bash
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: Recreate
```
``` bash
allan@ALLAN-Z:~/Descomplicando_Kubernetes/dia-3$ k get pods
NAME                                 READY   STATUS              RESTARTS   AGE
nginx-deployment-6c4849c57d-4zqgj    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-65jvb    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-7nmb5    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-glzk7    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-jrt5d    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-lvwkf    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-mzbhg    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-njb8b    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-whqmp    0/1     ContainerCreating   0          1s
nginx-deployment-6c4849c57d-zmm6q    0/1     ContainerCreating   0          1s

```