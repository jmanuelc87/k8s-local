# K8s Local

This repo holds several projects to be deployed in my local cluster.

### Projects

1. [NFS-Server](./nfs-server/) for storing images and videos of my security cameras
2. [PI-Hole](./pi-hole/) for filtering ads
3. [Frigate](./frigate/) for accessing security cameras
4. [Kestra](./kestra/) for implementing data pipelines
5. [PostgreSQL](./postgresql/) for provisioning postgresql databases
6. [Talos](./talos/) for creating a talos k8s cluster, only the patches are stored.

#### MetalLB (Load Balancer)

MetalLB is **not** included in the base Helm chart because it requires `hostNetwork=true`, elevated capabilities (`NET_ADMIN`, `NET_RAW`, `SYS_ADMIN`), and root privileges for BGP/L2 routing. Instead, it is deployed to the standard `metallb-system` namespace with relaxed pod security policies.

**Installation:**

```bash
# Add and install metallb to the default system namespace
helm repo add metallb https://metallb.github.io/metallb
helm repo update
helm install metallb metallb/metallb \
  --namespace metallb-system \
  --create-namespace \
  --set crds.enabled=true

# Label the metallb-system namespace to allow privileged pod security
kubectl label namespace metallb-system \
  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/enforce-version=latest

# Configure IP address pool
kubectl apply -f ./metallb/config.yaml
```