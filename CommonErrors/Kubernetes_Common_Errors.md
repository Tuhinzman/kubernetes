# Common Errors While Creating a Kubernetes Cluster

Creating a Kubernetes cluster can be a complex process, and various issues may arise during the setup. This document outlines some common errors encountered while creating a Kubernetes cluster, along with explanations and potential solutions.

## 1. **Swap Not Disabled**

### Error:
```
[ERROR Swap]: running with swap on is not supported. Please disable swap.
```

### Cause:
Kubernetes requires that swap be disabled on all nodes.

### Solution:
Disable swap temporarily and permanently:
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

## 2. **Firewall Blocking Ports**

### Error:
Nodes are unable to join the cluster, or services are unreachable.

### Cause:
Kubernetes requires certain ports to be open for communication between nodes and components. If these ports are blocked by a firewall, the nodes will fail to communicate.

### Solution:
Open the necessary ports on all nodes:
```bash
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250:10252/tcp
sudo ufw allow 10255/tcp
sudo ufw allow 8472/udp
sudo ufw allow 443/tcp
sudo ufw allow 179/tcp
```
You may also want to disable the firewall during testing:
```bash
sudo ufw disable
```

## 3. **Mismatched Pod Network CIDR**

### Error:
Pods are unable to communicate with each other across nodes.

### Cause:
The pod network CIDR specified during `kubeadm init` does not match the CIDR configured in the network plugin (e.g., Flannel).

### Solution:
Ensure the CIDR used in `kubeadm init` matches the network plugin configuration. For example:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```
And ensure your network plugin (e.g., Flannel) is configured with the same CIDR.

## 4. **kubeadm init Error: Cannot create certificate signing request**

### Error:
```
[ERROR FileExisting-crictl]: crictl not found in system path
```

### Cause:
The `crictl` tool is not installed, which is required for the Kubernetes setup.

### Solution:
Install `crictl` on the node:
```bash
VERSION="v1.25.0"
curl -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
```

## 5. **Node Not Ready**

### Error:
```
NAME       STATUS     ROLES    AGE   VERSION
node1      NotReady   <none>   10m   v1.20.2
```

### Cause:
The node may not be correctly joined to the cluster, or it may not be able to communicate with the master.

### Solution:
- Check the node status:
  ```bash
  kubectl describe node <node-name>
  ```
- Ensure that the necessary ports are open and the node can communicate with the master.
- Check the Kubelet logs for more details:
  ```bash
  sudo journalctl -u kubelet -f
  ```

## 6. **ImagePullBackOff**

### Error:
```
STATUS: ImagePullBackOff
```

### Cause:
Kubernetes is unable to pull the specified image, either because the image name is incorrect or there is no access to the image repository.

### Solution:
- Verify the image name and tag.
- Ensure that the node has internet access to pull the image.
- If the image is private, ensure that the correct credentials are provided.

## 7. **coredns Pod Stuck in Pending State**

### Error:
```
coredns-6955765f44-n8q54    0/1     Pending   0          10m
```

### Cause:
The coredns pods may be stuck in the `Pending` state if there is an issue with the network plugin.

### Solution:
- Ensure the network plugin is correctly installed and configured.
- Check the logs of the coredns pod:
  ```bash
  kubectl logs -n kube-system <coredns-pod-name>
  ```

## 8. **kubectl Connection Error**

### Error:
```
The connection to the server <server-ip>:6443 was refused - did you specify the right host or port?
```

### Cause:
The `kubectl` command is unable to connect to the API server, possibly because the API server is down or the kubeconfig is not properly configured.

### Solution:
- Ensure the API server is running:
  ```bash
  sudo systemctl status kube-apiserver
  ```
- Check the kubeconfig file and ensure it points to the correct API server endpoint:
  ```bash
  kubectl config view
  ```

## 9. **CrashLoopBackOff**

### Error:
```
STATUS: CrashLoopBackOff
```

### Cause:
A pod is crashing repeatedly due to an error in the container, such as a misconfiguration or an application error.

### Solution:
- Check the logs of the crashing pod to diagnose the issue:
  ```bash
  kubectl logs <pod-name>
  ```
- Debug and resolve any application or configuration errors.

## 10. **Failed to Start ContainerManager**

### Error:
```
[ERROR ContainerManager]: failed to create system container: failed to create cgroup /kubepods: cgroups: cannot find cgroup mount destination: unknown
```

### Cause:
This error occurs when the required cgroup settings are not properly configured on the node.

### Solution:
- Ensure that the system is using `systemd` as the cgroup driver:
  ```bash
  sudo sed -i 's/cgroup-driver=systemd/cgroup-driver=systemd/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
  ```
- Restart the Kubelet service:
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl restart kubelet
  ```

## 11. **containerd Errors**

### Error:
```
Failed to connect to containerd: failed to dial "/run/containerd/containerd.sock": connection error
```

### Cause:
`containerd` may not be running, or there might be an issue with the `containerd` socket.

### Solution:
- Check the status of `containerd`:
  ```bash
  sudo systemctl status containerd
  ```
- If `containerd` is not running, start it:
  ```bash
  sudo systemctl start containerd
  ```
- If the issue persists, check the logs for more details:
  ```bash
  sudo journalctl -u containerd -f
  ```

### Error:
```
containerd: failed to load cni configuration file /etc/cni/net.d/10-containerd-net.conflist: invalid character '{' after top-level value
```

### Cause:
There may be an issue with the CNI configuration file, such as a syntax error or invalid JSON.

### Solution:
- Inspect the CNI configuration file for errors:
  ```bash
  sudo vi /etc/cni/net.d/10-containerd-net.conflist
  ```
- Correct any issues, save the file, and restart `containerd`:
  ```bash
  sudo systemctl restart containerd
  ```

## 12. **Docker Errors**

### Error:
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

### Cause:
Docker is not running, or the Docker socket is not accessible.

### Solution:
- Check the status of Docker:
  ```bash
  sudo systemctl status docker
  ```
- Start Docker if it is not running:
  ```bash
  sudo systemctl start docker
  ```

### Error:
```
docker: Error response from daemon: conflict: unable to delete <container_id> (must be forced) - container is paused.
```

### Cause:
A Docker container is paused and cannot be removed.

### Solution:
- Unpause the container:
  ```bash
  sudo docker unpause <container_id>
  ```
- Then remove the container:
  ```bash
  sudo docker rm <container_id>
  ```

## Conclusion

This document covers some of the most common errors you might encounter while creating a Kubernetes cluster, including issues related to `containerd` and Docker. By understanding and troubleshooting these issues, you can ensure a smoother setup process and a more stable Kubernetes environment.
