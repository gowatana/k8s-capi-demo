# memo

## OS

Oracle Linux 7.7+

## Demo

base setup

```
$ vi hosts
$ ansible all -m ping
$ ansible-playbook setup.yml
```

pre check

```
$ ssh root@10.0.3.135
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

# Test KinD

Create cluster

```
# kind create cluster
```

check cluster

```
# kubectl get nodes
# kubectl cluster-info --context kind-kind
```

# SKIP: Cluster API

install Cluster API  
cluster-api-components

```
[root@k8s-f-01-01 ~]# kubectl create -f https://github.com/kubernetes-sigs/cluster-api/releases/download/v0.2.9/cluster-api-components.yaml
namespace/capi-system created
customresourcedefinition.apiextensions.k8s.io/clusters.cluster.x-k8s.io created
customresourcedefinition.apiextensions.k8s.io/machinedeployments.cluster.x-k8s.io created
customresourcedefinition.apiextensions.k8s.io/machines.cluster.x-k8s.io created
customresourcedefinition.apiextensions.k8s.io/machinesets.cluster.x-k8s.io created
role.rbac.authorization.k8s.io/capi-leader-election-role created
clusterrole.rbac.authorization.k8s.io/capi-manager-role created
rolebinding.rbac.authorization.k8s.io/capi-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/capi-manager-rolebinding created
deployment.apps/capi-controller-manager created
```

install Bootstrap providor

```
[root@k8s-f-01-01 ~]# kubectl create -f https://github.com/kubernetes-sigs/cluster-api-bootstrap-provider-kubeadm/releases/download/v0.1.5/bootstrap-components.yaml
namespace/cabpk-system created
customresourcedefinition.apiextensions.k8s.io/kubeadmconfigs.bootstrap.cluster.x-k8s.io created
customresourcedefinition.apiextensions.k8s.io/kubeadmconfigtemplates.bootstrap.cluster.x-k8s.io created
role.rbac.authorization.k8s.io/cabpk-leader-election-role created
clusterrole.rbac.authorization.k8s.io/cabpk-manager-role created
clusterrole.rbac.authorization.k8s.io/cabpk-proxy-role created
rolebinding.rbac.authorization.k8s.io/cabpk-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/cabpk-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/cabpk-proxy-rolebinding created
service/cabpk-controller-manager-metrics-service created
deployment.apps/cabpk-controller-manager created
```

base64

```
[root@k8s-f-01-01 ~]# echo administrator@vsphere.local | base64
YWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2FsCg==
[root@k8s-f-01-01 ~]# echo VMware1! | base64
Vk13YXJlMSEK

[gowatana@infra-jbox-01 ~]$ echo -n 'administrator@vsphere.local' | base64
YWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2Fs
[gowatana@infra-jbox-01 ~]$ echo -n 'VMware1!' | base64
Vk13YXJlMSE=
```

Upload vCenter credentials as a Kubernetes secret  

```
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  labels:
    control-plane: controller-manager
  name: capv-system
---
apiVersion: v1
kind: Secret
metadata:
  name: capv-manager-bootstrap-credentials
  namespace: capv-system
type: Opaque
data:
  username: "YWRtaW5pc3RyYXRvckB2c3BoZXJlLmxvY2Fs"
  password: "Vk13YXJlMSE="
EOF
```
 
infrastructure-components

```
[root@k8s-f-01-01 ~]# kubectl apply -f https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/releases/download/v0.5.4/infrastructure-components.yaml
namespace/capv-system unchanged
customresourcedefinition.apiextensions.k8s.io/vsphereclusters.infrastructure.cluster.x-k8s.io configured
customresourcedefinition.apiextensions.k8s.io/vspheremachines.infrastructure.cluster.x-k8s.io configured
customresourcedefinition.apiextensions.k8s.io/vspheremachinetemplates.infrastructure.cluster.x-k8s.io configured
role.rbac.authorization.k8s.io/capv-leader-election-role unchanged
clusterrole.rbac.authorization.k8s.io/capv-manager-role configured
clusterrole.rbac.authorization.k8s.io/capv-proxy-role unchanged
rolebinding.rbac.authorization.k8s.io/capv-leader-election-rolebinding unchanged
clusterrolebinding.rbac.authorization.k8s.io/capv-manager-rolebinding unchanged
clusterrolebinding.rbac.authorization.k8s.io/capv-proxy-rolebinding unchanged
service/capv-controller-manager-metrics-service unchanged
deployment.apps/capv-controller-manager unchanged
```

EOF

