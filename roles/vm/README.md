# mila.proxmox.vm

To manage virtual machines, use the role `mila.proxmox.vm`. This role makes use
of the Ansible modules `community.general.proxmox_kvm`.

The VMs are created from a list defined in `proxmox_vm`. Each item
in the list defines one VM.

Example for a VM:

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
