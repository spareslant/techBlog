---
title: "Kubernetes Cluster Using CRI-O"
date: 2021-02-07T22:18:02Z
draft: true
tags: ["k8s", "kubernetes", "vagrant", "vagrantfile", "cri-o", "virtualbox", "cgroups"]
---

## Introduction
* We will be creating a 2 node kubernetes cluster using vagrant and virtualbox.
* We will be using `cri-o` rather than `Docker` to configure the cluster.
* `CRI-O` uses the systemd `cgroup` driver per default. 
* We will be creating config files `init-defaults.yml` and `join-defaults.yml` to be used by `kubeadm` to configure control plane and worker rather than using long command line options. 
* In the config files, we shall specify the `cgroup` driver for `kubelet`.

## Create VMs
It is assumed that you already have `VirtualBox` and `Vagrant` installed in your machine.

### Vagrantfile
Run following commands to create Vagrantfile.
```bash
mkdir VAGRANT
cd VAGRANT
cat << EOF >> Vagrantfile
all_hosts = [
    {
        vagrant_hostname: "master",
        full_hostname: "master.virtual.machine",
        vmbox: "ubuntu/bionic64",
        #vmbox_version: "31.20191023.0",
        ip: "10.0.0.10",
        memory: 4096,
        cpus: 3
    },
    {
        vagrant_hostname: "worker1",
        full_hostname: "worker1.virtual.machine",
        vmbox: "ubuntu/bionic64",
        #vmbox_version: "31.20191023.0",
        ip: "10.0.0.12",
        memory: 2048,
        cpus: 1
    }
]

# individual machine names must be mentioned is below command line in
# order to bring machines. (due to autostart: false)
# vagrant up master worker1
Vagrant.configure("2") do |config|

    all_hosts.each do |host|
        config.vm.define host[:vagrant_hostname], autostart: false do |this_host|
            this_host.vm.network :private_network, ip: host[:ip]
            this_host.vm.hostname = host[:full_hostname]
            this_host.vm.box = host[:vmbox]
            this_host.vm.box_version = host[:vmbox_version]

            this_host.vm.provider "virtualbox" do |m|
                m.memory = host[:memory]
                m.cpus = host[:cpus]
            end
        end
    end
end
EOF
```

### Bring up the VMs
```bash
vagrant up master worker1
```

## Setup Master node as Kubernetes control plane
Login to master node.

```bash
vagrant ssh master
```

Optionally set `.vimrc` file with following content in `master` node.
```bash
cat <<EOF > ~/.vimrc
set nocompatible
set sw=2
set ts=2
set softtabstop=2
set backspace=eol,indent,start
set autoindent
set expandtab
set hlsearch
set incsearch
syntax on
filetype on
filetype plugin on
filetype indent on
filetype plugin indent on
EOF
```

### Fix the `/etc/hosts` file
In `/etc/hosts` file, make sure `master.virtual.machine` and `master` are pointing to `10.0.0.10`.
```bash
10.0.0.10 master.virtual.machine master
```
Note: `10.0.0.10` is defined in `vagrantfile`.

`Vagrantfile` create two network interfaces for the virtual machines and hence two networks are available (Private and NAT). One network interface which is `private` will have the IP `10.0.0.10`. The other network interface will be for NAT to have internet connectivity. IP to this network interface is assigned by virtualbox dynamically. 

The two VMs (`master` and `worker`) can communicate with each other on their private IPs only. Their NAT IPs will not be able ping each other. Therefore while configuring kubernetes control plane, we will have to explicity mention the private IP of the master in options.

`master` node network interfaces:

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:4d:8f:d1:c7:05 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 81105sec preferred_lft 81105sec
    inet6 fe80::4d:8fff:fed1:c705/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1b:d0:a5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.10/24 brd 10.0.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe1b:d0a5/64 scope link
       valid_lft forever preferred_lft forever
```

`worker` network interfaces:
```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:4d:8f:d1:c7:05 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 80357sec preferred_lft 80357sec
    inet6 fe80::4d:8fff:fed1:c705/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:64:89:e5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.12/24 brd 10.0.0.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe64:89e5/64 scope link
       valid_lft forever preferred_lft forever
