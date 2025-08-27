# Kubernetes

## Key Tools

- **KOPS**: Create and manage Kubernetes clusters.
- **KUBECTL**: Interact with Kubernetes clusters (typically the control plane).

---

## Kops Cluster Creation Commands

1. **Create Cluster**
  ```sh
  kops create cluster \
    --name=kube.haraapp.shop \
    --state=s3://kopsstate010 \
    --zones=us-east-1a,us-east-1b \
    --node-count=2 \
    --node-size=t3.small \
    --control-plane-size=t3.medium \
    --dns-zone=kube.haraapp.shop \
    --node-volume-size=12 \
    --control-plane-volume-size=12 \
    --ssh-public-key ~/.ssh/kube.pub
  ```
  - Stores cluster configuration in the specified S3 bucket.
  - **Node**: Worker node instance.
  - **Control plane**: Master node instance.
  - Adjust volume sizes as needed.
  - `dns-zone` should match your Route53 sub-domain.

2. **Update Cluster**
  ```sh
  kops update cluster --name=kube.haraapp.shop --state=s3://kopsstate010 --yes --admin
  ```
  - Applies the configuration from the S3 bucket.
  - `--yes`: Confirms and applies changes.
  - `--admin`: Generates kubeconfig with admin privileges.

3. **Validate Cluster**
  ```sh
  kops validate cluster --name=kube.haraapp.shop --state=s3://kopsstate010
  ```

4. **Delete Cluster**
  ```sh
  kops delete cluster --name=kube.haraapp.shop --state=s3://kopsstate010 --yes
  ```

---

- **Kubectl server version**: 1.32.3 (containerd runtime)
- **Default node OS**: Ubuntu 24.04.2 LTS
- **Kernel version**: 6.8.0-1029-aws
- **Container runtime**: containerd://1.7.25

---

## Kubernetes PATCH

### 1. Patching Kubernetes Resources

Use `kubectl patch` to update live resources (Deployments, Pods, ConfigMaps, etc.).

**Patch Strategies:**
- **Strategic Merge Patch**: Default, for nested fields.
- **JSON Patch**: Fine-grained (add, remove, replace).
- **JSON Merge Patch**: Simple key-value replacement.

**Examples:**

- Update container image:
  ```sh
  kubectl patch deployment my-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"my-container","image":"nginx:1.18"}]}}}}'
  ```
- Add a label:
  ```sh
  kubectl patch pod my-pod -p '{"metadata":{"labels":{"env":"production"}}}'
  ```
- Update replicas:
  ```sh
  kubectl patch deployment my-deployment -p '{"spec":{"replicas":5}}'
  ```
- Use a patch file:
  ```sh
  kubectl patch deployment my-deployment --patch-file=patch.yaml
  ```

---

### 2. Patching Node OS or Binaries

Nodes are managed via cloud-init or instance groups (e.g., AWS Auto Scaling Groups). To patch nodes:

1. Create a new AMI with updated OS and Kubernetes binaries.
2. Update the launch configuration/template in the instance group.

**Rolling Update Nodes:**
```sh
kops rolling-update cluster --yes
```
- Replaces old nodes with new ones using the updated AMI.

**Best Practices:**
- Use `kubectl patch` for quick changes.
- For infrastructure updates (security patches, kubelet upgrades), use Kops with new AMIs.
- Validate patches.
- Automate patching in CI/CD pipelines.

---

## Self-hosted Kubernetes: HTTP to HTTPS Redirection with NGINX Ingress

**Reference:**  
[Securing the Ingress using cert-manager (Medium)](https://medium.com/@muppedaanvesh/%EF%B8%8F-kubernetes-ingress-securing-the-ingress-using-cert-manager-part-7-366f1f127fd6)

**Steps:**
1. Create a cluster.
2. Create a self-signed certificate for your domain.
3. Deploy the ingress controller.
4. Create a namespace for your app.
5. Create deployment, service, and ingress resources.
6. Create a TLS secret:
  ```sh
  kubectl create secret tls gameapp-tls \
    --cert=/root/harapp_certificate/haraapp.shop.crt \
    --key=/root/harapp_certificate/haraapp.shop.key \
    -n game-2048
  ```
7. Use cert-manager to trust the self-signed certificate:
  - Install & update Helm.
  - Add the Cert-Manager Helm repository:
    ```sh
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    ```
  - Create a namespace for cert-manager:
    ```sh
    kubectl create namespace cert-manager
    ```
  - Install Cert-Manager:
    ```sh
    helm install cert-manager --namespace cert-manager --version v1.18.2 jetstack/cert-manager --set installCRDs=true
    ```
  - Create a Let’s Encrypt ClusterIssuer using a YAML file.

---

## AWS EKS

**Required tools:**
1. AWS CLI
2. EKSCTL: Create AWS EKS clusters (uses CloudFormation in the background).
3. KUBECTL
