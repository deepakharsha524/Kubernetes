#Kubernetes using Kubeadm

Install Kubernetes Cluster using kubeadm

Documentation to set up a Kubernetes cluster on CentOS 7 Virtual machines. Setting up cluster with one master node and two worker nodes
On both master and worker
##Pre-requisites
*Install, enable and start docker service*
*Disable SELinux*
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
*Disable Firewall*
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
