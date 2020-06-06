# Virtual Machines

# Quick way to provision a new VM

```bash
# Provision a new VM and run serial console to proceed over the installation steps
virt-install \
  --name vm1 \
  --memory 4096 \
  --memballoon model=virtio \
  --disk size=10 \
  --vcpus 4 \
  --os-type linux \
  --os-variant ubuntu18.04 \
  --graphics none \
  --location 'http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/' \
  --extra-args "console=tty0 console=ttyS0,115200n8"

# To exit the virsh console send "^]" -> CTRL + ]

# To entry the console again -> but this has to be supported by the OS in the VM
virsh console vm1

# Do not forget to select OpenSSH package in the installation
# and set meaningful username and password

# Later you can add GPU for virt-viewer/virt-manager access

# Get IP address
virsh domifaddr vm1

# Inspect network configuration
virsh net-list
virsh net-info default
virsh net-dhcp-leases default
```
