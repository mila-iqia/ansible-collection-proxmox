# mila.proxmox.vm

To manage virtual machines, use the role `mila.proxmox.vm`. This role makes use
of the Ansible modules `community.general.proxmox_kvm`.

The VMs are created from a list defined in `proxmox_vm`. Each item
in the list defines one VM.

Example for a VM:

```
  pve_kvm:
    - name: "clockwork1-dev"
      vmid: 1001
      node: vn-b001
      net:
        net0: virtio=42:49:54:11:01:00,bridge=vmbr0,tag=2
      cores: 4
      memory: 8192
      virtio:
        virtio0: vm-storage:64
```

To authenticate to the REST API of your Proxmox VE cluster, you first need to
create an [API token][pve_api_tokens], then you must define the 4 **mandatory**
variables below:

 - `proxmox_api_host`: the hostname or IP address of the Proxmox VE API server
 - `proxmox_api_user`: the user name to authenticate (e.g. `ansible@pam`)
 - `proxmox_api_token_id`: the token name (e.g. `prod`)
 - `proxmox_api_token_secret`: the token secret provided by ProxmoxVE

[pve_api_tokens]: https://pve.proxmox.com/pve-docs/chapter-pveum.html#pveum_tokens

By default, the proxmox modules in `community.general` do not validate SSL
certs. To enable certificate validation, define the variable:

 - `proxmox_api_validate_certs: true`

To enable update of VMs with new values:

 - `proxmox_kvm_update: true`

## High Availability (HA) Resources

The role supports configuring HA resources for virtual machines. To enable HA for a VM,
add an `ha` key to the VM definition. The `ha.state` parameter is mandatory.

Example with HA configuration:

```
pve_kvm:
  - name: "critical-vm"
    vmid: 1001
    node: pve-1
    net:
      net0: virtio=42:49:54:11:01:00,bridge=vmbr0,tag=2
    cores: 4
    memory: 8192
    virtio:
      virtio0: vm-storage:64
    ha:
      state: 'present'
      comment: 'Critical production VM'
      hastate: 'started'
      max_relocate: 3
      max_restart: 5
```

Supported HA parameters:
- `state`: (mandatory) The state of the HA resource (`present` or `absent`)
- `comment`: Optional comment for the HA resource
- `hastate`: The desired HA state
- `max_relocate`: Maximum number of relocations
- `max_restart`: Maximum number of restarts

If the `ha` key is not defined for a VM, any existing HA resource for that VM will be removed.
