# Configuring kubectl for Remote Access

In this lab you will generate a kubeconfig file for the `kubectl` command line utility based on the `admin` user credentials.

> **Remote cluster access**: This configures kubectl to communicate with the Kubernetes cluster from the jumpbox. The same pattern applies to accessing clusters from your local workstation - you just need the right kubeconfig.

> Run the commands in this lab from the `jumpbox` machine.

## The Admin Kubernetes Configuration File

Each kubeconfig requires a Kubernetes API Server to connect to.

You should be able to ping `master.kubernetes.local` based on the `/etc/hosts` DNS entry from a previous lab.

```bash
curl --cacert ca.crt \
  https://master.kubernetes.local:6443/version
```

```text
{
  "major": "1",
  "minor": "34",
  "gitVersion": "v1.34.1",
  "gitCommit": "32cc146f75aad04beaaa245a7157eb35063a9f99",
  "gitTreeState": "clean",
  "buildDate": "2025-03-11T19:52:21Z",
  "goVersion": "go1.23.6",
  "compiler": "gc",
  "platform": "linux/arm64"
}
```

Generate a kubeconfig file suitable for authenticating as the `admin` user:

> **Kubeconfig structure**: A kubeconfig defines clusters (API server endpoints), users (credentials), and contexts (cluster + user pairs). You switch between different clusters by changing contexts.

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://master.kubernetes.local:6443

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key

  kubectl config set-context kubernetes-the-hard-way \
    --cluster=kubernetes-the-hard-way \
    --user=admin

  kubectl config use-context kubernetes-the-hard-way
}
```

The results of running the command above should create a kubeconfig file in the default location `~/.kube/config` used by the `kubectl` commandline tool. This also means you can run the `kubectl` command without specifying a config.

## Verification

Check the version of the remote Kubernetes cluster:

```bash
kubectl version
```

```text
Client Version: v1.34.1
Kustomize Version: v5.5.0
Server Version: v1.34.1
```

List the nodes in the remote Kubernetes cluster:

```bash
kubectl get nodes
```

```
NAME     STATUS   ROLES    AGE    VERSION
node01   Ready    <none>   10m   v1.34.1
node02   Ready    <none>   10m   v1.34.1
```

Next: [Provisioning Pod Network Routes](11-pod-network-routes.md)
