Setup of a Highly Available( Kubernetes Cluster
------------------------------------------------------------------------

Setting up a CentOS 8 VM instance on the machine.
After installing CentOS 8 and ensuring creation of user with **sudo previleges**, the next step is installing **Docker** where our **Kubernetes** will run on.


### Installing docker

1. Install docker community edition using command: `yum -y install dnf install docker-ce-3:19.09.1-3.el7`.
2. Start and enabling docker daemon: `yum -y install systemctl enable --now docker`.
3. Adding to user to the docker group: `yum -y install usermod -aG docker $USER`.
4. After completion of installation log out and log back in.

### Installing kubernetes
 
1. Adding kubernetes repository from terminal: `yum -y install vim/etc/yum.repos.d/kubernetes.repo`
2. Adding content to newly created file:
    ```
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg                                                       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    ```
Ensure to save before closing the file.
3. Installing needed packages: `yum -y install dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`.
4. Enable kubelet daemon: `yum -y install systemctl enable --now kubelet`.
5. Using root user privileges run: `vim /etc/sysctl.d/k8s.conf`
    Inside the file add below lines:
    ```
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    ```
Save and close the file
6. Reloading the configuration: `sysctl --system`
7. Exit root user with `root` command
**_Note_**: Before initializing **Kubernetes cluster**, disable swap using: `sudo swapoff -a`

Below is flow diagram of the system built:

![kubernetes cluster](/images/kubernetes.PNG)

VIP(Virtual IP) setup:

1. Master 1: 192.168.1.1
2. Master 2: 192.168.1.2

Configuration of Kubernetes Master HA
----------------------------------------------------------

**1. Install Kubernetes on Master 2**: We create a new repo for Kubernetes:

```
vi /etc/yum.repos.d/virt7-docker-common-release.repo
 
[virt7-docker-common-release]
 
name=virt7-docker-common-release
baseurl=http://cbs.centos.org/repos/virt7-docker-common-release/x86_64/os/
gpgcheck=0
```

Installing Kubernetes:

```
yum -y install --enablerepo=virt7-docker-common-release kubernetes etcd flannel
```

**2. Create ETCD Cluster using etcd configuration**: ETCD configuration file is located at _/etc/etcd/etcd.conf_

**Master 1**

```
# [member]
 
ETCD_NAME=infra0
 
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
 
ETCD_LISTEN_PEER_URLS="http://192.168.1.1:2380"
 
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.1:2379.http://127.0.0.1:2379"
 
#
 
#[cluster]
 
ETCD_INITIAL_CLUSTER="infra0=http://192.168.1.1:2380,infra1=http://192.168.1.2:2380"
 
ETCD_INITIAL_CLUSTER_STATE="new"
 
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
 
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.1:2379"
```

**Master 2**

```
# [member]
 
ETCD_NAME=infra1
 
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
 
ETCD_LISTEN_PEER_URLS="http://192.168.1.2:2380"
 
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.2:2379.http://127.0.0.1:2379"
 
#
 
#[cluster]
 
ETCD_INITIAL_CLUSTER="infra0=http://192.168.1.1:2380,infra1=http://192.168.1.2:2380"
 
ETCD_INITIAL_CLUSTER_STATE="new"
 
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
 
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.2:2379"
```

**NOTE**: Restart etcd in all the master nodes `systemctl restart etcd`
To check if etcd cluster was formed properly: `etcdctl cluster-health`

**3. Configuring other Kubernetes master components**:

_**On Master**_

```
vi /etc/kubernetes/config
 
# Comma separated list of nodes running etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://Master_Private_IP:2379"
# Logging will be stored in system journal
KUBE_LOGTOSTDERR="--logtostderr=true"
# Journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"
# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"
# Api-server endpoint used in scheduler and controller-manager
KUBE_MASTER="--master=http://Master_Private_IP:8080"
```

_**On Minion**_

```
vi /etc/kubernetes/config
 
# Comma separated list of nodes running etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://k8_Master:2379"
# Logging will be stored in system journal
KUBE_LOGTOSTDERR="--logtostderr=true"
# Journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"
# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"
# Api-server endpoint used in scheduler and controller-manager
KUBE_MASTER="--master=http://k8_Master:8080"
```

Copy all the certificates from existing master to the other master(with same file permission). All the certificates are stored at **/srv/kubernetes/**

_**On Master**_: Updating API Server Configuration

```
vi /etc/kubernetes/apiserver
 
# Bind kube api server to this IP
KUBE_API_ADDRESS="--address=0.0.0.0"
# Port that kube api server listens to.
KUBE_API_PORT="--port=8080"
# Port kubelet listen on
KUBELET_PORT="--kubelet-port=10250"
# Address range to use for services(Work unit of Kubernetes)
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# Add your own!
KUBE_API_ARGS=" --client-ca-file=/srv/kubernetes/ca.crt --tls-cert-file=/srv/kubernetes/server.cert --tls-private-key-file=/srv/kubernetes/server.key"
```

_**On Master**_: Configuring Kubernetes Controller Manager

```
vi /etc/kubernetes/controller-manager
 
KUBE_CONTROLLER_MANAGER_ARGS="--root-ca-file=/srv/kubernetes/ca.crt --service-account-private-key-file=/srv/kubernetes/server.key"
```

_**On Master and Minion**_: Configure Flanneld

```
vi /etc/sysconfig/flanneld
 
# etcd url location. Point this to the server where etcd runs
FLANNEL_ETCD="http://k8_Master:2379"
# etcd config key. This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_PREFIX="/kube-centos/network"
# Any additional options that you want to pass
FLANNEL_OPTIONS=""
```

Only one master must be active at any particular time so that the cluster remains in the consistent state. For this, we need to configure Kubernetes Controller Manager and Scheduler. Start these two services with **–leader-elect** option.

Updating configuration file:

```
vi /etc/kubernetes/controller-manager
 
KUBE_CONTROLLER_MANAGER_ARGS="--root-ca-file=/srv/kubernetes/ca.crt --service-account-private-key-file=/srv/kubernetes/server.key --leader-elect"
 
vi /etc/kubernetes/scheduler
 
KUBE_SCHEDULER_ARGS="--leader-elect"
```

**4. Creating a Load Balancer**

_____ Master1 Port 8080
 |
Load Balancer Port 8080 -- ---- Master2 Port 8080

 _____ Master1 Port 2379
 |
Load Balancer Port 2379 -- ---- Master2 Port 2379

**5.** Replace Master IP in **/etc/hosts** of all **Minion** by IP address of Load Balancer and restart all kubernetes service in Minions
