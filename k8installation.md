# Kubernetes: Kubernetes is an open-source system for managing, scaling, and deploying containerized applications. It was originally developed at Google and released in 2014.

# ===========================================================================================
#                                   Ubuntu Server Setup
# ===========================================================================================

# *** Initial Setup: Update and Upgrade Ubuntu Server ***

# Update and upgrade Ubuntu packages
sudo apt update && sudo apt upgrade -y
sudo apt-get update && sudo apt-get upgrade -y

# Ensure the system is connected to the internet via DHCP before setting a static IP.
# A NAT Bridge should be provided to the virtual machine to ensure internet access during the initial setup.

# ===========================================================================================
#                                   Netplan Configuration
# ===========================================================================================

# *** Configure Network using Netplan ***

# Install net-tools package to enable ifconfig and other network tools
sudo apt install -y net-tools

# View the Netplan configuration file location
sudo cat /etc/netplan/00-installer-config.yaml

# Example configuration for static IP:
# Edit the configuration file as required:
#   network:
#     version: 2
#     renderer: NetworkManager
#     ethernets:
#       ens33:
#         addresses:
#           - 192.168.100.209/24
#         nameservers:
#           addresses: [8.8.8.8, 8.8.4.4]
#         routes:
#           - to: default
#             via: 192.168.100.1

# Apply the configuration
sudo netplan try
sudo netplan apply

# Verify the network configuration
ip a

# ===========================================================================================
#                                   NetworkManager Configuration
# ===========================================================================================

# *** Install and Configure NetworkManager ***

# Install NetworkManager
sudo apt install -y network-manager

# Enable and start NetworkManager service
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager

# Verify NetworkManager status
systemctl status NetworkManager

# Example of using nmcli to configure a network connection:
nmcli connection add con-name lan1 type ethernet ifname ens33 ipv4.method manual ipv4.addresses 192.168.100.10/24 ipv4.gateway 192.168.100.1 ipv4.dns 8.8.8.8 autoconnect yes

# In case of "Device is strictly unmanaged" error, bring up the connection manually:
nmcli connection up lan1

# ===========================================================================================
#                                   Docker Installation
# ===========================================================================================

# *** Add Docker's official GPG key and install Docker ***

# Update and install necessary certificates and tools
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to APT sources
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package list and install Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin

# Enable and start Docker service
sudo systemctl enable docker
sudo systemctl start docker

# Verify Docker installation
sudo systemctl status docker

# Example of running a Docker container:
sudo docker run -dit --name web1 nginx

# Verify running containers
sudo docker ps

# Inspect a container for detailed information:
sudo docker inspect web1

# Curl the container IP to check the nginx default welcome page:
sudo curl 172.17.0.2

# Example of removing a container and cleaning up:
sudo docker stop web1
sudo docker rm web1

# ===========================================================================================
#                                   Docker Networking
# ===========================================================================================

# *** Custom Docker Network ***

# Create a custom Docker network
sudo docker network create customsnet1 --subnet 192.168.1.0/24

# Verify the network creation
sudo docker network ls

# Example of running a container in the custom network:
sudo docker run -dit --name os1 --network customsnet1 ubuntu:latest

# Verify the container's IP address in the custom network:
sudo docker inspect os1 | grep IPAddress

# ===========================================================================================
#                                   Docker Copy (docker cp)
# ===========================================================================================

# *** Copying Files between Host and Container ***

# Example of copying a file from host to container:
sudo docker cp index.html nginx:/usr/share/nginx/html

# Example of copying a file from container to host:
sudo docker cp nginx:/usr/share/nginx/html/index.html ./

# ===========================================================================================
#                                   Kubernetes Cluster Setup
# ===========================================================================================

# *** Install Docker on Kubernetes Nodes ***
# *** Install kubeadm, kubelet, and kubectl ***
# Master Node: kubeadm + kubelet + kubectl
# Worker Node: kubeadm + kubelet

# Example command to initialize the Kubernetes Cluster:
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# After initialization, set up kubeconfig for the user:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Alternatively, as root user:
export KUBECONFIG=/etc/kubernetes/admin.conf

# ===========================================================================================
#                             Fixing containerd Configuration Issue
# ===========================================================================================

# *** Fix containerd configuration issue if encountered during kubeadm init ***
# If you encounter an error related to CRI runtime during kubeadm initialization,
# you may need to modify the containerd configuration.

# Option 1: Delete the containerd config.toml file
sudo rm /etc/containerd/config.toml

# Option 2: Comment out the "disabled_plugins = ['cri']" line in config.toml
sudo vim /etc/containerd/config.toml
# In vim, add a '#' in front of the line:
# disabled_plugins = ["cri"]

# Restart containerd service to apply changes
sudo systemctl restart containerd.service

# ===========================================================================================
#                                   Flannel Network Plugin
# ===========================================================================================

# Apply the Flannel network plugin:
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# ===========================================================================================
#                                   Kubernetes Management
# ===========================================================================================

# *** Kubernetes Objects and Management ***

# View cluster nodes:
kubectl get nodes

# View all pods in all namespaces:
kubectl get pods -A

# Example of creating a deployment:
kubectl create deployment dep1 --replicas 10 --image nginx

# Example of deleting all pods:
kubectl delete pods --all

# Verify the cluster status:
kubectl get nodes -o wide
