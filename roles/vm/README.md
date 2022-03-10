# mila.proxmox.vm

To manage virtual machines, use the role `mila.proxmox.vm`. This role makes use
of the Ansible modules `community.general.proxmox_kvm`.

The VMs are created from a list defined in `proxmox_vm`. Each item
in the list defines one VM.

Example for a VM:

	proxmox_vm:
	  - name: "some_name"
		vmid: 1001
		node: vn-c003
		net:
		  net0: virtio=12:49:34:41:21:11,bridge=vmbr0,tag=99
		cores: 4
		memory: 8192
		virtio:
		  virtio0: vm-storage:64
		  virtio1: vm-storage:128

To authenticate to the REST API of your Proxmox VE cluster, you first need to
create an [API token][pve_api_tokens], then you must define the 4 **mandatory**
variables below:

 - `proxmox_api_host`: the hostname or IP address of the Proxmox VE API server
 - `proxmox_api_user`: the user name to authenticate (e.g. `ansible@pam`)
 - `proxmox_api_token_id`: the token name (e.g. `prod`)
 - `proxmox_api_token_secret`: the token secret provided by ProxmoxVE

[pve_api_tokens]: https://pve.proxmox.com/pve-docs/chapter-pveum.html#pveum_tokens