```

**Note**: NAT interfaces (`enp0s3`) in both `master` and `worker1` nodes have same IP (10.0.2.15), but private inetrfaces (`enp0s8`) IPs are being assigned using the config defined in `Vagrantfile`.

### Install `cri-o` container runtime
Instructions for this page https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o were followed to install the container runtime

```bash
# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

Set variables according to the OS distribution
```bash
export OS=xUbuntu_18.04
export VERSION=1.20
```

```bash
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers-cri-o.gpg add -

sudo apt-get update
sudo apt-get install cri-o cri-o-runc
```

```bash
sudo systemctl status crio
```

```bash
$ sudo crio-status info
cgroup driver: systemd
storage driver: overlay
storage root: /var/lib/containers/storage
default GID mappings (format <container>:<host>:<size>):
  0:0:4294967295
default UID mappings (format <container>:<host>:<size>):
  0:0:4294967295
```
### Install Kube binaries.
Steps were followed from this link https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

```bash
sudo systemctl daemon-reload
sudo systemctl start crio
```

### Generate config file for control plane configuration using kubeadm
A helpful link: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

generate initial dummy configuration file.
```bash
sudo kubeadm config print init-defaults > init-defaults.yml
```
Make following modifications and some additions to the `init-defaults.yml` file

```diff
apiVersion: kubeadm.k8s.io/v1beta2
 bootstrapTokens:
 - groups:
-  - system:bootstrappers:kubeadm:default-node-token
-  token: abcdef.0123456789abcdef
-  ttl: 24h0m0s
   usages:
   - signing
   - authentication
 kind: InitConfiguration
 localAPIEndpoint:
-  advertiseAddress: 1.2.3.4
+  advertiseAddress: 10.0.0.10
   bindPort: 6443
 nodeRegistration:
-  criSocket: /var/run/dockershim.sock
+  criSocket: /var/run/crio/crio.sock
   name: master
   taints:
   - effect: NoSchedule
@@ -35,4 +32,9 @@
 networking:
   dnsDomain: cluster.local
   serviceSubnet: 10.96.0.0/12
+  podSubnet: 172.168.18.0/16
 scheduler: {}
+---
+apiVersion: kubelet.config.k8s.io/v1beta1
+kind: KubeletConfiguration
+cgroupDriver: systemd
```
Note the `+` and `-` in the very first column above.
   * leading `+` => lined added. 
   * leading `-` => lines removed

final `init-defaults.yml` will look below:
```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.0.0.10
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/crio/crio.sock
  name: master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.20.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 172.168.18.0/16
scheduler: {}
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
```

More options for `kubeadm init` configuration can be found here
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#pkg-types
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#InitConfiguration
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#ClusterConfiguration
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#Networking

More options for `kubelet` configuration can be found here
* https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/
* https://pkg.go.dev/k8s.io/kubelet/config/v1beta1#KubeletConfiguration

### Configure control plane using `init-defaults.yml`

Run following command
```bash
sudo kubeadm init --config=init-defaults.yml --node-name=master
```

following was the output
```bash
[init] Using Kubernetes version: v1.20.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master] and IPs [10.96.0.1 10.0.0.10]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost master] and IPs [10.0.0.10 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost master] and IPs [10.0.0.10 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 18.004566 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: iqcz6s.23d1vgw7vdm3k4cv
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.10:6443 --token iqcz6s.23d1vgw7vdm3k4cv \
    --discovery-token-ca-cert-hash sha256:bc1e33333b01e57b1f927fc5913668577cde464e08f833afe99218b6a43da306
```

### Configure local unix user to run kubectl commands
Run the following commands as mentioned in above output.
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Enable kubectl auto-complete (Optional)
```bash
source <(kubectl completion bash)
```
### Check various components in control plane
```bash
vagrant@master:~$ kubectl get nodes -o wide
NAME     STATUS     ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master   NotReady   control-plane,master   90s   v1.20.2   10.0.0.10     <none>        Ubuntu 18.04.5 LTS   4.15.0-130-generic   cri-o://1.20.0
```
**Note:** `master` node status in above output is `NotReady`

