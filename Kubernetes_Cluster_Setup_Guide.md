# Kubernetes Cluster Setup on Ubuntu 22.04

This document provides a detailed guide for manually installing and configuring a Kubernetes cluster on Ubuntu 22.04. The setup includes one master node and two worker nodes, and each step is explained with its purpose to ensure a smooth installation process.

## Prerequisites

Before starting, ensure that all the nodes (master and workers) are on the same network, have static IP addresses assigned, and can communicate with each other. You should also disable swap and install Docker as a container runtime.

---

## 1. Set Hostname

### Purpose:
Each machine in the cluster should have a unique hostname to avoid conflicts and make it easier to manage.

1. **Set the hostname**:
    ```bash
    sudo hostnamectl set-hostname <new-hostname>
    ```

    Replace `<new-hostname>` with `master`, `worker1`, or `worker2` as appropriate for each machine.

2. **Verify the hostname**:
    ```bash
    hostnamectl
    ```

3. **Update the /etc/hosts file**:
    - Open the `/etc/hosts` file:
      ```bash
      sudo vi /etc/hosts
      ```
    - Add the following lines to map hostnames to IP addresses:
      ```
      192.168.1.10   master
      192.168.1.15   worker1
      192.168.1.16   worker2
      ```
    - Save and exit the editor.

---

## 2. Generate SSH Keys and Copy to Other Machines

### Purpose:
SSH keys enable secure, password-less login between the master and worker nodes, which is necessary for seamless cluster management.

1. **Generate SSH key on the master node**:
    ```bash
    ssh-keygen -t rsa -b 4096 -C "tuhin"
    ```

    Press `Enter` to accept the default file location and passphrase (or you can set a passphrase if desired).

2. **Copy SSH key to worker nodes**:
    ```bash
    ssh-copy-id tuhin@192.168.1.15
    ssh-copy-id tuhin@192.168.1.16
    ```

---

## 3. Set Static IP Addresses

### Purpose:
Static IP addresses ensure that each machine in your Kubernetes cluster consistently uses the same network address, which is crucial for reliable communication.

1. **Configure static IP**:
    - Open the netplan configuration file:
      ```bash
      sudo vi /etc/netplan/00-installer-config.yaml
      ```
    - Modify the file to include your static IP configuration. Example for `ens160`:
      ```yaml
      network:
        version: 2
        renderer: networkd
        ethernets:
          ens160:
            dhcp4: no
            addresses:
              - 192.168.1.10/24 # Replace with the appropriate IP for master or worker nodes
            gateway4: 192.168.1.1
            nameservers:
              addresses:
                - 8.8.8.8
                - 8.8.4.4
      ```
    - Save and exit the editor.

2. **Apply the netplan configuration**:
    ```bash
    sudo netplan apply
    ```

3. **Verify the IP address**:
    ```bash
    ip addr show ens160
    ```

---

## 4. Install Docker

### Purpose:
Docker is required as the container runtime for Kubernetes. This step installs Docker on all the nodes in your cluster.

1. **Update the package list**:
    ```bash
    sudo apt-get update
    ```

2. **Install Docker**:
    ```bash
    sudo apt-get install docker.io
    ```

3. **Enable Docker to start on boot**:
    ```bash
    sudo systemctl enable docker
    ```

4. **Start Docker if it is not running**:
    ```bash
    sudo systemctl start docker
    ```

5. **Verify Docker is running**:
    ```bash
    sudo systemctl status docker
    ```

---

## 5. Install Necessary Packages

### Purpose:
This step ensures the system has the essential packages required for setting up Kubernetes components.

1. **Update the package list**:
    ```bash
    sudo apt-get update
    ```

2. **Install essential packages**:
    ```bash
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
    ```

---

## 6. Disable Swap Permanently

### Purpose:
Kubernetes requires swap to be disabled. This step disables swap and makes the change permanent by modifying the `/etc/fstab` file.

1. **Disable swap immediately**:
    ```bash
    sudo swapoff -a
    ```

