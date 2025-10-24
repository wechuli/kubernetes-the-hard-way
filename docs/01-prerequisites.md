# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Virtual or Physical Machines

This tutorial requires four (4) virtual or physical ARM64 or AMD64 machines running Debian 12 (bookworm). The following table lists the four machines and their CPU, memory, and storage requirements.

> **Why these machines?** Kubernetes is a distributed system requiring separate machines for administration (jumpbox), control plane (server), and worker nodes. This separation mirrors production architectures and helps you understand the communication patterns between components.

| Name    | Description             | CPU | RAM   | Storage |
|---------|-------------------------|-----|-------|---------|
| jumpbox | Administration host     | 1   | 512MB | 10GB    |
| master  | Kubernetes server       | 1   | 2GB   | 20GB    |
| node01  | Kubernetes worker node  | 1   | 2GB   | 20GB    |
| node02  | Kubernetes worker node  | 1   | 2GB   | 20GB    |

How you provision the machines is up to you, the only requirement is that each machine meet the above system requirements including the machine specs and OS version. 

> **OS Selection**: Debian 12 provides a stable, well-documented base with long-term support. Using a consistent OS across all nodes simplifies troubleshooting and ensures compatibility with Kubernetes binaries.

Once you have all four machines provisioned, verify the OS requirements by viewing the `/etc/os-release` file:

```bash
cat /etc/os-release
```

You should see something similar to the following output:

```text
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
VERSION="12 (bookworm)"
VERSION_CODENAME=bookworm
ID=debian
```

Next: [setting-up-the-jumpbox](02-jumpbox.md)
