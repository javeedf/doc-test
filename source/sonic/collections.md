Ansible Network Collection for Enterprise SONiC Distribution by Dell Technologies from .dm file
===============================================================================================
The SONiC Ansible network collection includes resource modules, plugins, and sample playbooks that can be used to work on Dell EMC PowerSwitch platforms running Enterprise SONiC Distribution by Dell Technologies. It also includes information that shows how the collection can be used. This version is tested on Enterprise SONiC Distribution by Dell Technologies, release 3.0.2.

Version compatibility
-----------------------------
Ansible version 2.10 or later.

Supported connections
---------------------
The SONiC Ansible network collection supports network_cli and httpapi connections.

## Included content
**CLICONF plugins**

Name | Description
--- | ---
[network_cli](https://github.com/ansible-collections/dellemc.sonic)|Use Ansible cliconf to run commands on Enterprise SONiC Distribution by Dell Technologies.

**HTTPAPI plugins**

Name | Description
--- | ---
[httpapi](https://github.com/ansible-collections/dellemc.sonic)|Perform REST operations on remote devices running Enterprise SONiC Distribution by Dell Technologies.

Modules
-------
Name | Description
--- | ---
[**dellemc.sonic.sonic_interfaces**](https://github.com/ansible-collections/dellemc.sonic/blob/master/plugins/modules/sonic_interfaces.py)|Enterprise SONiC interfaces resource module
[**dellemc.sonic.sonic_l2_interfaces**](https://github.com/ansible-collections/dellemc.sonic/tree/master/plugins/modules/sonic_l2_interfaces.py)|Enterprise SONiC l2 interfaces resource module
[**dellemc.sonic.sonic_l3_interfaces**](https://github.com/ansible-collections/dellemc.sonic/tree/master/plugins/modules/sonic_l3_interfaces.py)|Enterprise SONiC l3 interfaces resource module
[**dellemc.sonic.sonic_lag_interfaces**](https://github.com/ansible-collections/dellemc.sonic/tree/master/plugins/modules/sonic_lag_interfaces.py)|Enterprise  SONiC lag interfaces resource module
[**dellemc.sonic.sonic_vlans**](https://github.com/ansible-collections/dellemc.sonic/tree/master/plugins/modules/sonic_vlans.py)|Enterprise SONiC VLANs resource module
[**sonic_command**](https://github.com/ansible-collections/dellemc.sonic/blob/master/plugins/modules/sonic_command.py)|Run commands from Management Framework CLI on remote devices running Enterprise SONiC
[**sonic_config**](https://github.com/ansible-collections/dellemc.sonic/blob/master/plugins/modules/sonic_config.py)|Manage configuration of the Management Framework CLI on remote devices running Enterprise SONiC
[**sonic_api**](https://github.com/ansible-collections/dellemc.sonic/blob/master/plugins/modules/sonic_api.py)|Perform REST operations on remote devices running Enterprise SONiC

Playbooks
---------

The playbooks directory includes sample playbooks that show how to use the SONiC collections for provisioning devices running Enterprise SONiC Distribution by Dell Technologies.

Installation
----------------

Install the latest version of the SONiC collection from Ansible Galaxy.

      ansible-galaxy collection install dellemc.sonic

To install a specific version, specify a version range identifier. For example, to install the most recent version that is greater than or equal to 1.0.0 and less than 2.0.0.

      ansible-galaxy collection install 'dellemc.sonic:>=1.0.0,<2.0.0'

**Sample playbook to configure VLAN entry and show VLAN status using cliconf connection**

    ---

    - name: "Test Management Framework CLI show commands on SONiC"
      hosts: sonic
      gather_facts: no
      connection: network_cli
      collections:
        - dellemc.sonic
      tasks:
        - name: Add VLAN entry
          sonic_config:
            commands: ['interface Vlan 700','exit']
            save: yes
          register: config_op
        - name: Test SONiC single command
          sonic_command:
            commands: 'show vlan'
          register: cmd_op

**Sample playbook to configure VLAN using HTTPAPI connection**

    ---

    - name: "To configure VLAN using SONiC REST API"
      hosts: sonic
      gather_facts: no
      connection: httpapi
      collections:
        - dellemc.sonic
      tasks:
        - name: Perform "PUT" operation to add VLAN network instance
          sonic_api:
            url: data/openconfig-network-instance:network-instances/network-instance=Vlan100
            method: "PUT"
            body: {"openconfig-network-instance:network-instance": [{"name": "Vlan100","config": {"name": "Vlan100"}}]}
            status_code: 204
        - name: Perform "GET" operation to view VLAN network instance
          sonic_api:
            url: data/openconfig-network-instance:network-instances/network-instance=Vlan100
            method: "GET"
            status_code: 200
          register: api_op

**Sample playbook to configure VLANs, l2 interfaces, and l3 interfaces using SONiC resource module**

    ---

    - name: "Configuration of VLANs, l2 interfaces and l3 interfaces on SONiC"
      hosts: sonic
      gather_facts: False
      connection: httpapi
      collections:
        - dellemc.sonic
      tasks:
       - name: "Configuring VLANs"
         sonic_vlans:
            config:
             - vlan_id: 701
             - vlan_id: 702
             - vlan_id: 703
             - vlan_id: 704
            state: merged
         register: sonic_vlans_creation_output
       - name: "Configuring l2 interfaces"
         sonic_l2_interfaces:
            config:
            - name: Ethernet28
              access:
                vlan: 701
              trunk:
                allowed_vlans:
                  - vlan: 702
                  - vlan: 703
            state: merged
         register: sonic_l2_interfaces_configurarion_output
       - name: "Configuring l3 interfaces"
         sonic_l3_interfaces:
           config:
            - name: Ethernet20
              ipv4:
                - address: 8.1.1.1/16
              ipv6:
                - address: 3333::1/16
           state: merged
         register: sonic_l3_interfaces_configuration_output

**Sample host_vars/sonic_sw1.yaml**

    hostname: sonic_sw1
    # parameters for connection type httpapi
    ansible_ssh_user: xxxx
    ansible_ssh_pass: xxxx
    ansible_network_os: dellemc.sonic.sonic
    ansible_httpapi_use_ssl=true
    ansible_httpapi_validate_certs=false
    # parameters for connection type network_cli
    ansible_ssh_user: xxxx
    ansible_ssh_pass: xxxx
    ansible_network_os: dellemc.sonic.sonic

**Sample inventory.yaml**

    [sonic]
    sonic_sw1 ansible_host=100.104.28.119 ansible_network_os=dellemc.sonic.sonic

(c) 2020 Dell Inc. or its subsidiaries. All rights reserved.
