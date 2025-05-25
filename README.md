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

---

### Ansible Inventory Configuration

Hosts must be updated here for Ansible to connect to the correct MongoDB nodes.

* `ansible_host`: Target machine’s IP address.
* `ansible_user`: SSH login user (typically `root`).
* `ansible_ssh_private_key_file`: Path to the private SSH key for authentication.

---

### Variable Configuration and Security

Make sure to update the following variables with your environment-specific values:

* `mongo_replica_set_name`: Name of the replica set (e.g., `rs0`).
* `mongo_port`: MongoDB service port (default `27017`).
* `admin_user` / `admin_pass`: MongoDB admin credentials.
* `app_user` / `app_pass` / `app_db`: Application database credentials and name.

For security, encrypt sensitive variables using Ansible Vault:

```bash
ansible-vault encrypt <file_with_sensitive_vars>.yml
```
---

### MongoDB Configuration Template (`mongod.conf.j2`)

Defines MongoDB settings using Jinja2 variables for flexibility:

* `storage.dbPath`: Data directory path.
* `systemLog`: Log file location and append mode.
* `net.port`: MongoDB listening port (set via `mongo_port` variable).
* `net.bindIp`: Binds to all network interfaces (`0.0.0.0`).
* `replication.replSetName`: Replica set name (set via `mongo_replica_set_name` variable).
* `security.authorization`: Enables access control.
* `security.keyFile`: Path to internal authentication keyfile (created earlier i.e mongo-keyfile).

---

## Playbook Execution Order

Run the playbooks in the following sequence to ensure proper MongoDB setup:

1. `ansible-playbook playbooks/install_mongodb.yml` — Installs MongoDB on target hosts.
2. `ansible-playbook playbooks/configure_template.yml` — Applies configuration templates.
3. `ansible-playbook playbooks/create_user.yml` — Creates necessary MongoDB users.
4. `ansible-playbook playbooks/configure_replica_set.yml` — Sets up the MongoDB replica set.

---

### High Availability (HA) Setup
HA is achieved by configuring a MongoDB replica set with three members on different hosts. This setup ensures data redundancy and automatic failover in case one node becomes unavailable.

![image](https://github.com/user-attachments/assets/9316c2d7-b453-4b39-8f3f-4a28b6f5a104)
