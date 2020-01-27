Cluster API & vSphere デモ

## 想定 OS

Oracle Linux 7.7+

# 1. Ansible ホスト: 前提環境のセットアップ

Ansible でのセットアップ。  
KinD ホストで実行する場合は、Playbook の OS 再起動で一度切断される。

```
$ vi hosts
$ ansible all -m ping
$ ansible-playbook setup.yml
```

# 2. KinD ホスト: 確認と kind / clusterctl インストール

事前チェック。

```
$ ssh root@10.0.3.135
# go version
# docker info
# kubectl version
```

KinD のインストール。

```
# GO111MODULE="on" go get sigs.k8s.io/kind@v0.6.0
# which kind
# kind version
```

clusterctl のインストール。

```
# curl -L -o ./bin/clusterctl https://github.com/kubernetes-sigs/cluster-api/releases/download/v0.2.9/clusterctl-linux-amd64
# chmod +x ./bin/clusterctl
# which clusterctl
/root/bin/clusterctl
```

# 3. vCenter での OVA デプロイ

k8s クラスタを作成する vSphere 環境に、専用の OVA をデプロイしておく。

ダウンロード リンク集  
https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/release-0.5/README.md#kubernetes-versions-with-published-ovas

デモで利用する OVA  
http://storage.googleapis.com/capv-images/release/v1.16.3/centos-7-kube-v1.16.3.ova

# 4. KinD ホスト: Management-Cluster

create management cluster config from envvars.txt

```
# docker run --rm \
-v "$(pwd)":/out \
-v "$(pwd)/envvars.txt":/envvars.txt:ro \
gcr.io/cluster-api-provider-vsphere/release/manifests:v0.5.4 \
-c management-cluster
```

ex.

```
[root@k8s-f-11 ~]# docker run --rm \
> -v "$(pwd)":/out \
> -v "$(pwd)/envvars.txt":/envvars.txt:ro \
> gcr.io/cluster-api-provider-vsphere/release/manifests:v0.5.4 \
> -c management-cluster
Unable to find image 'gcr.io/cluster-api-provider-vsphere/release/manifests:v0.5.4' locally
Trying to pull repository gcr.io/cluster-api-provider-vsphere/release/manifests ...
v0.5.4: Pulling from gcr.io/cluster-api-provider-vsphere/release/manifests
fc7181108d40: Pull complete
869df5e15cd7: Pull complete
f9485c4e0620: Pull complete
1d959594fb73: Pull complete
ecbcb5c3ee03: Pull complete
d54144cee33b: Pull complete
67e982c8e4c9: Pull complete
5d03a15ed239: Pull complete
fb52fed35322: Pull complete
Digest: sha256:61254ceb3f36ff11699df82f3304504801942cdcead073ec55d391c35f5d5abe
Status: Downloaded newer image for gcr.io/cluster-api-provider-vsphere/release/manifests:v0.5.4
Checking 192.168.1.30 for vSphere version
Detected vSphere version 6.7.3
Generated ./out/management-cluster/addons.yaml
Generated ./out/management-cluster/cluster.yaml
Generated ./out/management-cluster/controlplane.yaml
Generated ./out/management-cluster/machinedeployment.yaml
Generated /build/examples/default/provider-components/provider-components-cluster-api.yaml
Generated /build/examples/default/provider-components/provider-components-kubeadm.yaml
Generated /build/examples/default/provider-components/provider-components-vsphere.yaml
Generated ./out/management-cluster/provider-components.yaml
WARNING: ./out/management-cluster/provider-components.yaml includes vSphere credentials
```

create cluster  
-v, --v Level number for the log level verbosity (ex. 6)

```
$ clusterctl create cluster \
--bootstrap-type kind \
--bootstrap-flags name=management-cluster \
--cluster ./out/management-cluster/cluster.yaml \
--machines ./out/management-cluster/controlplane.yaml \
--provider-components ./out/management-cluster/provider-components.yaml \
--addon-components ./out/management-cluster/addons.yaml \
--kubeconfig-out ./out/management-cluster/kubeconfig
```

ex.