2. **Make swap disabling permanent**:
    - Open the `/etc/fstab` file in your preferred text editor (e.g., `vi`):
      ```bash
      sudo vi /etc/fstab
      ```
    - Find the line containing the word `swap` and add a `#` at the beginning of the line to comment it out. It might look something like this:
      ```bash
      # /swapfile swap swap defaults 0 0
      ```
    - Save and exit the editor.

---

## 7. Load Kernel Modules for Containerd

### Purpose:
Certain kernel modules are required for containerd, the container runtime, to function correctly with Kubernetes.

1. **Load the necessary modules**:
    - Open or create a configuration file for the kernel modules:
      ```bash
      sudo vi /etc/modules-load.d/k8s.conf
      ```
    - Add the following lines to the file:
      ```
      overlay
      br_netfilter
      ```
    - Save and exit the editor.

2. **Load the modules immediately**:
    ```bash
    sudo modprobe overlay
    sudo modprobe br_netfilter
    ```

---

## 8. Configure System Parameters for Kubernetes Networking

### Purpose:
This step configures the system to allow proper networking for Kubernetes components.

1. **Create a sysctl configuration file**:
    - Open or create a configuration file for sysctl parameters:
      ```bash
      sudo vi /etc/sysctl.d/k8s.conf
      ```
    - Add the following lines to the file:
      ```
      net.bridge.bridge-nf-call-iptables  = 1
      net.bridge.bridge-nf-call-ip6tables = 1
      net.ipv4.ip_forward                 = 1
      ```
    - Save and exit the editor.

2. **Apply the sysctl parameters**:
    ```bash
    sudo sysctl --system
    ```

---

## 9. Install Containerd

### Purpose:
Containerd is the container runtime used by Kubernetes. This step involves installing containerd and configuring it.

1. **Install containerd**:
    ```bash
    sudo apt-get -y install containerd
    ```

2. **Configure containerd**:
    - Create the necessary directory:
      ```bash
      sudo mkdir -p /etc/containerd
      ```
    - Generate the default configuration for containerd:
      ```bash
      sudo containerd config default | sudo tee /etc/containerd/config.toml
      ```
    - Open the configuration file to modify it:
      ```bash
      sudo vi /etc/containerd/config.toml
      ```
    - Find the line `SystemdCgroup = false` and change it to:
      ```bash
      SystemdCgroup = true
      ```
    - Save and exit the editor.

3. **Restart containerd to apply the new configuration**:
    ```bash
    sudo systemctl restart containerd
    ```

---

## 10. Install Kubernetes Components

### Purpose:
This step installs Kubernetes components like `kubeadm`, `kubelet`, and `kubectl` from the official Kubernetes repository.

1. **Download the Kubernetes signing key**:
    ```bash
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    ```

2. **Add the Kubernetes repository**:
    - Open or create a new repository list file:
      ```bash
      sudo vi /etc/apt/sources.list.d/kubernetes.list
      ```
    - Add the following line to the file:
      ```bash
      deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
      ```
    - Save and exit the editor.

3. **Update the package list again**:
    ```bash
    sudo apt-get update
    ```

4. **Install Kubernetes tools**:
    ```bash
    sudo apt-get install -y kubeadm kubelet kubectl
    ```

5. **Prevent automatic updates for Kubernetes tools**:
    ```bash
    sudo apt-mark hold kubeadm kubelet kubectl
    ```

6. **Verify the installation**:
    ```bash
    kubeadm version
    ```

---

## 11. Worker Node Preparation

### Purpose:
Before joining the worker nodes to the cluster, they need to be prepared with the necessary software and configuration.

1. **Repeat Steps 4 to 10**:
    - On each worker node, repeat the following steps from this guide:
        - **Step 4**: Install Docker
                - **Step 5**: Install Necessary Packages
        - **Step 6**: Disable Swap Permanently
        - **Step 7**: Load Kernel Modules for Containerd
        - **Step 8**: Configure System Parameters for Kubernetes Networking
        - **Step 9**: Install Containerd
        - **Step 10**: Install Kubernetes Components

    - These steps will ensure that the worker nodes are configured similarly to the master node and are ready to join the cluster.

