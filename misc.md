# Misc

## Fix Keepass2 segfault on Ubuntu
```bash
sudo sed -i "s/cli/cli --verify-all/" $(which keepass2)
```

## Mount NFS during the boot safely

The system will not hang up if there is no connection
to the NFS server. Put the following line into `/etc/fstab`.

```bash
192.168.1.100:/nfsshare /mnt/nfsshare nfs auto,nofail,x-systemd.device-timeout=30s,noatime,nolock,intr,tcp,actimeo=1800,_netdev 0 0
```