```
[root@k8s-f-11 ~]# clusterctl create cluster \
>   --bootstrap-type kind \
>   --bootstrap-flags name=management-cluster \
>   --cluster ./out/management-cluster/cluster.yaml \
>   --machines ./out/management-cluster/controlplane.yaml \
>   --provider-components ./out/management-cluster/provider-components.yaml \
>   --addon-components ./out/management-cluster/addons.yaml \
>   --kubeconfig-out ./out/management-cluster/kubeconfig
NOTICE: clusterctl has been deprecated in v1alpha2 and will be removed in a future version.
I0115 01:45:24.785664   18380 createbootstrapcluster.go:27] Preparing bootstrap cluster
I0115 01:46:36.326118   18380 clusterdeployer.go:82] Applying Cluster API stack to bootstrap cluster
I0115 01:46:36.326150   18380 applyclusterapicomponents.go:26] Applying Cluster API Provider Components
I0115 01:46:39.349951   18380 clusterdeployer.go:87] Provisioning target cluster via bootstrap cluster
I0115 01:46:39.362470   18380 applycluster.go:42] Creating Cluster referenced object "infrastructure.cluster.x-k8s.io/v1alpha2, Kind=VSphereCluster" with name "management-cluster" in namespace "default"
I0115 01:46:39.431714   18380 applycluster.go:48] Creating cluster object management-cluster in namespace "default"
I0115 01:46:39.443363   18380 clusterdeployer.go:96] Creating control plane machine "management-cluster-controlplane-0" in namespace "default"
I0115 01:46:39.449877   18380 applymachines.go:40] Creating Machine referenced object "infrastructure.cluster.x-k8s.io/v1alpha2, Kind=VSphereMachine" with name "management-cluster-controlplane-0" in namespace "default"
I0115 01:46:39.521449   18380 applymachines.go:40] Creating Machine referenced object "bootstrap.cluster.x-k8s.io/v1alpha2, Kind=KubeadmConfig" with name "management-cluster-controlplane-0" in namespace "default"
I0115 01:46:39.579613   18380 applymachines.go:46] Creating machines in namespace "default"
I0115 01:49:29.648163   18380 clusterdeployer.go:105] Creating target cluster
I0115 01:49:29.680259   18380 applyaddons.go:25] Applying Addons
I0115 01:49:30.650032   18380 clusterdeployer.go:123] Pivoting Cluster API stack to target cluster
I0115 01:49:30.650103   18380 pivot.go:76] Applying Cluster API Provider Components to Target Cluster
I0115 01:49:32.360220   18380 pivot.go:81] Pivoting Cluster API objects from bootstrap to target cluster.
I0115 01:50:13.825221   18380 clusterdeployer.go:128] Saving provider components to the target cluster
I0115 01:50:13.886393   18380 clusterdeployer.go:150] Creating node machines in target cluster.
I0115 01:50:13.893621   18380 applymachines.go:46] Creating machines in namespace "default"
I0115 01:50:13.893668   18380 clusterdeployer.go:164] Done provisioning cluster. You can now access your cluster with kubectl --kubeconfig ./out/management-cluster/kubeconfig
I0115 01:50:13.894783   18380 createbootstrapcluster.go:36] Cleaning up bootstrap cluster.
```

WIP: 失敗時の management-cluster 削除

```
# kind get kubeconfig --name=management-cluster > ./out/management-cluster/kubeconfig
# clusterctl delete cluster --bootstrap-type kind --kubeconfig=./out/management-cluster/kubeconfig --provider-components ./out/management-cluster/provider-components.yaml
# clusterctl delete cluster --bootstrap-type kind --kubeconfig out/management-cluster/kubeconfig -p out/management-cluster/provider-components.yaml -n default -v 5
```

get cluster info

```
[root@k8s-f-11 ~]# export KUBECONFIG="$(pwd)/out/management-cluster/kubeconfig"
[root@k8s-f-11 ~]# kubectl get nodes
NAME                                STATUS   ROLES    AGE     VERSION
management-cluster-controlplane-0   Ready    master   6m32s   v1.16.3
```

## 4. KinD ホスト: Workload-cluster

export kubeconfig

```
[root@k8s-f-11 ~]# export KUBECONFIG=$(pwd)/out/management-cluster/kubeconfig
```

create config yaml

```
# docker run --rm \
-v "$(pwd)":/out \
-v "$(pwd)/envvars.txt":/envvars.txt:ro \
gcr.io/cluster-api-provider-vsphere/release/manifests:v0.5.4 \
-c workload-cluster-1
```

ex.

