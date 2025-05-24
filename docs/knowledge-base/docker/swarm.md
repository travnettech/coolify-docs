---
title: "Docker Swarm"
description: "A guide on how to use Docker Swarm with Coolify."
---

# Docker Swarm

::: danger Caution
***This is an experimental feature.***
:::

## Setup in Coolify

If you want to deploy resources to a Docker Swarm, you must:

1. **Add the Swarm Manager** to Coolify.  
2. *(Optional)* **Add Swarm Workers** to Coolify – this lets Coolify perform clean-ups and other maintenance tasks on those nodes.

### Docker Registry

A **registry external to the Swarm** is required so that every worker can pull the images Coolify builds.

- The Swarm Manager **pushes** the image to the registry.  
- The Swarm Workers **pull** the image from the registry.

Configure your Docker credentials accordingly. More details [here](/knowledge-base/docker/registry).

---

## Install a Swarm Cluster (quick guide)

> **WIP** – For comprehensive instructions, see the  
> [Docker Swarm documentation](https://docs.docker.com/engine/swarm/).

### 1  Prerequisites

- Example provider: [Hetzner](https://coolify.io/hetzner) (referral link) – any provider works.  
- **Three or more servers** with the **same architecture** (ARM or AMD64):  
  - **1 manager node**  
  - **2 worker nodes** (add more if needed)  
- Enable **private networking** on every server if possible.

### 2  Install Docker on every server

Follow the [official install guide](https://docs.docker.com/engine/install/) or run **one** of the scripts below:

```bash
# Option A – Rancher script
curl https://releases.rancher.com/install-docker/24.0.sh | sh

# Option B – Docker's official script
curl -fsSL https://get.docker.com | sh -s -- --version 24.0
```

### 3  Enable and configure the Docker daemon

Start Docker and enable it to start on boot:

```bash
systemctl start docker
systemctl enable docker
```

::: warning Caution
**Hetzner-specific MTU fix**  
Hetzner uses MTU 1450. Set Docker’s MTU on **all nodes**:

```bash
mkdir -p /etc/docker
cat <<EOF > /etc/docker/daemon.json
{
  "mtu": 1450
}
EOF

systemctl restart docker
```
:::

### 4  Create the Swarm (manager node)

```bash
# Replace <MANAGER_IP> with the manager’s private IP (e.g. 10.0.0.2)
docker swarm init --advertise-addr <MANAGER_IP>
```

Docker prints a **join command**. Example (do **not** copy verbatim):

```bash
docker swarm join --token SWMTKN-1-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 10.0.0.2:2377
```

### 5  Join worker nodes

Run the printed `docker swarm join ...` command on **each worker node**.

### 6  Verify the cluster (manager node)

```bash
docker node ls
```

Example output:

```bash
ID                            HOSTNAME          STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
ua38ijktbid70em257ymxufif *   swarm-manager     Ready     Active         Leader           24.0.2
7rss9rvaqpe9fddt5ol1xucmu     swarm-worker-1    Ready     Active                          24.0.2
12239rvaqp43gddtgfsdxucm2     swarm-worker-2    Ready     Active                          24.0.2
```

---

## Deploying services with persistent storage

Swarm can reschedule a service onto any worker. To avoid data loss you need **shared storage** accessible from **every worker**:

- **AWS EFS**  
- **NFS** server  
- **GlusterFS** cluster  
- Any other storage solution supported by Docker volumes

> **WIP** – A detailed volume-setup guide is in progress.
