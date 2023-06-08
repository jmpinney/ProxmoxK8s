# ProxmoxK8s
Pipeline for deploying a kubernetes cluster on an existing proxmox cluster

You'll need an existing proxmox node and VM's to run jenkins, terraform, and ansible.
Jenkins will kick off terraform which will deploy an arbitrary number of VMs based on a cloudinit image. Ansible will then configure the VM's with kubernetes. And when you're done, you can easily destroy the cluster via jenkins as well. 
