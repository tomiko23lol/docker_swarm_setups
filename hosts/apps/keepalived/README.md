# Keepalived – Virtual Floating IPs for Docker Nodes

This document describes how **Keepalived** is used on my Docker node VMs to provide
**stable virtual (floating) IP addresses** that remain reachable regardless of
which node is currently active.

The goal is to decouple **DNS records and external access** from individual nodes.

---

## What Is Keepalived

Keepalived is a Linux service that implements **VRRP (Virtual Router Redundancy Protocol)**.

It allows multiple machines on the same network to:
- Share one or more **virtual IP addresses**
- Elect a **master node** for each virtual IP
- Automatically **fail over** if the master goes offline

To the outside world, the virtual IP behaves like a normal host.

---

## What Keepalived Is Used For in This Setup

In this setup, Keepalived is used to:

- Provide **stable IP addresses** for services running in a Docker Swarm
- Avoid dependency on a specific node being online for deployed service to be available
- Allow DNS records to always point to the same IP no matter on which host it is running on
- Enable clean failover without manual intervention

---

## How Keepalived Works (High-Level)

1. Each node runs Keepalived
2. Nodes advertise their **priority** for a given virtual IP
3. The node with the highest priority becomes **MASTER**
4. The MASTER owns the virtual IP
5. If the MASTER fails, another node automatically takes over

This happens in seconds and requires no client-side changes.

---

## My Cluster Layout

### Physical Nodes

| Node | IP Address |
|----|-----------|
| dockernode1 | `192.168.23.21` |
| dockernode2 | `192.168.23.22` |
| dockernode3 | `192.168.23.23` |

---

### Virtual IPs

| Virtual IP | Preferred Node |
|-----------|---------------|
| `192.168.23.101` | dockernode1 (`.21`) |
| `192.168.23.102` | dockernode2 (`.22`) |
| `192.168.23.103` | dockernode3 (`.23`) |

Each virtual IP has:
- A **primary (preferred) node**
- Automatic failover to the other nodes if needed

---

## Installation

### Step 1: Debian / Ubuntu install

```bash
sudo apt update
sudo apt install -y keepalived

```

### Step 2: Configure Keepalived
Create or edit the Keepalived configuration file on each node (usually located at `/etc/keepalived/keepalived.conf`). Below is a sample configuration for two nodes (Node 1 and Node 2).

#### Example Configuration for Node 1:

```bash
vrrp_instance VI_1 {
    state MASTER
    interface <your-network-interface>  # e.g., eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass your_password
    }
    virtual_ipaddress {
        192.168.23.101
    }
}


```
#### Example Configuration for Node 2:

```bash
vrrp_instance VI_1 {
    state BACKUP
    interface <your-network-interface>  # e.g., eth0
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass your_password
    }
    virtual_ipaddress {
        192.168.23.101
    }
}

```


###  Step 3: Start Keepalived
After configuring Keepalived on both nodes, start the service:

```bash
sudo systemctl start keepalived
sudo systemctl enable keepalived

```



### Step 5: Verify the Setup

#### Check Keepalived Status:
On both nodes, check if Keepalived is running and which node is currently the MASTER.

```bash
sudo systemctl status keepalived
```
- Test Connectivity:
From another machine on your local network, try to connect to some service in cluster using the virtual IP (`192.168.23.101`):

```bash
mosquitto_sub -h 192.168.23.101 -t test/topic
```
 - Simulate Failover:
To test failover, stop Keepalived on the MASTER node:

```bash
sudo systemctl stop keepalived
```
The BACKUP node should take over and respond to requests at `192.168.23.101`.



Here are actual configs that I have running on my docker nodes:


