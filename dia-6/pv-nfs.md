### Criando NFS

mkdir /mnt/nfs
chown allanz /mnt/nfs

sudo apt-get install nfs-kernel-server nfs-common

sudo vim /etc/exports

/mnt/nfs *(rw,sync,no_root_squash,no_subtree_check)

### Criando yaml do PV com NFS

``` bash
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    storage: nfs
  name: meu-pv-nfs
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce # Disponível somente em 1 nó
  persistentVolumeReclaimPolicy: Retain
  nfs: 
    server: 192.168.0.58
    path: "/mnt/nfs"
  storageClassName: giropops
```