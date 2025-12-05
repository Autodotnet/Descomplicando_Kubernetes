# Kubernetes - Comandos Essenciais (Dia 1)

## 📋 Verificação de Status

### Criar cluster Kind
```bash
kind create cluster --config cluster.yaml
```

### Comandos básicos de status
```bash
# Verificar nodes do cluster
kubectl get nodes

# Listar pods
kubectl get pods
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o wide

# Verificar namespaces
kubectl get namespace

# Verificar deployments e replicasets
kubectl get deployment
kubectl get replicaset

# Verificar serviços
kubectl get service -n kube-system

# Acessar pod interativamente
kubectl exec -ti giropops -- bash
```

## ⚙️ Configuração do Bash Completion

```bash
# Instalar bash-completion
apt install bash-completion

# Verificar opções de completion
kubectl completion --help

# Configurar completion para kubectl
kubectl completion bash > ~/.kube/completion.bash.inc
echo "source '$HOME/.kube/completion.bash.inc'" >> $HOME/.bash_profile

# Editar e aplicar configurações
vim /root/.bash_profile
source /root/.bash_profile
```

## 🚀 Criando Pods

### Comandos kubectl run
```bash
# Criar pod com namespace específico
kubectl run --image nginx --port 80 -n giropops

# Criar pod com nome específico
kubectl run --image nginx --port 80 --name giropops

# Criar pod simples
kubectl run giropops --image nginx --port 80
```

### Dry Run - Gerar YAML
```bash
# Gerar arquivo YAML sem criar o recurso
kubectl run giropops --image nginx --port 80 --dry-run=client -o yaml > pod.yaml

# Aplicar o arquivo YAML
kubectl apply -f pod.yaml
```

## 🔧 Configuração de Cluster

### Criar cluster multi-node com Kind
```bash
kind create cluster --config kind-multinode-cluster.yaml
```

### Verificar instalação do Kubernetes
```bash
kubectl version
```

## 📝 Aliases Úteis

```bash
# Adicionar ao ~/.bashrc ou ~/.bash_profile
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgn='kubectl get nodes'
```
### Informações detalhadas dos nodes
```bash
kubectl get nodes -o wide
```

### Informações dos namespaces disponíveis
```bash
kubectl get namespaces
```
Namespace é uma forma de dividir os recursos do cluster em grupos virtuais, permitindo isolamento lógico entre os apps.

### Criar namespace
```bash
kubectl create namespace k8s-demo
```
### Criar pod
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: k8s-demo
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80

```

### Verificando logs do pod
``` bash
kubectl logs nginx-pod -n k8s-demo

```

### Joga saída do comando get pods para yaml 'output'
``` bash
kubectl get pods -o yaml
kubectl delete pods <name>

```
## Busybox
``` bash
kubectl run -ti girus --image busybox