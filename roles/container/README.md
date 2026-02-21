# mila.proxmox.container

To manage containers, use the role `mila.proxmox.container`. This role makes use
of the Ansible modules `community.general.proxmox_template` and
`community.general.proxmox`.

The containers are created from a list defined in `proxmox_container`. Each item
in the list defines one container.

Example for a container:

    proxmox_container:
      - hostname: 'container1'
        node: 'pve-1'
        ostemplate: 'local:vztmpl/debian-11-standard_11.0-1_amd64.tar.gz'
        disk: 'local:2'
        cores: 2
        nameserver: '10.10.0.1'
        netif: '{"net0":"name=eth0,gw=10.10.0.254,hwaddr=42:4d:69:6c:61:00,ip=dhcp,bridge=vmbr0"}'
        unprivileged: True
        features:
          - nesting=1  # For systemd, otherwise takes a lot of time to open ssh session

The role will parse `ostemplate` to ensure the required template is available on
all hosts.

Most of the parameters supported by the module `community.general.proxmox` can
be defined in the list. The following parameters are not supported yet, but
this might change in future development: `api_password`, `ip_address`, 
`password`, `pool`, `purge`, `searchdomain` and `storage`.

To authenticate to the REST API of your Proxmox VE cluster, you first need to
create an [API token][pve_api_tokens], then you must define the 4 **mandatory**
variables below:

 - `proxmox_api_host`: the hostname or IP address of the Proxmox VE API server
 - `proxmox_api_user`: the user name to authenticate (e.g. `ansible@pam`)
 - `proxmox_api_token_id`: the token name (e.g. `prod`)
 - `proxmox_api_token_secret`: the token secret provided by ProxmoxVE

[pve_api_tokens]: https://pve.proxmox.com/pve-docs/chapter-pveum.html#pveum_tokens

It is also possible to define the user, token id or token secret on a per
container basis for use cases where the management of a container requires
different access privileges.

    proxmox_container:
      - hostname: 'container2'
        node: 'pve-1'
        api_user: 'root@pam'
        api_token_secret: 'MySecretShouldBeInAVault ;)'
        features:
          - keyctl=1   # This feature needs root access in ProxmoxVE
        [...]

In the example above, `item.api_user` and `item.api_token_secret` will
respectively replace the values of `proxmox_api_user` and
`proxmox_api_token_secret`.

By default, the proxmox modules in `community.general` do not validate SSL
certs. To enable certificate validation, define the variable:

 - `proxmox_api_validate_certs: true`

The example below creates two containers that will serve as DHCP servers. The
configuration of the DHCP service is not managed by this role. The variable
`proxmox_container_dhcp` is defined as a play variable. It is then passed to
`proxmox_container` as a role parameter. It is advised to define it in your
Ansible inventory.

    - name: Instanciate DHCP servers in PVE
      hosts: localhost
      vars:
        proxmox_api_token_secret: "{{ lookup('env','PROXMOX_API_TOKEN') }}"
        proxmox_container_dhcp:
          - hostname: 'dhcp1'
            node: 'pve-1'
            ostemplate: 'local:vztmpl/debian-11-standard_11.0-1_amd64.tar.gz'
            disk: 'local:2'
            cores: 2
            nameserver: '10.10.0.1'
            netif: '{"net0":"name=eth0,gw=10.10.0.254,ip=10.10.0.2/24,bridge=vmbr0"}'
            unprivileged: True
            features:
              - nesting=1  # For systemd, otherwise takes a lot of time to open ssh session
          - hostname: 'dhcp2'
            node: 'pve-2'
            ostemplate: 'local:vztmpl/debian-11-standard_11.0-1_amd64.tar.gz'
            disk: 'local:2'
            cores: 2
            nameserver: '10.10.0.1'
            netif: '{"net0":"name=eth0,gw=10.10.0.254,ip=10.10.0.3/24,bridge=vmbr0"}'
            unprivileged: True
            features:
              - nesting=1  # For systemd, otherwise takes a lot of time to open ssh session
    roles:
      - name: Download proxmox appliance container template
        role: mila.proxmox.container
        vars:
          proxmox_container: "{{ proxmox_container_dhcp }}"


Once a container exists, some parameters cannot be changed with the
`community.general.proxmox` module. To circumvent this limitation, it is
possible to destroy the containers and recreate these. For this, include the
role `mila.proxmox.container` in the pre_tasks of your playbook, with
`tasks_from: destroy`.

    - name: Instanciate containers
      hosts: localhost
      pre_tasks:
        - name: Ensure existing containers are removed
          block:
            - name: Remove SSH public host keys
              ansible.builtin.known_hosts:
                name: "{{ item }}"
                state: absent
              loop:
                - 10.10.0.2
                - 10.10.0.3
            - name: Cleanup containers
              ansible.builtin.include_role:
                name: mila.proxmox.container
                tasks_from: destroy
              vars:
                proxmox_container: "{{ proxmox_container_list }}"
          when: proxmox_container_destroy_first is defined and proxmox_container_destroy_first
      roles:
        - name: Instanciate containers in Proxmox VE cluster
          role: mila.proxmox.container
          vars:
            proxmox_container: "{{ proxmox_container_list }}"

In the example above, ansible-playbook must run with `-e
proxmox_container_destroy_first=True` for the destruction of the containers to
be effective.

To enable update of LXC containers with new values:

 - `proxmox_container_update: true`

## High Availability (HA) Resources

The role supports configuring HA resources for containers. To enable HA for a container,
add an `ha` key to the container definition. The `ha.state` parameter is mandatory.

Example with HA configuration:

    proxmox_container:
      - hostname: 'container1'
        node: 'pve-1'
        ostemplate: 'local:vztmpl/debian-11-standard_11.0-1_amd64.tar.gz'
        disk: 'local:2'
        cores: 2
        nameserver: '10.10.0.1'
        netif: '{"net0":"name=eth0,gw=10.10.0.254,hwaddr=42:4d:69:6c:61:00,ip=dhcp,bridge=vmbr0"}'
        unprivileged: True
        ha:
          state: 'present'
          comment: 'Critical service container'
          hastate: 'started'
          max_relocate: 3
          max_restart: 5

Supported HA parameters:
- `state`: (mandatory) The state of the HA resource (`present` or `absent`)
- `comment`: Optional comment for the HA resource
- `hastate`: The desired HA state
- `max_relocate`: Maximum number of relocations
- `max_restart`: Maximum number of restarts

If the `ha` key is not defined for a container, any existing HA resource for that
container will be removed.
