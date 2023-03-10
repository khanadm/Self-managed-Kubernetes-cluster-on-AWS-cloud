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



Now run In Master kubectl --insecure-skip-tls-verify get nodes -o wide



![NODES](https://user-images.githubusercontent.com/106643382/209911383-38b1d614-239f-4377-80a2-c0a9f7f8c5b9.png "NODES")


Step 3 Deploy one WordPress website using deployment kind
---

Before creating deployment we need to create Database because Wordpress requires WORDPRESS_DB_HOST, WORDPRESS_DB_USER, 
WORDPRESS_DB_PASSWORD, WORDPRESS_DB_NAME else Wordpress container will exit again and again.


So, First create a RDS(MySQL) DB from AWS console

![Create databases](https://user-images.githubusercontent.com/106643382/209912295-862584dc-d5cc-4c97-a336-e2a4ba4b1ace.png "Create databases")


Click Create database

![mysql](https://user-images.githubusercontent.com/106643382/209912603-74df870e-72a7-42d6-ba88-eed56b0e5cba.png "mysql")



![CREDEMITAL SETTING](https://user-images.githubusercontent.com/106643382/209912841-4c2739c2-6729-493f-9991-4183beb23092.png "CREDEMITAL SETTING")


![secqurity group,ports](https://user-images.githubusercontent.com/106643382/209913086-3e318246-f17f-4205-9df8-82697c0b0098.png "secqurity group ,ports")



![additional conf](https://user-images.githubusercontent.com/106643382/209913264-70613aa4-2ee7-48b6-8937-b942e138fa9f.png "additional conf")




![logs](https://user-images.githubusercontent.com/106643382/209913460-cc71d9a7-7b88-4b28-90c4-07cbc1092c90.png "logs")



Now, time to create deployment using YAML file



```
 sudo vim wordpress.yml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:latest
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: database-1.c9qq8he3gv6d.ap-south-1.rds.amazonaws.com
        - name: WORDPRESS_DB_USER
          value: admin
        - name: WORDPRESS_DB_PASSWORD
          value: password
        - name: WORDPRESS_DB_NAME
          value: blog
        ports:
        - containerPort: 80
          name: wordpress
```



![wordpress.yml](https://user-images.githubusercontent.com/106643382/209914578-da1bcd5a-c924-4a42-a7f7-6439d50860d1.png "wordpress.yml")

Databases Host

![database host](https://user-images.githubusercontent.com/106643382/209915872-9aac9c52-25be-4c81-9199-516e26e5bfbb.png "database host")


Now, create deployment using kubectl create -f <FILENAME.yml>


To check pods is running successfully or not run ```kubectl get pods -o wide```


To describe pods run ```kubectl describe pods <POD_NAME>```


Now we have create a service file called service.yml to expose  .

```
sudo vim service.yml
```

```
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30534
  selector:
    app: wordpress
    tier: frontend
  type: NodePort
```               

![Service.yml](https://user-images.githubusercontent.com/106643382/209915501-a2723c1a-8e83-454a-bfbe-826076108f3a.png "Service.yml")



Now, create deployment using kubectl create -f <FILENAME.yml>


To check pods is running successfully or not run ```kubectl get svc```

![svc](https://user-images.githubusercontent.com/106643382/209916955-f7323a29-55a8-4b47-b07a-2344bdee38fc.png "svc")


To describe pods run ```kubectl describe pods <POD_NAME>```

Now hit publicip:31456

![wordpress](https://user-images.githubusercontent.com/106643382/209917185-4bc3cbc6-7906-42e0-bd62-e0348d412be4.png "wordpress")


Now install wordpress and create a post

![PAGE](https://user-images.githubusercontent.com/106643382/209918228-a7ae6f1a-d337-4bfa-b848-7130f851f3a9.png "PAGE")

#### Step-4: Connect RDS as a DB with WordPress Site

Already done in step-3

 #### Step-5: Migrate the data of K8S DB in RDS

 Now take a dump of mysql db from any node using ```mysqldump -h <ENDPOINT> -u <USERNAME> -p <DB_NAME>```
 
 
 mysql -h database-1.c9qq8he3gv6d.ap-south-1.rds.amazonaws.com -u admin -p blog < mydb.sql

![dump](https://user-images.githubusercontent.com/106643382/209921102-b38b26fd-3ff6-40b7-9021-609865aff352.png "dump")


Now delete db using drop database <DB_NAME> & hit publicip:31456

![db deleted](https://user-images.githubusercontent.com/106643382/209921417-3861d4a3-4e1f-47a9-a694-06defba86814.png "db deleted")



Wordpress is not able to access db as db is deleted. So, create a new db with same name and put dump inside this db.


 create a new db with same name and put dump inside this db.

```
 mysql -h database-1.c9qq8he3gv6d.ap-south-1.rds.amazonaws.com -u admin -p blog < mydb.sql
```

Now again try to hit InstanceIP:30838 This time you can see yours site



![RE-LOADED](https://user-images.githubusercontent.com/106643382/209922161-f7dc37b7-3cd1-4f02-9359-fc55be1c58f0.png "RE-LOADED")


#### THANKYOU
