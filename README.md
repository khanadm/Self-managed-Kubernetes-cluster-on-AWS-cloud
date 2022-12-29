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


Accessing K8S cluster from master node
---

For this, run below commands from master node:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


To check kubectl working fine run ```kubectl get pods --all-namespaces```



![PODS](https://user-images.githubusercontent.com/106643382/209909930-39c8df29-ed94-4408-83c9-a3e8553194b3.png "PODS]")



Now, time to configure slave/worker node
---


First login to slave/worker node via ssh:


Install docker as Kubernetes requires a container runtime and will use docker

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


Install kubeadm that will join slave to master node. Also, kubectl and kubelet will also be installed as dependency



```
sudo yum install kubeadm kubelet -y
```


Enable kubelet using ```sudo systemctl enable kubelet --now```


Now, install **iproute-tc** using ```sudo yum install iproute-tc -y```


Lastly, join slave to master node using ```sudo kubeadm join 172.31.39.152:6443 --token nlk5qe.7tj1wedsmxdyycio --discovery-token-ca-cert-hash sha256:38353362ba17b9079be482d2c118880b5ae35eb9f4deed6af0a378ab8ab1fda3```



Similarly, configure another slave node






























