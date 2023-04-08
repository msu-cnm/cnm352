## mount - Mounting a Storage Device

Mount a local device

```bash
# Ex. 
sudo mount /dev/sdb	/mnt/<MOUNTDIR>
```

Mount an NFS export (share) over the network

```bash
# Ex.
sudo mount -o nolock 10.11.1.72:/REMOTEDIR ~/LOCALDIR/

# Options
-o		# Specifies options
		nolock # species not to lock the filesystem using NLM sideband.  You MUST use this with NFS shares
		nfsvers=3 # Specifies the NFS version to mount with
```

Unmount a device

```bash
sudo umount ~/LOCALDIR/
```

Show all mounts

```bash
cat /proc/mounts
```

