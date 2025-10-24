# Bootstrapping the Kubernetes Worker Nodes

In this lab you will bootstrap two Kubernetes worker nodes. The following components will be installed: [runc](https://github.com/opencontainers/runc), [container networking plugins](https://github.com/containernetworking/cni), [containerd](https://github.com/containerd/containerd), [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet), and [kube-proxy](https://kubernetes.io/docs/concepts/cluster-administration/proxies).

> **Worker node stack**: Worker nodes run your application containers. The stack includes: containerd (container runtime), runc (low-level container executor), CNI plugins (networking), kubelet (node agent), and kube-proxy (network proxy for Services).

## Prerequisites

The commands in this section must be run from the `jumpbox`.

Copy the Kubernetes binaries and systemd unit files to each worker instance:

```bash
for HOST in node01 node02; do
  SUBNET=$(grep ${HOST} machines.txt | cut -d " " -f 4)
  sed "s|SUBNET|$SUBNET|g" \
    configs/10-bridge.conf > 10-bridge.conf

  sed "s|SUBNET|$SUBNET|g" \
    configs/kubelet-config.yaml > kubelet-config.yaml

  scp 10-bridge.conf kubelet-config.yaml \
  root@${HOST}:~/
done
```

```bash
for HOST in node01 node02; do
  scp \
    downloads/worker/* \
    downloads/client/kubectl \
    configs/99-loopback.conf \
    configs/containerd-config.toml \
    configs/kube-proxy-config.yaml \
    units/containerd.service \
    units/kubelet.service \
    units/kube-proxy.service \
    root@${HOST}:~/
done
```

```bash
for HOST in node01 node02; do
  scp \
    downloads/cni-plugins/* \
    root@${HOST}:~/cni-plugins/
done
```

The commands in the next section must be run on each worker instance: `node01`, `node02`. Login to the worker instance using the `ssh` command. Example:

```bash
ssh root@node01
```

## Provisioning a Kubernetes Worker Node

Install the OS dependencies:

```bash
{
  apt-get update
  apt-get -y install socat conntrack ipset kmod
}
```

> **OS dependencies explained**: `socat` enables kubectl port-forward, `conntrack` tracks network connections for kube-proxy, `ipset` efficiently stores IP addresses for network policies, and `kmod` manages kernel modules needed for networking.

Disable Swap

> **Why disable swap?** Kubernetes has limited support for the use of swap memory, as it is difficult to provide guarantees and account for pod memory utilization when swap is involved. Swap can hide memory pressure issues and make performance unpredictable.

Verify if swap is disabled:

```bash
swapon --show
```

If output is empty then swap is disabled. If swap is enabled run the following command to disable swap immediately:

```bash
swapoff -a
```

> To ensure swap remains off after reboot consult your Linux distro documentation.

Create the installation directories:

```bash
mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Install the worker binaries:

```bash
{
  mv crictl kube-proxy kubelet runc \
    /usr/local/bin/
  mv containerd containerd-shim-runc-v2 containerd-stress /bin/
  mv cni-plugins/* /opt/cni/bin/
}
```

### Configure CNI Networking

Create the `bridge` network configuration file:

> **CNI (Container Network Interface)**: CNI plugins handle pod networking. The bridge plugin creates a Linux bridge on each node, connecting containers to the network. This enables pod-to-pod communication within and across nodes.

```bash
mv 10-bridge.conf 99-loopback.conf /etc/cni/net.d/
```

To ensure network traffic crossing the CNI `bridge` network is processed by `iptables`, load and configure the `br-netfilter` kernel module:

> **br-netfilter module**: This kernel module enables iptables to see bridged traffic. Without it, traffic between containers on the same node would bypass iptables rules, breaking network policies and Services.

```bash
{
  modprobe br-netfilter
  echo "br-netfilter" >> /etc/modules-load.d/modules.conf
}
```

```bash
{
  echo "net.bridge.bridge-nf-call-iptables = 1" \
    >> /etc/sysctl.d/kubernetes.conf
  echo "net.bridge.bridge-nf-call-ip6tables = 1" \
    >> /etc/sysctl.d/kubernetes.conf
  sysctl -p /etc/sysctl.d/kubernetes.conf
}
```

### Configure containerd

> **containerd's role**: containerd is the container runtime that manages the container lifecycle (pull images, create/start/stop containers). It replaced Docker as Kubernetes' default runtime for better integration and resource efficiency.

Install the `containerd` configuration files:

```bash
{
  mkdir -p /etc/containerd/
  mv containerd-config.toml /etc/containerd/config.toml
  mv containerd.service /etc/systemd/system/
}
```

### Configure the Kubelet

> **Kubelet**: The kubelet is the primary node agent. It registers the node with the API server, watches for pod assignments, ensures assigned pods are running and healthy, and reports node/pod status back to the control plane.

Create the `kubelet-config.yaml` configuration file:

```bash
{
  mv kubelet-config.yaml /var/lib/kubelet/
  mv kubelet.service /etc/systemd/system/
}
```

### Configure the Kubernetes Proxy

> **kube-proxy function**: kube-proxy implements Kubernetes Services by maintaining network rules (iptables or IPVS) on each node. It enables load balancing across pod backends and provides a stable IP/DNS for accessing services.

```bash
{
  mv kube-proxy-config.yaml /var/lib/kube-proxy/
  mv kube-proxy.service /etc/systemd/system/
}
```

### Start the Worker Services

```bash
{
  systemctl daemon-reload
  systemctl enable containerd kubelet kube-proxy
  systemctl start containerd kubelet kube-proxy
}
```

Check if the kubelet service is running:

```bash
systemctl is-active kubelet
```

```text
active
```

Be sure to complete the steps in this section on each worker node, `node01` and `node02`, before moving on to the next section.

## Verification

Run the following commands from the `jumpbox` machine.

List the registered Kubernetes nodes:

```bash
ssh root@server \
  "kubectl get nodes \
  --kubeconfig admin.kubeconfig"
```

```
NAME     STATUS   ROLES    AGE    VERSION
node01   Ready    <none>   1m     v1.34.1
node02   Ready    <none>   10s    v1.34.1
```

Next: [Configuring kubectl for Remote Access](10-configuring-kubectl.md)
