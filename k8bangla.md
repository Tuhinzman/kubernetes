# ============================================================
# Kubernetes Cluster Setup Script
# ============================================================

# *** Ubuntu Server Initial Setup: Update and Upgrade Ubuntu Server ***
# সার্ভার আপডেট ও আপগ্রেড করা
sudo apt update && sudo apt upgrade -y
sudo apt-get update && sudo apt-get upgrade -y

# *** ইন্টারনেট সংযোগ নিশ্চিত করুন ***
# DHCP এর মাধ্যমে ইন্টারনেট সংযোগ নিশ্চিত করুন স্ট্যাটিক আইপি সেট করার আগে
# ইন্টারনেট না থাকলে সিস্টেম আপডেট এবং আপগ্রেড করা যাবে না
# ভার্চুয়াল মেশিনের জন্য একটি NAT ব্রিজ প্রদান করুন

# ============================================================
# Netplan Configuration
# ============================================================

# *** Netplan ব্যবহার করে নেটওয়ার্ক কনফিগারেশন ***
# প্রয়োজনীয় net-tools প্যাকেজ ইনস্টল করা
sudo apt install -y net-tools

# Netplan কনফিগারেশন ফাইল দেখতে
sudo cat /etc/netplan/00-installer-config.yaml

# স্ট্যাটিক আইপি সেট করার জন্য কনফিগারেশন ফাইলটি এডিট করুন
sudo vim /etc/netplan/00-installer-config.yaml
# উদাহরণ কনফিগারেশন:
# network:
#   version: 2
#   renderer: NetworkManager
#   ethernets:
#     ens33:
#       addresses:
#         - 192.168.100.209/24
#       nameservers:
#         addresses: [8.8.8.8, 8.8.4.4]
#       routes:
#         - to: default
#           via: 192.168.100.1

# Netplan কনফিগারেশন প্রয়োগ করুন এবং পরীক্ষা করুন
sudo netplan try
sudo netplan apply

# নেটওয়ার্ক কনফিগারেশন যাচাই করুন
ip a

# ============================================================
# NetworkManager Configuration
# ============================================================

# *** NetworkManager ইনস্টল এবং কনফিগার করুন ***
sudo apt install -y network-manager

# NetworkManager সার্ভিস সক্রিয় এবং শুরু করা
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager

# NetworkManager এর স্ট্যাটাস যাচাই করা
systemctl status NetworkManager

# উদাহরণ: nmcli ব্যবহার করে নেটওয়ার্ক সংযোগ কনফিগার করুন
nmcli connection add con-name lan1 type ethernet ifname ens33 ipv4.method manual ipv4.addresses 192.168.100.10/24 ipv4.gateway 192.168.100.1 ipv4.dns 8.8.8.8 autoconnect yes

# যদি "Device is strictly unmanaged" এরর আসে, তাহলে এই কমান্ড ব্যবহার করে ম্যানুয়ালি কনফিগারেশন আপ করুন
nmcli connection up lan1

# ============================================================
# Docker Installation
# ============================================================

# *** Docker ইনস্টল করার জন্য ***
# প্রয়োজনীয় সার্টিফিকেট এবং টুলস ইনস্টল করা
sudo apt-get update
sudo apt-get install -y ca-certificates curl

# Docker এর অফিসিয়াল GPG কী যোগ করুন
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Docker এর রিপোজিটরি APT sources এ যোগ করুন
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# প্যাকেজ লিস্ট আপডেট এবং Docker ইনস্টল করা
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin

# Docker সার্ভিস সক্রিয় এবং শুরু করা
sudo systemctl enable docker
sudo systemctl start docker

# Docker ইনস্টলেশন যাচাই করা
sudo systemctl status docker

# উদাহরণ: একটি Docker কনটেইনার চালানো
sudo docker run -dit --name web1 nginx

# চলমান কনটেইনার যাচাই করা
sudo docker ps

# কনটেইনার পরিদর্শন করার জন্য
sudo docker inspect web1