```bash
vagrant@master:~$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE    IP          NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-74ff55c5b-4gntg          0/1     Pending   0          111s   <none>      <none>   <none>           <none>
kube-system   coredns-74ff55c5b-xhrrl          0/1     Pending   0          111s   <none>      <none>   <none>           <none>
kube-system   etcd-master                      1/1     Running   0          2m     10.0.0.10   master   <none>           <none>
kube-system   kube-apiserver-master            1/1     Running   0          2m     10.0.0.10   master   <none>           <none>
kube-system   kube-controller-manager-master   1/1     Running   0          2m     10.0.0.10   master   <none>           <none>
kube-system   kube-proxy-cp58h                 1/1     Running   0          112s   10.0.0.10   master   <none>           <none>
kube-system   kube-scheduler-master            1/1     Running   0          2m     10.0.0.10   master   <none>           <none>
```
**Note:** `coredns` pods in above output is `pending`

### Apply `calico` networking
```bash
$ kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Following was the output
```
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
poddisruptionbudget.policy/calico-kube-controllers created
```
### Re-check node and pod status now
```bash
vagrant@master:~$ kubectl get nodes -o wide
NAME     STATUS   ROLES                  AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master   Ready    control-plane,master   3m55s   v1.20.2   10.0.0.10     <none>        Ubuntu 18.04.5 LTS   4.15.0-130-generic   cri-o://1.20.0
```
**Note:** `master` node status is `Ready` now

```bash
vagrant@master:~$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-86bddfcff-tt7gq   1/1     Running   0          99s     172.168.219.67   master   <none>           <none>
kube-system   calico-node-72zf5                         1/1     Running   0          99s     10.0.0.10        master   <none>           <none>
kube-system   coredns-74ff55c5b-4gntg                   1/1     Running   0          4m29s   172.168.219.66   master   <none>           <none>
kube-system   coredns-74ff55c5b-xhrrl                   1/1     Running   0          4m29s   172.168.219.65   master   <none>           <none>
kube-system   etcd-master                               1/1     Running   0          4m38s   10.0.0.10        master   <none>           <none>
kube-system   kube-apiserver-master                     1/1     Running   0          4m38s   10.0.0.10        master   <none>           <none>
kube-system   kube-controller-manager-master            1/1     Running   0          4m38s   10.0.0.10        master   <none>           <none>
kube-system   kube-proxy-cp58h                          1/1     Running   0          4m30s   10.0.0.10        master   <none>           <none>
kube-system   kube-scheduler-master                     1/1     Running   0          4m38s   10.0.0.10        master   <none>           <none>
```
**Note:** `coredns` pods status is `Running` now.

### Prepare config file worker nodes
We can use the token generated by `kubeadm init` command above. However we shall create a new one.

```bash
vagrant@master:~$ sudo kubeadm token create --print-join-command
kubeadm join 10.0.0.10:6443 --token wk27p3.dwyr6hbg17pcncso     --discovery-token-ca-cert-hash sha256:bc1e33333b01e57b1f927fc5913668577cde464e08f833afe99218b6a43da306
```

Take a note of `token` and `discovery-token-ca-cert-hash` in abobe output. We shall use that shortly in creating `join-defaults.yml` file.

Run the following command to generate the `join-default` configuration for workers.
```bash
sudo kubeadm config print join-defaults > join-defaults.yml
```

Make following modifications and additions to the `join-defaults.yml` file.
We are using `token` and `caCertHashes` from above `kubeadm token create` command output.
```diff
 apiVersion: kubeadm.k8s.io/v1beta2
 caCertPath: /etc/kubernetes/pki/ca.crt
 discovery:
   bootstrapToken:
-    apiServerEndpoint: kube-apiserver:6443
-    token: abcdef.0123456789abcdef
-    unsafeSkipCAVerification: true
+    apiServerEndpoint: 10.0.0.10:6443
+    token: wk27p3.dwyr6hbg17pcncso
+    caCertHashes:
+      - sha256:bc1e33333b01e57b1f927fc5913668577cde464e08f833afe99218b6a43da306
   timeout: 5m0s
-  tlsBootstrapToken: abcdef.0123456789abcdef
+  tlsBootstrapToken: wk27p3.dwyr6hbg17pcncso
 kind: JoinConfiguration
 nodeRegistration:
-  criSocket: /var/run/dockershim.sock
+  criSocket: /var/run/crio/crio.sock
   name: master
   taints: null
