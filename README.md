# KinD demo

## OS

Oracle Linux 7.7+

## Demo

pre check

```
# go version
# docker info
# kubectl version
```

Install KinD

```
# GO111MODULE="on" go get sigs.k8s.io/kind@v0.6.0
# which kind
# kind version
```

Create cluster

```
# LANG=C kind create cluster
```

check cluster

```
# kubectl get nodes
# kubectl cluster-info --context kind-kind
```

# Cluster API

install Cluster API

```
# kubectl create -f https://github.com/kubernetes-sigs/cluster-api/releases/download/v0.2.9/cluster-api-components.yaml
```

install Bootstrap providor

```
# kubectl create -f https://github.com/kubernetes-sigs/cluster-api-bootstrap-provider-kubeadm/releases/download/v0.1.5/bootstrap-components.yaml
```

Upload vCenter credentials as a Kubernetes secret

```
WIP
```

# clustercli

WIP


# Reference

https://kind.sigs.k8s.io/docs/user/quick-start/


