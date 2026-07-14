# Commands for the EXAM

```bash
# Run playbook in dry run mode
ansible-playbook <playbook> --check
```

```bash
# Run playbook in dry run mode with ansible-navigator
ansible-navigator run <playbook> --check -m stdout
```

```bash
# Execute the playbooks
ansible-navigator run <playbook> -m stdout
```

```bash
# Change vault password
ansible-vault rekey secret.yaml
```

```bash
# Check if a configuration is correct
ansible all -a "<shell command>"
```

```bash
# Check if the inventory is correct
ansible all --list-hosts
```

```bash
# Test the repository configuration
ansible all -a "yum repolist all"
```

```bash
# Install collections
ansible-galaxy collection install <collection> -p <directory_to_install_the_collection>
```

```bash
# Install a role from file
ansible-galaxy install -r <file.yaml> -p <directory_to_install_the_role>
```

```bash
# Create a new role
ansible-galaxy init <role_name>
```

```bash
# List roles available
ansible-galaxy list
```

```bash
# Decrypt or encrypt a file with vault using a file as source of secret
ansible-vault decrypt vault.yml --vault-pass-file secret.txt

ansible-vault encrypt vault.yml --vault-pass-file secret.txt
```

# LVM Exercise Nodes preparation
```bash
# For LVM excercise
# Attach a 2Gb size disk on serverd
# Attach a 1Gb size disk on servere
# Create a partition on serverd
ssh serverd -- sudo parted /dev/vdb --script mklabel gpt mkpart primary 0% 100% set 1 lvm on

ssh serverd -- sudo partprobe /dev/vdb

ssh serverd -- sudo pvcreate /dev/vdb1

ssh serverd -- sudo vgcreate research /dev/vdb1

ssh serverd -- sudo vgs
# Create a partition on servere
ssh servere -- sudo parted /dev/vdb --script mklabel gpt mkpart primary 0% 100% set 1 lvm on

ssh servere -- sudo partprobe /dev/vdb

ssh servere -- sudo pvcreate /dev/vdb1

ssh servere -- sudo vgcreate research /dev/vdb1

ssh servere -- sudo vgs
```

```bash
# Check selinux config
ansible all -a "cat /etc/selinux/config"
```

```bash
# Check NTP configuration
ansible all -a "chronyc sources -v"
# or
ansible all -a "timedatectl"
```
