# Ansible Collection - mila.proxmox

The purpose of this collection is to manage containers and virtual machines in
the [Proxmox VE](https://www.proxmox.com/en/proxmox-ve) virtualization
management platform.

NOTE: The collection is still in its early stages of development, things might
change quickly and without notice while version is still 0.x.y.

## Requirements

The below requirements are needed on the host that executes this collection:

 - collection `community.proxmox`
 - jmespath
 - proxmoxer
 - requests

## Roles

* [container](roles/container/README.md)
* [ha](roles/ha/README.md)
* [patch](roles/patch/README.md)
* [vm](roles/vm/README.md)
