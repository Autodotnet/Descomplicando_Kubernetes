### Persistent Volume Claim

Usado para fazer uma solicitação para o uso do disco dentro do Pod.

``` bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    pvc: meu-pvc
  name: pvc-exemplo
spec:
  accessModes: 
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: giropops
  selector:
    matchLabels:
      storage: nfs
``` 

Após a criação ele fica em pending pois está esperando algo consumir o recurso, no caso adicionamos o claim no Pod.

``` bash
kubectl get pvc
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-exemplo   Pending                                      giropops       2m17s
```

### Adicionando o pvc no Pod

``` bash
piVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: meu-pvc
      mountPath: /usr/share/nginx/html
  volumes:
  - name: meu-pvc
    persistentVolumeClaim:
      claimName: meu-pvc

```