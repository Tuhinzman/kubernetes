# Deploying Flannel Network Plugin with Custom CIDR

This document provides step-by-step instructions for deploying the Flannel network plugin in a Kubernetes cluster and configuring it with a custom CIDR.

## Prerequisites

- Kubernetes cluster initialized (either with `kubeadm` or another method).
- `kubectl` configured to interact with the cluster.
- Basic understanding of YAML files and Kubernetes networking.

## 1. Download the Flannel YAML Configuration File

### Purpose:
The Flannel YAML file contains the necessary configurations to deploy the Flannel network plugin across your Kubernetes cluster.

1. **Download the Flannel YAML file**:
    ```bash
    wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
    ```

    This command downloads the latest Flannel configuration file to your working directory.

## 2. Modify the CIDR in the Flannel Configuration

### Purpose:
To customize the pod network CIDR to match the one you specified during Kubernetes cluster initialization.

1. **Open the Flannel YAML file in a text editor**:
    ```bash
    vi kube-flannel.yml
    ```

2. **Locate the `net-conf.json` Section**:
    - Scroll down to find the `ConfigMap` section with the name `kube-flannel-cfg`.
    - Within this section, locate the following block:
      ```yaml
      net-conf.json: |
        {
          "Network": "10.244.0.0/16", # Default CIDR used by Flannel
          "EnableNFTables": false,
          "Backend": {
            "Type": "vxlan"
          }
        }
      ```

3. **Change the CIDR to Your Desired Range**:
    - Replace `10.244.0.0/16` with your desired CIDR. For example, if you want to use `192.168.0.0/16`, modify the block as follows:
      ```yaml
      net-conf.json: |
        {
          "Network": "192.168.0.0/16", # Changed CIDR to 192.168.0.0/16
          "EnableNFTables": false,
          "Backend": {
            "Type": "vxlan"
          }
        }
      ```

4. **Save and Exit the Editor**:
    - After making the changes, save the file and exit the editor.

    - In `vi`, you can do this by pressing `Esc` and typing `:wq`, then hitting `Enter`.

## 3. Apply the Flannel Network Configuration

### Purpose:
To deploy the Flannel network plugin using the customized configuration.

1. **Apply the Flannel YAML configuration**:
    ```bash
    kubectl apply -f kube-flannel.yml
    ```

    This command applies the Flannel configuration to your Kubernetes cluster, setting up the networking layer according to the specifications in the YAML file.

2. **Verify the Deployment**:
    - Ensure that all Flannel pods are running and the nodes are ready:
      ```bash
      kubectl get pods -n kube-flannel -o wide
      kubectl get nodes
      ```

    - The output should show all Flannel pods in the `Running` state and all nodes in the `Ready` state.

## 4. Initialize Kubernetes with Matching CIDR (if not already done)

### Purpose:
When initializing your Kubernetes cluster, the pod network CIDR must match the CIDR specified in the Flannel configuration.

1. **Initialize the Kubernetes master node with the custom CIDR**:
    ```bash
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    ```

    - This command initializes your Kubernetes cluster with the `192.168.0.0/16` pod network CIDR, matching the one specified in the Flannel configuration.

## 5. Troubleshooting

- **Pods are not able to communicate**:
  - Ensure the CIDR specified during the Kubernetes initialization matches the one in the Flannel configuration.
  - Verify that the Flannel pods are running using:
    ```bash
    kubectl get pods -n kube-flannel
    ```
  - Check the logs of any problematic Flannel pod:
    ```bash
    kubectl logs <flannel-pod-name> -n kube-flannel
    ```

- **Nodes are not in Ready state**:
  - Ensure the network configuration is correct and that there are no conflicts with other networking solutions or firewalls.
  - Restart the kubelet on affected nodes:
    ```bash
    sudo systemctl restart kubelet
    ```

## Conclusion

By following these steps, you have successfully deployed the Flannel network plugin with a custom pod network CIDR in your Kubernetes cluster. Ensure that the CIDR used during cluster initialization matches the CIDR in the Flannel configuration for seamless networking.
