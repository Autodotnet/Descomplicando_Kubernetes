# Criando Cluster k8s com kubeadm usando VM's do KVM-Compose

O KVM compose é uma forma de gerenciar máquinas virtuais Kernel Virtual Machines (KVM) como se fossem containers, similiar ao Docker Compose. 

## Para fazer a instalação siga os passos no repositório do Paulo.

https://github.com/paulozagaloneves/kvm-compose 

Para usar o kvm você precisa configurar um interface br0 (bridge), como estou usando ZorinOS precisei usar o nmcli, ferramenta para configurar o Network Manager.

``` bash
$ apt-get install bridge-utils

### Configura interface bridge

$ sudo nmcli conn add type bridge con-name br0 ifname br0

### Adiciona interface Ethernet como bridge

$ sudo nmcli conn add type ethernet slave-type bridge con-name bridge-br0 ifname enp1s0 master br0

obs: lembre se substituir a interface depois de ifname pela que mostrar no seu Linux com o comando ifconfig

### Ative a nova conexão 

$ sudo nmcli conn up br0

### Em seguida, desative a interface ou conexão Ethernet.

$ sudo nmcli conn down Ethernet\ connection\ 1
OR
sudo nmcli conn down 525284a9-60d9-4396-a1c1-a37914d43eff

### Verifique se a interface Ethernet está como slave na br0 

$ sudo nmcli conn show --active

(vai aparecer em name 'bridge-br0' e na parte device algo como 'enp1s0')
```
---
# Configurando Kubeadm

Nesse cenário utilizei 3 máquinas Debian 13 com os recursos abaixo.

``` bash
== VMs disponíveis no cluster-k8s.yaml ===
--------------- --------------- ---------- ------ -------- ---------------- ---------------- ------------------
Nome            Distro          Memória    vCPUs  Disco    Username         IP               Status            
--------------- --------------- ---------- ------ -------- ---------------- ---------------- ------------------
k8s-cp-01       debian13        4096MB     4      20GB     debian           192.168.0.40     🟢 running         
k8s-wrk-01      debian13        2048MB     2      15GB     debian           192.168.0.41     🟢 running         
k8s-wrk-02      debian13        2048MB     2      15GB     debian           192.168.0.42     🟢 running         
``` 

## Carregando módulos do Kernel para rodar o Cluster

``` bash
swapoff -a

sudo apt update

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay \

sudo modprobe br_netfilter

cat <<EOF /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```
A saída do comando deve ser

```bash
 Applying /usr/lib/sysctl.d/10-coredump-debian.conf ...
* Applying /usr/lib/sysctl.d/50-default.conf ...
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
kernel.core_pattern = core
kernel.sysrq = 0x01b6
kernel.core_uses_pid = 1
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.enp1s0.rp_filter = 2
net.ipv4.conf.lo.rp_filter = 2
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.enp1s0.accept_source_route = 0
net.ipv4.conf.lo.accept_source_route = 0
net.ipv4.conf.default.promote_secondaries = 1
net.ipv4.conf.enp1s0.promote_secondaries = 1
net.ipv4.conf.lo.promote_secondaries = 1
net.ipv4.ping_group_range = 0 2147483647
net.core.default_qdisc = fq_codel
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_regular = 2
fs.protected_fifos = 1
vm.max_map_count = 1048576
kernel.pid_max = 4194304
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

```

## Instalando os pacotes do repositório do K8s

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

## Instalar Kubeadm, Kubelet e Kubectl

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

## Instalação Containerd

apt-get install gnupg lsb-release ca-certificates

sudo apt install containerd -y

containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd

sudo systemctl enable --now kubelet

```
---

## Iniciando o cluster e adicionando os nós no Cluster

Para iniciar o cluster com kubeadm precisaremos rodar 1 comando no node do Control Plane e outro comando nos nós restantes.

``` bash
# Iniciando o cluster a partir do Control Plane

sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=(ip da sua vm)

# Configurando os outros nós para se juntarem ao cluster

Quando você aplicar o comando acima ele vai mostrar na saída um comando similar a este abaixo para você possa aplicar nos outros nodes do Cluster sem ser o Control Plane.

sudo kubeadm join 192.168.0.40:6443 --token kyqqvs.tjfzg4ohufj9mx9n \
	--discovery-token-ca-cert-hash sha256:cb412f1cbaf4f150e8e836a94fdb911264188b48ab5036c1505800cea4266012

# Configurar kube config para utilizar o kubectl

Adicionar só no control plane
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

## Instalando CNI

Para habilitar a conexão entre os nodes do K8S é preciso instalar um plugin chamados CNI. Existem várias opções como Calico, Cillium e Fannel. 

Para instalar o Cillium use o comando

``` bash

cilium install --version 1.18.5
``` 

## Resultado

Os nodes ficaram prontos para serem usados.

``` bash
debian@k8s-cp-01:~$ kubectl get nodes
NAME         STATUS   ROLES           AGE     VERSION
k8s-cp-01    Ready    control-plane   23h     v1.34.3
k8s-wrk-01   Ready    <none>          22h     v1.34.3
k8s-wrk-02   Ready    <none>          3h26m   v1.34.3
```