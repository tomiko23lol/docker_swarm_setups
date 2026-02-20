# docker_swarm_setups

This repo is about documentation of my docker swarm setup, setup of nodes, services, networking and reasoning why I did it way I did.


Currently I am running with 3 VMs

| node | IP | role | type | host | specs |
|-----------|---------------|----------|--------|-----------|----------|
| `dockernode1` | `192.168.23.21` | manager, worker | VM | Proxmox01 | 2xCore 4G-8G_RAM 64GB_HDD |
| `dockernode2` | `192.168.23.22` | manager, worker | VM | xcmpng01 | 2xCore 4G-8G_RAM 64GB_HDD |
| `dockernode3` | `192.168.23.23` | manager | VM | TruenasScale N100 | 1xCore 1G_RAM 64GB_HDD |

Previously I had dockernode3 set up as worker as well, but when services migrated there, it was causing issues. Most probably some kind of issue with Truenas itself and how it does networking for VMs. That is why I degraded this node to manager only, to act just as another "whitners" for cluster to have quorum in cluster and prevent split-brain scenarios.


On top of that networking is enhanced with [KeepAlived](hosts/apps/keepalived/)
I am runing **Portainer** as orchestrator
Storage is replicated across all nodes by [GlusterFS](hosts/apps/glusterfs/)