---

## 12. Initialize the Kubernetes Cluster

### Purpose:
This step initializes the Kubernetes master node and sets up the Flannel network plugin for pod networking.

1. **Ensure swap is disabled**:
    ```bash
    sudo swapoff -a
    ```

2. **Initialize the Kubernetes master node**:
    ```bash
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ```

    - This command initializes your Kubernetes cluster with a pod network CIDR, which is necessary for Flannel. The output will provide you with commands to join worker nodes to the master.

3. **Set up kubeconfig for the current user**:
    - If you are logged in as a **non-root** user, use the following commands:
        ```bash
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```

    - If you are logged in as the **root** user, use these commands instead:
        ```bash
        export KUBECONFIG=/etc/kubernetes/admin.conf
        ```

    - These steps configure the `kubectl` command-line tool to use the configuration of your newly created cluster.

4. **Deploy Flannel network plugin**:
    - Download the Flannel YAML configuration file:
      ```bash
      wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
      ```
    - Apply the Flannel network configuration:
      ```bash
      kubectl apply -f kube-flannel.yml
      ```

    - Flannel is used here as the networking layer for Kubernetes. Applying the YAML file sets up the network across all nodes.

5. **Verify the status of the cluster**:
    ```bash
    kubectl get nodes
    kubectl get pods -A
    ```

    - The `kubectl get nodes` command checks that all nodes are connected, and `kubectl get pods -A` ensures all pods across namespaces are running correctly.

6. **Enable kubectl bash completion**:
    ```bash
    kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
    ```

    - This command enables bash completion for `kubectl`, which enhances your command-line experience by auto-completing commands and options.

---

## 13. Join Worker Nodes to the Cluster

### Purpose:
This step connects the worker nodes to the Kubernetes master node, enabling them to function as part of the cluster.

1. **Copy the join command from the master node**:
    - After initializing the master node, a command is provided in the output to join worker nodes to the cluster. It looks something like this:
      ```bash
      kubeadm join 192.168.1.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
      ```

2. **Run the join command on each worker node**:
    - SSH into each worker node and run the command provided by the master node:
      ```bash
      kubeadm join 192.168.1.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
      ```

    - This will connect the worker nodes to the master node, allowing them to participate in the cluster.

3. **Verify that worker nodes have joined the cluster**:
    - On the master node, run:
      ```bash
      kubectl get nodes
      ```

    - Ensure that the worker nodes are listed and their status is `Ready`.

---

## 14. Final Verification and Cluster Setup Completion

### Purpose:
This final step verifies that the Kubernetes cluster is fully operational with all nodes properly configured.

1. **Check the status of all nodes**:
    ```bash
    kubectl get nodes
    ```

    - All nodes (master and workers) should be in the `Ready` state.

2. **Check the status of all system pods**:
    ```bash
    kubectl get pods --all-namespaces
    ```

    - Ensure all system pods are running correctly without any issues.

3. **Deploy a test application** (optional):
    - You can deploy a simple test application to ensure everything is working as expected:
      ```bash
      kubectl create deployment nginx --image=nginx
      kubectl expose deployment nginx --port=80 --type=NodePort
      ```

    - This will deploy an NGINX web server and expose it on a NodePort, allowing you to access it via your worker nodes' IP addresses.

4. **Access the test application** (optional):
    - Find the NodePort assigned to the NGINX service:
      ```bash
      kubectl get svc
      ```
    - Access the service using the worker node's IP and the NodePort in a web browser:
      ```
      http://<worker-node-ip>:<NodePort>
      ```

    - You should see the default NGINX welcome page, confirming that your cluster is functioning correctly.

---

Congratulations! Your Kubernetes cluster on Ubuntu 22.04 with Flannel as the network plugin is now fully set up and ready for use.
