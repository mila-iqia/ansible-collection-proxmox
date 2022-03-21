# mila.proxmox.ha

To manage HA Groups assignations of VMs or CTs , use the role `mila.proxmox.ha`.

You must provide the `proxmox_ha_resources` variable in order to assign
ressources to existing HA Groups.

Example:

	---
	proxmox_ha_resources:
	  - group: "HA_all"
		sid: "vm:110"
	  - group: "HA_vn-c"
		sid: "vm:108"


This role DOES NOT creates the HA Groups ! You must create them yourself.

To authenticate to the REST API of your Proxmox VE cluster, you first need to
create an [API token][pve_api_tokens], then you must define the 4 **mandatory**
variables below:

 - `proxmox_api_host`: the hostname or IP address of the Proxmox VE API server
 - `proxmox_api_user`: the user name to authenticate (e.g. `ansible@pam`)
 - `proxmox_api_token_id`: the token name (e.g. `prod`)
 - `proxmox_api_token_secret`: the token secret provided by ProxmoxVE

[pve_api_tokens]: https://pve.proxmox.com/pve-docs/chapter-pveum.html#pveum_tokens
