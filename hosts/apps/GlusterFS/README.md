# GlusterFS

[GlusterFS](https://gluster.org/) is a distributed filesystem that allows multiple servers to
share storage and present it as a **single, unified filesystem**.

It is designed for redundancy, scalability, and high availability.

---

## What Is GlusterFS

GlusterFS aggregates storage from multiple nodes into a single volume.
Files are replicated or distributed across nodes based on the chosen volume type.

Key features:
- Distributed storage across multiple machines
- Replication for redundancy
- No single point of failure
- POSIX-compliant filesystem

---

## How to Install GlusterFS

here is [generic_quick_start_guide](https://docs.gluster.org/en/latest/Quick-Start-Guide/Quickstart/)

### Debian / Ubuntu install:

```bash
sudo apt update
sudo apt install -y glusterfs-server

```

### Enable and start the service:

```bash
sudo systemctl enable glusterd
sudo systemctl start glusterd
```

### Verify:

```bash
gluster peer status
```

## How to use

### prepare brick on each node

```bash
sudo mkdir -p /gluster/gvolume1/
```

### Peer the Nodes

Run this on one node only ( node1 in this case )

```bash
gluster peer probe node2
gluster peer probe node3
```

Verify

```bash
gluster peer status
```
All nodes should show as Peer in Cluster.

### Create a Replicated Volume

```bash
gluster volume create staging-gfs replica 3 \
  node1:/gluster/gvolume1/ \
  node2:/gluster/gvolume1/ \
  node3:/gluster/gvolume1/
```

### Star the volume

```bash
gluster volume start staging-gfs
```

### Mount volume (persistent)
edit `/etc/fstab` on echa node and add:

```bash
localhost:/staging-gfs /mnt glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0
```
This way you will use entire `/mnt` as gluster and it will replicate everything on `/mnt` to other nodes. this is how I use it.
Little bit better aproach would be to use just one mount `/mnt/gluster` as replicated mount. It is cleaner way, but I am already doint it with entire /mnt and I dont want to rewrite all scripts and 
stacks to accomodate the change. So ideal mount wuld be like this:

```bash
localhost:/staging-gfs /mnt/gluster glusterfs defaults,_netdev,backupvolfile-server=localhost 0 0
```




## Final thoughts

So Far GlusterFS set up as described above worked for me very well. There where no issues, it works realiably and I dont need to worry or think about it too much. It was set up one and since than I am just working with `/mnt` as normaly,
and it is doing replication for me.
