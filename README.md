
# 🚀 Project: Ansible Cluster Setup Using Docker & Kubernetes (Red Hat-Based Image)

## 📌 Objective

To manually set up an **Ansible Master-Node architecture** inside both **Docker containers** and **Kubernetes pods**, using a custom-built Red Hat-based image without relying on pre-built Ansible images.

---

## 🧱 Environment Stack

- OS Base: AlmaLinux 8 (RHEL-compatible) or Rocky Linux
- Automation Tool: Ansible
- Container Runtime: Docker
- Orchestration Platform: Kubernetes
- Communication: SSH between Master and Nodes

---

## 🧩 Part A: Ansible Cluster Using Docker

---

### 🔹 Step 1: Create a Custom Dockerfile with Ansible

```Dockerfile
# Dockerfile
#ansible configurations using redhat
FROM redhat/ubi9

ENV container=docker \
    ANSIBLE_FORCE_COLOR=true \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

RUN dnf update -y && \
    dnf install -y python3 python3-pip openssh-server openssh-clients sudo vim passwd && \
    pip3 install --upgrade pip && \
    pip3 install ansible && \
    echo "root:root" | chpasswd && \
    mkdir -p /var/run/sshd /run/sshd && \
    ssh-keygen -A && \
    echo "PermitRootLogin yes" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
```

---

### 🔹 Step 2: Build the Docker Image

```bash
docker build -t rhel-ansible .
```

---

### 🔹 Step 3: Create Docker Network

```bash
docker network create ansible-net
```

---

### 🔹 Step 4: Run Ansible Master and Nodes

```bash
# Master Node
docker run -dit --name ansible-master --network ansible-net rhel-ansible

# Worker Nodes
docker run -dit --name ansible-node1 --network ansible-net rhel-ansible
docker run -dit --name ansible-node2 --network ansible-net rhel-ansible
```

---

### 🔹 Step 5: Configure SSH Access from Master to Nodes

```bash
docker exec -it ansible-master bash

# Inside container:
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa

# Install sshpass if needed:
dnf install -y sshpass

# Copy SSH keys to worker nodes
sshpass -p root ssh-copy-id root@ansible-node1
sshpass -p root ssh-copy-id root@ansible-node2
```

---

### 🔹 Step 6: Create Ansible Inventory File

```bash
echo "
[node]
ansible-node1
ansible-node2
" > /etc/ansible/hosts
```

---

### 🔹 Step 7: Test Connection with `ansible -m ping`

```bash
ansible all -m ping
```

---

## ☸️ Part B: Ansible Cluster Using Kubernetes Pods

---

### 🔹 Step 1: Push Docker Image to DockerHub

```bash
docker tag rhel-ansible <your-dockerhub-username>/rhel-ansible:v1
docker push <your-dockerhub-username>/rhel-ansible:v1
```

---

### 🔹 Step 2: Kubernetes YAML for Master Pod

```yaml
# ansible-master.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ansible-master
  labels:
    role: master
spec:
  containers:
  - name: master
    image: <your-dockerhub-username>/rhel-ansible:v1
    ports:
    - containerPort: 22
```

---

### 🔹 Step 3: Kubernetes YAML for Node Deployment

```yaml
# ansible-node-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ansible-node
spec:
  replicas: 2
  selector:
    matchLabels:
      role: node
  template:
    metadata:
      labels:
        role: node
    spec:
      containers:
      - name: node
        image: <your-dockerhub-username>/rhel-ansible:v1
        ports:
        - containerPort: 22
```

---

### 🔹 Step 4: Apply Kubernetes YAMLs

```bash
kubectl apply -f ansible-master.yaml
kubectl apply -f ansible-node-deploy.yaml
```

---

### 🔹 Step 5: Get Pod IPs

```bash
kubectl get pods -o wide
```

---

### 🔹 Step 6: Configure SSH from Master to Node Pods

```bash
kubectl exec -it ansible-master -- bash

# Inside Pod
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
dnf install -y sshpass
sshpass -p root ssh-copy-id root@<node1-pod-ip>
sshpass -p root ssh-copy-id root@<node2-pod-ip>
```

---

### 🔹 Step 7: Create Inventory & Ping Test

```bash
echo "
[node]
<node1-pod-ip>
<node2-pod-ip>
" > /etc/ansible/hosts

ansible all -m ping
```

---

## ✅ Final Outcome

- Ansible Master is able to connect and control Nodes inside **Docker** and **Kubernetes**.
- You’ve built everything manually using **Red Hat-based custom images**.
- Setup mimics real-world automation infrastructure.

---

## 🔧 Optional: Sample Playbook to Install HTTPD

```yaml
# install_httpd.yaml
- name: Install Apache Web Server
  hosts: node
  become: true
  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: present
    - name: Start httpd
      service:
        name: httpd
        state: started
        enabled: true
```

Run it using:

```bash
ansible-playbook install_httpd.yaml
```

---

## 📝 Author

- **Your Name:** Ritik Kumar Sahu 
