# K8s Local

This repo holds several projects to be deployed in my local cluster.

### Projects

1. [Base](./base/) for the cluster foundations: storage (Longhorn, NFS) and metrics-server.
2. [csi-driver-nfs](./csi-driver-nfs/) for installing the NFS CSI driver in the cluster.
3. [Frigate](./frigate/) for accessing security cameras.
4. [MetalLB](./metallb/) for the load balancer IP address pool configuration.
5. [PI-Hole](./pi-hole/) for filtering ads.
6. [PostgreSQL](./postgresql/) for provisioning postgresql databases.
7. [Talos](./talos/) for creating a talos k8s cluster, only the patches are stored.

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
  pod-security.kubernetes.io/audit=privileged \
  pod-security.kubernetes.io/warn=privileged \
  pod-security.kubernetes.io/enforce-version=latest

# Configure IP address pool
kubectl apply -f ./metallb/config.yaml
```

#### NFS CSI driver (dynamic NFS volumes)

The `nfs-csi` StorageClass in the base chart needs the
[csi-driver-nfs](https://github.com/kubernetes-csi/csi-driver-nfs) driver. It is
**not** a dependency of the base chart because its node DaemonSet needs
`hostNetwork`, `privileged` and hostPath mounts, and Talos enforces the
PodSecurity `baseline` profile on every namespace except `kube-system`. A Helm
subchart can only deploy into the release namespace, so the driver is installed
as its own release in `kube-system` instead.

**Installation:**

```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm repo update
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
  --namespace kube-system \
  --version 4.13.4 \
  -f ./csi-driver-nfs/values.yaml
```

Claims on the `nfs-csi` class get their own subdirectory on the NFS server's
cold-storage share.
