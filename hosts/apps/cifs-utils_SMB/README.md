# cifs-utils

`cifs-utils` is a set of Linux utilities used to mount and manage **SMB / CIFS network shares**.
It enables Linux systems to access remote storage hosted on Windows servers, NAS devices,
or Samba shares as if they were local filesystems.

---

## What Is cifs-utils

`cifs-utils` provides the tools required to:
- Mount SMB (Server Message Block) network shares
- Authenticate using usernames, passwords, or credentials files
- Integrate network storage into the Linux filesystem

It is most commonly used with the `mount.cifs` command.

---

## How to Install cifs-utils

### Debian / Ubuntu install

```bash
sudo apt update
sudo apt install -y cifs-utils

```


## How to Use cifs-utils

### Create a Mount Point

```bash
sudo mkdir -p /mnt/MyNasMount

```

### Mount an SMB Share (Manual)
 
```bash
sudo mount -t cifs //SERVER/SHARE /mnt/MyNasMount \
  -o username=SMBUSER,password=SMBPASSWORD,iocharset=utf8,vers=3.1.1

```


### Persistent Mount Using /etc/fstab (Recommended)


```bash
sudo nano /root/.smbcredentials

username=SMBUSER
password=SMBPASSWORD

```
#### Add to /etc/fstab:

```bash
//SERVER/SHARE /mnt/backups cifs credentials=/root/.smbcredentials,iocharset=utf8,vers=3.1.1 0 0

```
#### mount all filesystems

```bash
sudo mount -a
```




## How I Use cifs-utils in My Setup

### In my infrastructure:
-	All Docker nodes mount the same SMB share at `/mnt/nas_Install`
-	The mount is configured via `/etc/fstab`
-	**cifs-utils** ensures the share is available before containers start
-	Docker Swarm services can use the mounted path as persistent storage

### This allows:
-	Backups to be written to shared external storage
-	Containers to remain stateless
-	Backup data to survive node failures
