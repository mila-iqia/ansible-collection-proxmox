# mila.proxmox.patch

The role `mila.proxmox.patch` allows to apply pre-defined patches on top of the
Proxmox installation. By default, no patch are applied, this is an opt-in
feature.

## List of patches

- Allow configuration of CT/VM with a token for root@pam
  - URL: https://bugzilla.proxmox.com/show_bug.cgi?id=3829
  - Enable with: `proxmox_patch_3829: True`