+---
+apiVersion: kubelet.config.k8s.io/v1beta1
+kind: KubeletConfiguration
+cgroupDriver: systemd
```

Note the `+` and `-` in the very first column above.
   * leading `+` => lined added. 
   * leading `-` => lines removed

final `join-defaults.yml` will look like below:
```yaml
apiVersion: kubeadm.k8s.io/v1beta2
caCertPath: /etc/kubernetes/pki/ca.crt
discovery:
  bootstrapToken:
    apiServerEndpoint: 10.0.0.10:6443
    token: wk27p3.dwyr6hbg17pcncso
    caCertHashes:
      - sha256:bc1e33333b01e57b1f927fc5913668577cde464e08f833afe99218b6a43da306
  timeout: 5m0s
  tlsBootstrapToken: wk27p3.dwyr6hbg17pcncso
kind: JoinConfiguration
nodeRegistration:
  criSocket: /var/run/crio/crio.sock
  name: master
  taints: null
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
```

More options for `kubeadm join` configuration can be found here
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#pkg-types
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2#JoinConfiguration
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2#Discovery
* https://pkg.go.dev/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2#BootstrapTokenDiscovery


## Setup worker node

Login to the `worker1` node
```bash
vagrant ssh worker1
```

## Fix the `/etc/hosts` file
In `/etc/hosts` file, make sure `worker1.virtual.machine` and `worker1` are pointing to `10.0.0.12`.
```bash
10.0.0.12 worker1.virtual.machine worker1
```
Note: `10.0.0.12` is defined in `vagrantfile`.

### Install `cri-o` container runtime
Instructions for this page https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o were followed to install the container runtim

```bash
# Create the .conf file to load the modules at bootup
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Set up required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system
```

Set variables according to the OS distribution
```bash
export OS=xUbuntu_18.04
export VERSION=1.20
```

```bash
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF

curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers-cri-o.gpg add -

sudo apt-get update
sudo apt-get install cri-o cri-o-runc
```

```bash
sudo systemctl daemon-reload
sudo systemctl start crio
``

```bash
sudo systemctl status crio
```

```bash
$ sudo crio-status info
cgroup driver: systemd
storage driver: overlay
storage root: /var/lib/containers/storage
default GID mappings (format <container>:<host>:<size>):
  0:0:4294967295
default UID mappings (format <container>:<host>:<size>):
  0:0:4294967295
```
### Install Kube binaries.
Steps were followed from this link https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Joining the `worker1` node
Copy the `join-defaults.yml` file created in `master` node to `worker1` node.

Run the following command in `worker1` node.
```bash
sudo kubeadm join --config=join-defaults.yml --node-name=worker1
```

Following output was produced
```bash
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

## Verify cluster installation

### Check nodes status on `master` (control plane) node
```bash
vagrant@master:~$ kubectl get nodes -o wide
NAME      STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master    Ready    control-plane,master   27m   v1.20.2   10.0.0.10     <none>        Ubuntu 18.04.5 LTS   4.15.0-130-generic   cri-o://1.20.0
worker1   Ready    <none>                 78s   v1.20.2   10.0.0.12     <none>        Ubuntu 18.04.5 LTS   4.15.0-130-generic   cri-o://1.20.0
```

### Create an `nginx` deployment
```bash
vagrant@master:~$ kubectl create deployment --image=nginx nginx --port=80
deployment.apps/nginx created

vagrant@master:~$ kubectl get deployments.apps
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           25s

vagrant@master:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP                NODE      NOMINATED NODE   READINESS GATES
nginx-7848d4b86f-mfvt5   1/1     Running   0          56s   172.168.235.129   worker1   <none>           <none>
```

### Create a service
```bash
vagrant@master:~$ kubectl expose deployment nginx --port=8080 --target-port=80
service/nginx exposed

vagrant@master:~$ kubectl get service -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE   SELECTOR
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    30m   <none>
nginx        ClusterIP   10.103.27.202   <none>        8080/TCP   31s   app=nginx
```

```bash
vagrant@master:~$ kubectl get ep -o wide
NAME         ENDPOINTS            AGE
kubernetes   10.0.0.10:6443       30m
nginx        172.168.235.129:80   61s
```

#### Check nginx service created above.
```bash
vagrant@master:~$ curl http://10.103.27.202:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```











