# Mealie-k3s 🍳

A self-contained Helm chart to deploy the [Mealie](https://docs.mealie.io/) recipe manager on Kubernetes (k3s, k8s, microk8s) with a single command.

This chart bundles:
* **Mealie**: The application itself.
* **PostgreSQL**: A persistent database backend using the [Bitnami/postgresql](https://github.com/bitnami/charts/tree/main/bitnami/postgresql) chart.
* **MetalLB**: Automatic `LoadBalancer` IP assignment for bare-metal clusters.

## Prerequisites

1.  **A Kubernetes Cluster**: This guide assumes you have a cluster running. For quick local setup, see the **MicroK8s** instructions below.
2.  **Helm**: The Kubernetes package manager. [Install Helm](https://helm.sh/docs/intro/install/).
3.  **kubectl**: The Kubernetes command-line tool, configured to access your cluster.

---

## Option A: Quick Cluster Setup (MicroK8s)

If you don't have a cluster, [MicroK8s](https://microk8s.io/) is a lightweight, easy-to-install option.

1.  **Install MicroK8s**:
    ```bash
    sudo snap install microk8s --classic
    ```

2.  **Join User Group**:
    ```bash
    sudo usermod -a -G microk8s $USER
    sudo chown -f -R $USER ~/.kube
    newgrp microk8s
    ```

3.  **Check Status**:
    ```bash
    microk8s status --wait-ready
    ```

4.  **Enable Core Addons**:
    ```bash
    microk8s enable dns storage
    ```
    * **Note**: Do **NOT** enable `microk8s enable metallb`. This chart deploys MetalLB via Helm, which is the preferred way, to manage it as part of the application stack.

---

## Option B: Existing k3s/k8s Cluster

If you have an existing cluster, just ensure `kubectl` is configured and you have Helm installed.

---

## 🚀 Deployment

1.  **Clone This Repository**:
    ```bash
    git clone [https://github.com/YOUR-USERNAME/mealie-k3s.git](https://github.com/YOUR-USERNAME/mealie-k3s.git)
    cd mealie-k3s
    ```

2.  **Add Helm Repositories**:
    This adds the Bitnami and MetalLB chart repositories so Helm can find the dependencies.
    ```bash
    helm repo add bitnami [https://charts.bitnami.com/bitnami](https://charts.bitnami.com/bitnami)
    helm repo add metallb [https://metallb.github.io/metallb](https://metallb.github.io/metallb)
    helm repo update
    ```

3.  **Build Chart Dependencies**:
    This downloads the `postgresql` and `metallb` charts into the `chart/charts` directory.
    ```bash
    helm dependency build ./chart
    ```

4.  **‼️ IMPORTANT: Configure `values.yaml`**:
    Before you install, you **must** edit the configuration file:
    
    `nano ./chart/values.yaml`

    * **MetalLB IP Range**: Under `metallb.configInline.address-pools.addresses`, change the IP range (`"192.168.1.240-192.168.1.250"`) to a free, static IP range on your local network. This range must *not* be part of your router's DHCP pool.
    * **Passwords**: Change the default passwords under `postgresql.auth`.

5.  **Install the Chart**:
    This command deploys Mealie, PostgreSQL, and MetalLB into a new namespace called `mealie`.
    
    ```bash
    helm install mealie ./chart --namespace mealie --create-namespace
    ```

## Accessing Mealie

1.  **Wait for Deployment**:
    It may take 3-5 minutes for the database to initialize and the application to start. You can watch the pods:
    ```bash
    kubectl get pods -n mealie -w
    ```

2.  **Find Your IP Address**:
    Once the pods are running, find the external IP assigned by MetalLB:
    ```bash
    kubectl get svc -n mealie
    ```
    
    Look for `mealie-mealie-svc` and copy the `EXTERNAL-IP`.

3.  **Log In**:
    Open the `EXTERNAL-IP` in your browser. The default Mealie admin credentials are:
    * **Email**: `changeme@example.com`
    * **Password**: `MyPassword`

## Uninstalling

To remove the deployment and all its data:
```bash
helm uninstall mealie -n mealie
kubectl delete namespace mealie

## LICENSE
GNU GENERAL PUBLIC LICENSE v3