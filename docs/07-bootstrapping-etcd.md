# Bootstrapping the etcd Cluster

Kubernetes components are stateless and store cluster state in [etcd](https://github.com/etcd-io/etcd). In this lab you will bootstrap a single node etcd cluster.

> **What is etcd?** etcd is a distributed key-value store that provides a reliable way to store cluster state. It uses the Raft consensus algorithm to ensure consistency across multiple nodes. All Kubernetes objects (pods, services, etc.) are stored here.

## Prerequisites

Copy `etcd` binaries and systemd unit files to the `server` machine:

```bash
scp \
  downloads/controller/etcd \
  downloads/client/etcdctl \
  units/etcd.service \
  root@server:~/
```

The commands in this lab must be run on the `server` machine. Login to the `server` machine using the `ssh` command. Example:

```bash
ssh root@server
```

## Bootstrapping an etcd Cluster

### Install the etcd Binaries

Extract and install the `etcd` server and the `etcdctl` command line utility:

```bash
{
  mv etcd etcdctl /usr/local/bin/
}
```

### Configure the etcd Server

> **etcd data directory**: `/var/lib/etcd` stores the etcd database. The restrictive permissions (700) ensure only the etcd process can access cluster state. Regular backups of this directory are essential for disaster recovery.

```bash
{
  mkdir -p /etc/etcd /var/lib/etcd
  chmod 700 /var/lib/etcd
  cp ca.crt kube-api-server.key kube-api-server.crt \
    /etc/etcd/
}
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:

> **Single-node etcd**: While etcd is designed to run in a cluster (typically 3 or 5 nodes for high availability), we're using a single node for simplicity. Production clusters should run multiple etcd instances for fault tolerance.

Create the `etcd.service` systemd unit file:

```bash
mv etcd.service /etc/systemd/system/
```

### Start the etcd Server

```bash
{
  systemctl daemon-reload
  systemctl enable etcd
  systemctl start etcd
}
```

## Verification

List the etcd cluster members:

```bash
etcdctl member list
```

```text
6702b0a34e2cfd39, started, controller, http://127.0.0.1:2380, http://127.0.0.1:2379, false
```

Next: [Bootstrapping the Kubernetes Control Plane](08-bootstrapping-kubernetes-controllers.md)
