---
title: "K8s Vagrant Centos Stream Crio Automated Deployment"
date: 2021-08-23T22:29:22+01:00
draft: true
tags: ["k8s", "kubernetes", "vagrant", "vagrantfile", "cri-o", "virtualbox"]
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
<style>
/* To highlight text in Green in pre tag */
.hl {color: #008A00;}
/* To highlight text in Bold Green in pre tag */
.hlb {color: #008A00; font-weight: bold;}
/* To highlight text in Bold Red in pre tag */
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {
    color:#7e168d; 
    background: #f0f0f0; 
    padding: 0.1em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>

## Introduction
* We will be creating a fully automated `Vagrant` based `Kubernetes` cluster deployment.
* Following are the software versions used:
  * Hardware: MacBook Pro
  * HostOS: MacOS Big Sur 11.5.2
  * Kube Version: 1.21.4
  * CRIO: 1.21
  * Cluster OS: Centos8 Stream
* It is assumed that VirtualBox and Vagrant is already installed on your hostOS.

## Vagrantfile
* Create a directory and create a file named `Vagrantfile` with following contents in that directory.
* cd to this directory 

```ruby
require 'json'

all_hosts = [
    {
        vagrant_hostname: "master",
        full_hostname: "master.virtual.machine",
        vmbox: "centos/stream8",
        vmbox_version: "20210210.0",
        ip: "10.0.0.10",
        memory: 2048,
        cpus: 3
    },
    {
        vagrant_hostname: "worker1",
        full_hostname: "worker1.virtual.machine",
        vmbox: "centos/stream8",
        vmbox_version: "20210210.0",
        ip: "10.0.0.12",
        memory: 2048,
        cpus: 1
    },
    {
        vagrant_hostname: "worker2",
        full_hostname: "worker2.virtual.machine",
        vmbox: "centos/stream8",
        vmbox_version: "20210210.0",
        ip: "10.0.0.14",
        memory: 2048,
        cpus: 1
    },
]

CRIO_VERSION="1.21"
OS="CentOS_8_Stream"
K8S_VERSION="1.21.4"



# individual machine names must be mentioned in below command line in
# order to bring up machines. (due to autostart: false)
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
    config.vm.provision :shell, privileged: true do |s|
      s.inline = $fix_ip_address
      # we need to pass quoted string here
      s.args = "'" + JSON.generate(all_hosts) + "'"
    end
    config.vm.provision :shell, privileged: true, name: "allow_bridged_traffic", inline: $allow_bridged_traffic
    config.vm.provision :shell, privileged: true, name: "install_crio", inline: $install_crio, args: [CRIO_VERSION, OS]
    config.vm.provision :shell, privileged: true, name: "install_k8s", inline: $install_k8s, args: K8S_VERSION
    config.vm.provision :shell, privileged: true, name: "setup_k8s_master", inline: $setup_k8s_master
    config.vm.provision :shell, privileged: true, name: "generate_join_cmd", inline: $generate_join_cmd
    config.vm.provision :shell, privileged: true, name: "setup_k8s_node", inline: $setup_k8s_node

end

########################################
#     Fix /etc/hosts file              #
########################################
$fix_ip_address = <<-FIXIP
# This script is idempotent
set -euo pipefail
echo "===================== Fixing /etc/hosts file ========================="
yum install -y perl jq

# remove `127.0.0.1 <full_hostname>` entry from /etc/hosts file
perl -wnl -i -s -e 'm!127.+\s+$hostname! or print' -- -hostname="$HOSTNAME" /etc/hosts

echo "$1" > all_hosts.json

cat all_hosts.json | jq -r '.[] | [.ip , .full_hostname] | @tsv' | tr '\t' ' ' > ip_host_format.txt

set +e
non_existence_entries=$(egrep -v -x -f /etc/hosts ip_host_format.txt)

if [[ ! -z "${non_existence_entries}" ]]
then
  echo "${non_existence_entries}" >> /etc/hosts
fi

cat /etc/hosts

FIXIP


########################################
#  IPtables must see Bridged traffic   #
########################################
$allow_bridged_traffic = <<-BRIDGED
set -euo pipefail
# This script is idempotent
swapoff -a
yum install -y  iproute-tc
cat <<EOF > /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
BRIDGED


########################################
#         Install CRIO                 #
########################################
$install_crio = <<-CRIO
set -euo pipefail
# This script is idempotent
echo "=====================Installing CRIO ========================="
yum -y install grubby
grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"

cat <<EOF > /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

cat <<EOF > /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system

VERSION="$1"
OS="$2"

curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo
yum -y install cri-o
systemctl daemon-reload
systemctl enable crio --now

CRIO



########################################
#         Install K8S                  #
########################################
$install_k8s = <<-K8S
set -euo pipefail

k8s_version="$1"

# This script is idempotent
echo "=====================Installing K8S ========================="
cat <<-'EOF' > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet-${k8s_version} kubeadm-${k8s_version} kubectl-${k8s_version} --disableexcludes=kubernetes

systemctl enable --now kubelet
K8S



########################################
#         Setup K8S Master             #
########################################
$setup_k8s_master = <<-SETUPMASTER
set -euo pipefail
if [[ $HOSTNAME == "master.virtual.machine" ]]
then
  echo "===================== Setup K8S Master ========================="
  kubeadm init --apiserver-advertise-address=10.0.0.10 --control-plane-endpoint=10.0.0.10 --node-name $HOSTNAME --pod-network-cidr="172.20.0.0/16" --upload-certs  2>&1 | tee kubeadm-init.out || :
  messages=$(cat kubeadm-init.out | egrep 'File.+exists|Port.+use' | wc -l || :)
  if [[ "$messages" -eq 10 ]]
  then
    echo "$HOSTNAME already configured as K8S control plane.."
  else
    mkdir -p ~vagrant/.kube
    cp -v -f /etc/kubernetes/admin.conf ~vagrant/.kube/config
    chown -R vagrant:vagrant ~vagrant/.kube
    echo "source <(kubectl completion bash)" >> ~vagrant/.bashrc

    # k8s control may not be ready by this time. So keep on trying for 2 mins
    time_now=$(date +%s)
    # try for 2 mins, 120 seconds
    time_expire=$((time_now + 120))
    calico_networking=false
    while [[ $time_now -lt $time_expire ]]
    do
      echo "=============== Applying Calico Networking =============="
      kubectl --kubeconfig=/etc/kubernetes/admin.conf apply -f https://docs.projectcalico.org/manifests/calico.yaml \
        && calico_networking=true \
        && break \
        || echo "FAILED..will keep on trying for 2 mins.." 1>&2
      sleep 2
      time_now=$(date +%s)
    done
    if [[ $calico_networking == false ]]
    then
      echo "ERROR: Failed to apply calico networking" 1>&2
      exit 1
    fi
  fi
fi

SETUPMASTER



########################################
# generate new join cmd on master      #
########################################
$generate_join_cmd = <<-JOINCMD
set -euo pipefail
if [[ $HOSTNAME == "master.virtual.machine" ]]
then
    mkdir -p /k8s_shared
    kubeadm token create --print-join-command > /k8s_shared/kube_join_command.sh
    yum install -y nfs-utils
    echo "====================== Exporting NFS Share: /k8s_shared ====================="
    systemctl start nfs-server
    echo "/k8s_shared *(ro,sync)" > /etc/exports
    exportfs -av
fi
JOINCMD



########################################
#         Setup K8S Node               #
########################################
$setup_k8s_node = <<-SETUPNODE
set -euo pipefail
if [[ $HOSTNAME != "master.virtual.machine" ]]
then
  echo "===================== Setup K8S Node ========================="
  # k8s control may not be ready by this time. So keep on trying for 2 mins
  mkdir -p /k8s_shared

  time_now=$(date +%s)
  # try for 2 mins, 60 seconds
  time_expire=$((time_now + 60))
  node_joined=false
  while [[ $time_now -lt $time_expire ]]
  do
    echo "=============== mounting master.virtual.machine:/k8s_shared on $HOSTNAME:/k8s_shared ... =============="
    mount -t nfs -o soft,timeo=10,retrans=1,retry=1,fg master.virtual.machine:/k8s_shared /k8s_shared \
      || { echo "mounting failed..will retry..." 1>&2 ; time_now=$(date +%s);  continue; }
    echo "=============== $HOSTNAME is trying to join cluster ... =============="
    messages=$(bash /k8s_shared/kube_join_command.sh 2>&1 | egrep 'File.+exists|Port.+use' | wc -l || :)
    if [[ "$messages" -eq 3 ]]
    then
      echo "$HOSTNAME already configured and joined to cluster"
      node_joined=true
      break
    fi

    bash /k8s_shared/kube_join_command.sh \
      && node_joined=true \
      && break \
      || echo "FAILED..will keep on trying for 1 mins.." 1>&2
    sleep 2
    time_now=$(date +%s)
  done
  if [[ $node_joined == false ]]
  then
    echo "ERROR: $HOSTNAME failed to join.." 1>&2
    exit 1
  fi
else
  echo "INFO: This is Control Plane. join command will not run here."
fi

SETUPNODE
```

## Deployment
Run the following command to deploy 1 master and 2 node kubernetes cluster

```bash
vagrant up master1 worker1 worker2
```
Above command will setup the whole cluster. When above command is finished, run the following command to verify the cluster state.

```bash
vagrant ssh master
kubectl get nodes -o wide
```

you shall see following output:
```
NAME                      STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE          KERNEL-VERSION          CONTAINER-RUNTIME
master.virtual.machine    Ready    control-plane,master   28m   v1.21.4   10.0.0.10     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
worker1.virtual.machine   Ready    <none>                 25m   v1.21.4   10.0.0.12     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
worker2.virtual.machine   Ready    <none>                 22m   v1.21.4   10.0.0.14     <none>        CentOS Stream 8   4.18.0-277.el8.x86_64   cri-o://1.21.2
```

## Adding more nodes
* If you want to add more nodes later on, you just need to add the host information in the top of `Vagrantfile` in `all_hosts` array.
* You need to generate the fresh token for new node to join the cluster, as the previous one might have been expired.
```bash
vagrant provision master --provision-with=generate_join_cmd
```
Above command will generate a new join command in a file in master node. Master node exports this file as NFS share which new node mounts it.
* Start the new node
```bash
vagrant up worker3
```
If you had started the new node prior to generating the join command, then new node may not join the cluster. In this case then generate the token on `master` node by running `vagrant provision master --provision-with=generate_join_cmd` command. and then run `vagrant up worker --provision`

## Notes
* All provisioning scripts are idempotent.
* you can run `vagrant up master --provision` or `vagrant up worker1 --provision` as many times as you like without any harm and in any order.
* You can create worker node without the existence of master node (`vagrant up worker2`) even, ofcourse it will not join the cluster then. You can create `master` node afterwards and then run command `vagrant up worker2 --provision` to join the cluster.
* GitHUB repo: https://github.com/spareslant/VagrantFiles.git