```
[root@k8s-c01-f-01 ~]# docker run --rm \
> -v "$(pwd)":/out \
> -v "$(pwd)/envvars.txt":/envvars.txt:ro \
> gcr.io/cluster-api-provider-vsphere/release/manifests:v0.5.4 \
> -c workload-cluster-1
Checking 192.168.1.30 for vSphere version
Detected vSphere version 6.7.3
Generated ./out/workload-cluster-1/addons.yaml
Generated ./out/workload-cluster-1/cluster.yaml
Generated ./out/workload-cluster-1/controlplane.yaml
Generated ./out/workload-cluster-1/machinedeployment.yaml
Generated /build/examples/default/provider-components/provider-components-cluster-api.yaml
Generated /build/examples/default/provider-components/provider-components-kubeadm.yaml
Generated /build/examples/default/provider-components/provider-components-vsphere.yaml
Generated ./out/workload-cluster-1/provider-components.yaml
WARNING: ./out/workload-cluster-1/provider-components.yaml includes vSphere credentials
```

Create workload-cluster

```
[root@k8s-f-11 ~]# kubectl apply -f ./out/workload-cluster-1/cluster.yaml
cluster.cluster.x-k8s.io/workload-cluster-1 created
vspherecluster.infrastructure.cluster.x-k8s.io/workload-cluster-1 created
```

control plane

```
[root@k8s-f-11 ~]# kubectl apply -f ./out/workload-cluster-1/controlplane.yaml
kubeadmconfig.bootstrap.cluster.x-k8s.io/workload-cluster-1-controlplane-0 created
machine.cluster.x-k8s.io/workload-cluster-1-controlplane-0 created
vspheremachine.infrastructure.cluster.x-k8s.io/workload-cluster-1-controlplane-0 created
```

machine deployment

```
[root@k8s-f-11 ~]# kubectl apply -f ./out/workload-cluster-1/machinedeployment.yaml
kubeadmconfigtemplate.bootstrap.cluster.x-k8s.io/workload-cluster-1-md-0 created
machinedeployment.cluster.x-k8s.io/workload-cluster-1-md-0 created
vspheremachinetemplate.infrastructure.cluster.x-k8s.io/workload-cluster-1-md-0 created
```

get workload-cluster-1/kubeconfig

```
[root@k8s-f-11 ~]# kubectl get secret workload-cluster-1-kubeconfig -o=jsonpath='{.data.value}' | { base64 -d 2>/dev/null || base64 -D; } > ./out/workload-cluster-1/kubeconfig
```

# use workload-cluster

test workload-cluster

```
[root@k8s-f-11 ~]# export KUBECONFIG=./out/workload-cluster-1/kubeconfig
[root@k8s-f-11 ~]# kubectl get nodes
NAME                                       STATUS     ROLES    AGE     VERSION
workload-cluster-1-controlplane-0          NotReady   master   6m59s   v1.16.3
workload-cluster-1-md-0-78469c8cf9-5kl9b   NotReady   <none>   5m45s   v1.16.3
```

setup network add-on: antrea

```
# kubectl apply -f https://github.com/vmware-tanzu/antrea/releases/download/v0.2.0/antrea.yml
```

antrea is "Running" status

```
[root@k8s-f-03-01 ~]# kubectl get pods -n kube-system
NAME                                                        READY   STATUS    RESTARTS   AGE
antrea-agent-dmrdf                                          2/2     Running   0          2m2s
antrea-agent-gl9rp                                          2/2     Running   0          2m2s
antrea-controller-54fcbd7f67-hhcnf                          1/1     Running   0          2m2s
coredns-5644d7b6d9-k7xjj                                    1/1     Running   0          12h
coredns-5644d7b6d9-zw77h                                    1/1     Running   0          12h
etcd-workload-cluster-1-controlplane-0                      1/1     Running   0          12h
kube-apiserver-workload-cluster-1-controlplane-0            1/1     Running   0          12h
kube-controller-manager-workload-cluster-1-controlplane-0   1/1     Running   0          12h
kube-proxy-c7g66                                            1/1     Running   0          12h
kube-proxy-zbwsb                                            1/1     Running   0          12h
kube-scheduler-workload-cluster-1-controlplane-0            1/1     Running   0          12h
vsphere-cloud-controller-manager-ptx26                      1/1     Running   0          12h
vsphere-csi-controller-0                                    5/5     Running   0          12h
vsphere-csi-node-2wcst                                      3/3     Running   0          81s
vsphere-csi-node-6955j                                      3/3     Running   0          85s
```

# Reference

https://kind.sigs.k8s.io/docs/user/quick-start/


