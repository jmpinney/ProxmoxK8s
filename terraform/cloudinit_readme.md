# Making a debian VM Template for Proxmox and CloudInit
#Adapted from https://yetiops.net/posts/proxmox-terraform-cloudinit-saltstack-prometheus/
#and https://github.com/UntouchedWagons/Ubuntu-CloudInit-Docs

## Creating the vendor.yaml file for cloudinit. Installs the qemu agent and restarts.

    cat << EOF | sudo tee /var/lib/vz/snippets/vendor.yaml
    #cloud-config
    runcmd:
        - apt update
        - apt install -y qemu-guest-agent
        - systemctl start qemu-guest-agent
        - reboot
    EOF

# Navigate to the ISO directory for Proxmox
$ cd /var/lib/vz/templates/isos

# Source the image
$ wget http://cdimage.debian.org/cdimage/openstack/current-10/debian-10-openstack-amd64.qcow2

# Create the instance
qm create 9000 -name debian-cloudinit -memory 1024 -net0 virtio,bridge=vmbr0 -cores 1 -sockets 1

# Import the OpenStack disk image to Proxmox storage
qm importdisk 9000 debian-10-openstack-amd64.qcow2 local-lvm

# Attach the disk to the virtual machine
qm set 9000 -scsihw virtio-scsi-pci -virtio0 local-lvm:vm-9000-disk-0

# Add a serial output
qm set 9000 -serial0 socket

# Set the bootdisk to the imported Openstack disk
qm set 9000 -boot c -bootdisk virtio0

# Enable the Qemu agent
qm set 9000 -agent 1

# Allow hotplugging of network, USB and disks
qm set 9000 -hotplug disk,network,usb

# Add a single vCPU (for now)
qm set 9000 -vcpus 1

# Add a video output
qm set 9000 -vga qxl

# Set a second hard drive, using the inbuilt cloudinit drive
qm set 9000 -ide2 local-lvm:cloudinit

# Resize the primary boot disk (otherwise it will be around 2G by default)
# This step adds another 8G of disk space, but change this as you need to
qm resize 9000 virtio0 +8G

## Configuring CloudInit

    sudo qm set 9000 --cicustom "vendor=local:snippets/vendor.yaml"
    sudo qm set 9000 --ciuser k8suser
    sudo qm set 9000 --cipassword password

# Convert the VM to the template
qm template 9000
