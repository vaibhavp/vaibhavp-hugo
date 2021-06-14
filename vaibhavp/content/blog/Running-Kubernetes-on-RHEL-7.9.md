---
title: "Running Kubernetes on RHEL 7.9"
date: 2021-06-14T17:31:48+05:30
draft: false
---

# Introduction
We know what is Kubernetes and how much it's important in today's world of micro-services. So today we are going to see how to create a Kubernetes cluster on Red Hat Linux 7. For this tutorial we are going to use RHEL 7.9 version.  

In this Kubernetes cluster we are just going to have only one node cluster i.e. we will use `kubeadm` tool to install all Kubernetes master node components on single node. On master node following components will be installed. 

- **API Server** – It provides Kubernetes API using Jason / Yaml over http, states of API objects are stored in etcd
- **Scheduler** – It is a program on master node which performs the scheduling tasks like launching containers in worker nodes based on resource availability
- **Controller Manager** – Main Job of Controller manager is to monitor replication controllers and create pods to maintain desired state.
- **etcd** – It is a Key value pair data base. It stores configuration data of cluster and cluster state.
- **Kubectl utility** – It is a command line utility which connects to API Server on port 6443. It is used by administrators to create pods, services etc.

![kube-arch](/img/k8s-archi-diagram.svg)

Please follow below steps to setup your own Kubernetes cluster. 

# Step 1: Disable SELinux & setup firewall rules

Login into you RHEL7 host and disable the selinux and setup firewall rule. 

```bash
[root@k8s-master ~] sudo setenforce 0
[root@k8s-master ~] sudo sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux

# Now will set firewall rules.
# Note first checks if you have a firewall installed or not. If you are using the GCP instance then you don't require to do it.  
[root@k8s-master ~] sudo firewall-cmd --permanent --add-port=6443/tcp
[root@k8s-master ~] sudo firewall-cmd --permanent --add-port=2379-2380/tcp
[root@k8s-master ~] sudo firewall-cmd --permanent --add-port=10250/tcp
[root@k8s-master ~] sudo firewall-cmd --permanent --add-port=10251/tcp
[root@k8s-master ~] sudo firewall-cmd --permanent --add-port=10252/tcp
[root@k8s-master ~] sudo firewall-cmd --permanent --add-port=10255/tcp
[root@k8s-master ~] sudo firewall-cmd --reload
[root@k8s-master ~] sudo modprobe br_netfilter
[root@k8s-master ~] sudo echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

# Step 2: Disable Swap

Disable Swap in all nodes using “swapoff -a” command and remove or comment out swap partitions or swap file from fstab file

# Step 3: Cofigure Kubernetes Repository

Use below command to configure its package repositories. 

```bash
[root@k8s-master ~] sudo cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=1
> repo_gpgcheck=1
> gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
>         https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
> EOF 
[root@k8s-master ~]#
```

# Step 4: Install Kubeadm and Docker

Now its time to install `Docker` and `Kubeadm` tool which we will required for installing and running kubernetes cluster components. 

```bash
[root@k8s-master ~] sudo yum install kubeadm docker -y

# After installing kubeadm and docker restart its services.
[root@k8s-master ~] sudo systemctl restart docker && sudo systemctl enable docker
[root@k8s-master ~] sudo systemctl  restart kubelet && sudo systemctl enable kubelet
```

# Step 4: Initialize Kubernetes Master Node with ‘kubeadm init’

Now we are going to initialize the process of creating Kubernetes cluster.

```bash
[root@k8s-master ~] sudo kubeadm init --pod-network-cidr=192.168.0.0/16
.
.
.# After completing this command 
.
[root@k8s-master ~] mkdir -p $HOME/.kube
[root@k8s-master ~] sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@k8s-master ~] sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# Setp 5: Install CNI plugin

Now we are going to install `calico` for our cluster networking part.

```bash
[root@k8s-master ~] kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
[root@k8s-master ~] kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
[root@k8s-master ~] kubectl taint nodes --all node-role.kubernetes.io/master-

#Noe check your node it ready or not 
[root@k8s-master ~] kubectl get nodes -o wide
```

Here we complete the cluster creation, now we can use it.  In next step we will see how to deploy web ui for our cluster on it. 

# Step 6: Install Dashboard on Cluster

Here we will deploy Dashboard to visualize our cluster.

```bash
[root@k8s-master ~] kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```

This will deploy dashboard application in dashboard namespaces. Now we will create user for our dashboard.

 

```bash
[root@k8s-master ~] cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

[root@k8s-master ~] cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Now we need to find a token we can use to log in. Execute the following command:

[root@k8s-master ~] kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

#It should print something like:

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
```

 Use above generated token for Dashboard login.

 # Conclusion
 Thus we have seen how to create a single node cluster on your RHEL 7.9 host. Also deployed a simple app provided by the Kubernetes community to control your cluster using simple UI. 