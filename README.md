# Self-managed-Kubernetes-cluster-on-AWS-cloud



#### Description:

- Launch 3 EC2 servers
- Configure first as a master node and the other two as a slave nodes
- Deploy one WordPress website using deployment kind
- Connect RDS as a DB with WordPress Site
- Migrate the data of K8S DB in RDS

---



#### First step: Create 3 EC2 server 1 Master & 2 worker node
 
  - Firstly, log in to the AWS management console.
  - After login, navigate to the search bar, type EC2, and select EC2
  - Now, the EC2 dashboard will appear. Click Instances
  - Click the Launch instances button.
  - To launch an EC2 instance, a few details are required, i.e., instance name, OS image (AMI), instance type, etc.
  - Select a key pair to attach with the instance to log in with that key.
  - Click the Launch instance button
  - Take the public IP from the EC2 dashboard and use it to login inside the instance using ssh.
  - Finally, it will be logged in successfully if everything is configured correctly.
  
  ---


#### Step-2: Configure first as a master node and the other two as a slave nodes

Let's configure master node

Install docker as Kubernetes service like kube-apiserver, kube-controller-manager, kube-scheduler, etc will run in containers.

```
sudo yum install docker -y
```

After installation done enable docker service so that after os reboot docker service will be in running state.

```
sudo systemctl enable docker --now
```

Now we have to configure yum so that yum can able to download and install kubeadm and kubelet.


For this create a repo file with any name inside **/etc/yum.repos.d/** using ```sudo vim /etc/yum.repos.d/kubernetes.repo```

Write below content inside file:


```
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

Now, press Esc key to exit from insert mode then type :wq!


Install kubeadm that will create a k8s cluster. Also, kubectl and kubelet will also be installed as dependency

```
sudo yum install kubeadm -y
```


Now, download all docker images required for each service like kube-apiserver, kube-controller-manager, kube-scheduler, etc using:

```
sudo kubeadm config images pull
```

Enable kubelet using ```sudo systemctl enable kubelet --now```


Now, install **iproute-tc** using ```sudo yum install iproute-tc -y```


Let's create kubernetes cluster using ```sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem```


Here in my case, i used t2.micro instance type which provides 1 CPU and 1 GB RAM but At least 2 CPU and 2 GB RAM is recommended by Kubernetes. So to 
work with t2.micro instance type we passed **--ignore-preflight-errors=NumCPU --ignore-preflight-errors=Mem**







