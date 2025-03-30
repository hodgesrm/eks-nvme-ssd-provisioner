# eks-nvme-ssd-provisioner

The eks-nvme-ssd-provisioner will format and mount NVMe SSD disks on EKS
nodes. This is needed to make the sig-storage-local-static-provisioner work
well with EKS clusters. The eks-nvme-ssd-provisioner will create a raid0 device
if multiple NVMe SSD disks are found.

The resources in `manifests` expect the following node selector

```
aws.amazon.com/eks-local-ssd: "true"
```

Therefore you must make sure to set that label on all nodes that you want to
use with the eks-nvme-ssd-provisioner and sig-storage-local-static-provisioner.

## Build 

```
git clone git@github.com:hodgesrm/eks-nvme-ssd-provisioner.git
cd eks-nvme-ssd-provisioner
# Intel-only build: 
docker build . -t eks-nvme-ssd-provisioner
# Cross-platform build:
docker buildx build . -t eks-nvme-ssd-provisioner --platform linux/amd64,linux/arm64
```

We need to enable multi-platform builds for convenience, since we would
like this controller to run on both Intel and ARM (e.g., Graviton). See
[here](https://docs.docker.com/engine/storage/containerd/) for how to
set up on the standalone Docker engine. You also need to 
[install QEMU manually](https://docs.docker.com/build/building/multi-platform/#install-qemu-manually).

## Install

Install the DaemonSet by applying the following resource
```
kubectl apply -f manifests/eks-nvme-ssd-provisioner.yaml
```

Optionally you can also apply a pre-configed local-storage-provisioner that
plays well with the eks-nvme-ssd-provisioner
```
kubectl apply -f manifests/storage-local-static-provisioner.yaml
```

## Helm

```
helm upgrade --install --namespace=kube-system eks-nvme-ssd-provisioner ./helm/eks-nvme-ssd-provisioner
```

## Relation to sig-storage-local-static-provisioner
 - eks-nvme-ssd-provisioner creates disks from block storage
 - sig-storage-local-static-provisioner creates PersistentVolumens from disks 

In most cases you want both, if you have a another way to setup you disks jump directly to
[sig-storage-local-static-provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner)
