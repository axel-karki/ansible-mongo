## Initialization

### AVX Support for MongoDB

MongoDB requires AVX instruction set support. CPUs using `x86-64-v2-AES` do **not** support AVX in Proxmox VM.

To verify AVX support:

```bash
lscpu | grep -o avx
```

If no output is returned, AVX is not supported.

To enable AVX support for a VM in Proxmox, update the CPU configuration:

```bash
nano /etc/pve/qemu-server/<vmid>.conf
# Change this line:
cpu: x86-64-v2-AES
# To:
cpu: host
```

Alternatively, use the command:

```bash
qm set <vmid> --cpu host
```

### MongoDB Keyfile Setup

Generate a secure keyfile for internal authentication between MongoDB nodes:

```bash
openssl rand -base64 756 > roles/mongodb/tasks/mongo-keyfile
chmod 400 roles/mongodb/tasks/mongo-keyfile
```

### Change IP

Update IP configurations as required in network settings. (Include exact steps or location if applicable.)
