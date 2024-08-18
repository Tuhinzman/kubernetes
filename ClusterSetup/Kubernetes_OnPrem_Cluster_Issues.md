# Common Issues in On-Premises Kubernetes Clusters

Managing an on-premises Kubernetes cluster can present unique challenges, such as missing `.kube/config` files, services not starting on boot, and more. This document outlines common issues and provides solutions to ensure your cluster runs smoothly after a reboot or other disruptions.

## 1. **Missing `.kube/config` File**

### Issue:
After restarting the master node, the `.kube/config` file is missing or not found, leading to `kubectl` commands failing to connect to the cluster.

### Cause:
The `.kube/config` file may not have been persisted, or it was accidentally deleted.

### Solution:

1. **Regenerate the `.kube/config` file**:
    - You can recreate the `.kube/config` file by copying the `admin.conf` file to the appropriate location.
    - On the master node:
      ```bash
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      ```

2. **Backup `.kube/config`**:
    - To prevent future issues, it's a good idea to back up your `.kube/config` file:
      ```bash
      cp $HOME/.kube/config $HOME/.kube/config.backup
      ```

## 2. **Services Not Starting on Boot**

### Issue:
After a machine restart, essential services like `containerd`, `kubelet`, or Docker do not start automatically.

### Cause:
These services may not be enabled to start at boot, or there might be issues with systemd configuration.

### Solution:

1. **Enable Services to Start on Boot**:
    - Ensure that all necessary services are enabled to start on boot:
      ```bash
      sudo systemctl enable kubelet
      sudo systemctl enable containerd
      sudo systemctl enable docker
      ```

2. **Check and Start Services Manually**:
    - After a reboot, check the status of the services and start them if they are not running:
      ```bash
      sudo systemctl status kubelet
      sudo systemctl status containerd
      sudo systemctl status docker
      ```
      - If any service is not running, start it manually:
        ```bash
        sudo systemctl start kubelet
        sudo systemctl start containerd
        sudo systemctl start docker
        ```

3. **Ensure Proper Systemd Configuration**:
    - Sometimes, systemd unit files may need adjustments. Verify that the `systemd` configuration for each service is correct.
    - For example, for `kubelet`:
      ```bash
      sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
      ```
    - Ensure the following line exists to use `systemd` as the cgroup driver:
      ```
      Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
      ```
    - Reload the systemd daemon and restart the service:
      ```bash
      sudo systemctl daemon-reload
      sudo systemctl restart kubelet
      ```

4. **Automate Checks with a Script**:
    - You can create a script that checks and starts these services at boot, and place it in the `/etc/rc.local` file (or create a systemd service for it):
      ```bash
      #!/bin/bash
      sudo systemctl start kubelet
      sudo systemctl start containerd
      sudo systemctl start docker
      ```
    - Ensure the script is executable:
      ```bash
      sudo chmod +x /etc/rc.local
      ```

## 3. **Persisting Configuration and Data**

### Issue:
After a restart, configurations and data related to Kubernetes services are lost, leading to an unstable cluster.

### Cause:
Configuration files or important data may not be stored in a persistent manner, causing them to be lost on reboot.

### Solution:

1. **Persist Important Configuration Files**:
    - Ensure that configuration files, especially those in `/etc/kubernetes`, are persisted and backed up regularly.
    - Use a version control system (e.g., Git) to track changes to configuration files.

2. **Use Persistent Volumes**:
    - For critical data that needs to survive reboots, use Persistent Volumes (PVs) in Kubernetes. Ensure that these PVs are properly configured and backed by reliable storage.

3. **Regular Backups**:
    - Regularly back up important directories like `/etc/kubernetes`, `/var/lib/etcd`, and any custom scripts or configurations you rely on.
    - Example using `tar`:
      ```bash
      sudo tar -czvf kubernetes_backup_$(date +%F).tar.gz /etc/kubernetes /var/lib/etcd
      ```

## Conclusion

Managing an on-premises Kubernetes cluster can introduce challenges, especially related to service persistence and configuration management. By following the solutions outlined in this document, you can mitigate common issues and ensure your cluster remains stable and reliable, even after reboots.
