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







