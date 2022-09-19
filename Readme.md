# Kubernetes using Kubeadm

Install Kubernetes Cluster using kubeadm





Documentation to set up a Kubernetes cluster on CentOS 7 Virtual machines. Setting up cluster with one master node and two worker nodes
On both master and worker
## Pre-requisites
* Install, enable and start docker service *
* Disable SELinux *
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
* Disable Firewall *
systemctl disable firewalld
systemctl stop firewalld
Disable swap
sed -i '/swap/d' /etc/fstab
swapoff -a
Update sysctl settings for Kubernetes networking
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

Kubernetes Setup
Add yum repository
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

Install Kubernetes
yum install -y kubeadm kubelet kubectl


Enable and Start kubelet service
systemctl enable kubelet
systemctl start kubelet

On kmaster
Initialize Kubernetes Cluster








# Activate Cluster autoscaler

The cluster autoscaler automatically launches additional worker nodes if more resources are needed, and shutdown worker nodes if they are underutilized. The autoscaling works within a nodegroup, hence create a nodegroup first which has this feature enabled.

## create nodegroup with autoscaler enabled

```bash
eksctl create nodegroup --config-file=eks-course.yaml

eksctl delete nodegroup --cluster=EKS-course-cluster --name=ng-1 --approve
```

## deploy the autoscaler

the autoscaler itself:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

put required annotation to the deployment:

```bash
kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```

get the autoscaler image version:  
open https://github.com/kubernetes/autoscaler/releases and get the latest release version matching your Kubernetes version, e.g. Kubernetes 1.14 => check for 1.14.n where "n" is the latest release version

edit deployment and set your EKS cluster name:

```bash
kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```

* set the image version at property ```image=k8s.gcr.io/cluster-autoscaler:vx.yy.z```  
* set your EKS cluster name at the end of property ```- --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<<EKS cluster name>>```

## view cluster autoscaler logs

```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler
```

## test the autoscaler

### create a deployment of nginx

```bash
kubectl apply -f nginx-deployment.yaml
```

### scale the deployment

```bash
kubectl scale --replicas=3 deployment/test-autoscaler
```

### check pods

```bash
kubectl get pods -o wide --watch
```

### check nodes 

```bash
kubectl get nodes
```

### view cluster autoscaler logs

```bash
kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "Expanding Node Group"

kubectl -n kube-system logs deployment.apps/cluster-autoscaler | grep -A5 "removing node"
```
