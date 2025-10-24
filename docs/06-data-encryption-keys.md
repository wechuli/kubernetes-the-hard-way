# Generating the Data Encryption Config and Key

Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

> **Encryption at rest**: While TLS encrypts data in transit, encryption at rest protects data stored in etcd. This is critical if someone gains access to etcd backups or the underlying storage. Without this, Secrets are stored in base64 (easily decoded).

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key

Generate an encryption key:

> **Key generation**: This creates a 32-byte (256-bit) random key, which provides strong encryption. The key is base64-encoded for use in YAML configuration files. Store this key securely - losing it means encrypted data becomes unrecoverable.

```bash
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File

Create the `encryption-config.yaml` encryption config file:

```bash
envsubst < configs/encryption-config.yaml \
  > encryption-config.yaml
```

Copy the `encryption-config.yaml` encryption config file to each controller instance:

```bash
scp encryption-config.yaml root@server:~/
```

Next: [Bootstrapping the etcd Cluster](07-bootstrapping-etcd.md)