# Nginx এর ডিফল্ট ওয়েলকাম পেজ যাচাই করতে কনটেইনারের IP এ curl চালান
sudo curl 172.17.0.2

# উদাহরণ: একটি কনটেইনার সরিয়ে ফেলা এবং ক্লিন আপ করা
sudo docker stop web1
sudo docker rm web1

# ============================================================
# Docker Networking
# ============================================================

# *** কাস্টম Docker নেটওয়ার্ক তৈরি করা ***
sudo docker network create customsnet1 --subnet 192.168.1.0/24

# নেটওয়ার্ক তৈরি যাচাই করা
sudo docker network ls

# উদাহরণ: কাস্টম নেটওয়ার্কে একটি কনটেইনার চালানো
sudo docker run -dit --name os1 --network customsnet1 ubuntu:latest

# কাস্টম নেটওয়ার্কে কনটেইনারের IP ঠিকানা যাচাই করা
sudo docker inspect os1 | grep IPAddress

# ============================================================
# Docker Copy (docker cp)
# ============================================================

# *** হোস্ট এবং কনটেইনারের মধ্যে ফাইল কপি করা ***
# উদাহরণ: হোস্ট থেকে কনটেইনারে ফাইল কপি করা
sudo docker cp index.html nginx:/usr/share/nginx/html

# উদাহরণ: কনটেইনার থেকে হোস্টে ফাইল কপি করা
sudo docker cp nginx:/usr/share/nginx/html/index.html ./

# ============================================================
# Kubernetes Cluster Setup
# ============================================================

# *** Kubernetes ক্লাস্টার সেটআপের জন্য Docker ইনস্টল করুন ***
# Master Node: kubeadm + kubelet + kubectl
# Worker Node: kubeadm + kubelet

# Kubernetes ক্লাস্টার ইনিশিয়ালাইজ করার উদাহরণ কমান্ড:
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# ইনিশিয়ালাইজেশনের পরে, ব্যবহারকারীর জন্য kubeconfig সেট আপ করা:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# বিকল্পভাবে, root ইউজার হিসাবে:
export KUBECONFIG=/etc/kubernetes/admin.conf

# ============================================================
# Fixing containerd Configuration Issue
# ============================================================

# *** containerd কনফিগারেশন সমস্যা সমাধান ***
# যদি kubeadm init এর সময় CRI runtime সম্পর্কিত কোনো ত্রুটি আসে, তাহলে containerd কনফিগারেশন পরিবর্তন করতে হতে পারে
# বিকল্প ১: containerd এর config.toml ফাইলটি মুছে ফেলুন
sudo rm /etc/containerd/config.toml

# বিকল্প ২: config.toml ফাইলে "disabled_plugins = ['cri']" লাইনটি মন্তব্য করে দিন
sudo vim /etc/containerd/config.toml
# disabled_plugins = ["cri"] লাইনটি কমেন্ট করে দিন

# পরিবর্তনগুলি প্রয়োগ করতে containerd সার্ভিস পুনরায় চালু করুন
sudo systemctl restart containerd.service

# ============================================================
# Flannel Network Plugin
# ============================================================

# Flannel নেটওয়ার্ক প্লাগিন প্রয়োগ করুন:
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# ============================================================
# Kubernetes Management
# ============================================================

# *** Kubernetes অবজেক্ট এবং ম্যানেজমেন্ট ***
# ক্লাস্টার নোডগুলি দেখুন:
kubectl get nodes

# সমস্ত namespaces এ সমস্ত পড দেখুন:
kubectl get pods -A

# উদাহরণ: একটি ডিপ্লয়মেন্ট তৈরি করা:
kubectl create deployment dep1 --replicas 10 --image nginx

# উদাহরণ: সমস্ত পড মুছে ফেলা:
kubectl delete pods --all

# ক্লাস্টারের স্থিতি যাচাই করুন:
kubectl get nodes -o wide
