######################### 
Taxonomy of a playbook
#########################

These module examples explain how to create a simple Ansible playbook, run the Dell EMC Ansible modules, then configure a switch using Ansible roles.

Create simple Ansible playbook
******************************

**Step 1** 

Create an inventory file called ``inventory.yaml``, then specify the IP address.

::

    spine1 ansible_host=10.11.182.16

**Step 2**

Create a host variable file called ``host_vars/spine1.yaml``, then define the host, credentials, and transport.

::

    hostname: spine1
    ansible_ssh_user: xxxxx
    ansible_ssh_pass: xxxxx
    ansible_become_method: enable
    ansible_become: yes
    ansible_become_pass: xxxxx
    ansible_network_os: xxxxx


**Step 3**

Create a playbook called ``showver.yaml``.

::

  hosts: spine1
  connection: network_cli
  gather_facts: no

  tasks:
  - name: "Get Dell EMC OS9 Show version"
    dellos9_command:
      commands: ['show version']
    register: show_ver

  - debug: var=show_ver

**Step 4**

Run the playbook.

::

    ansible-playbook -i inventory.yaml showver.yaml

Create simple Ansible playbook using connection="netconf"
*********************************************************

**Step 1**

Create an inventory file called ``inventory.yaml``, then specify the IP address.

::

    spine1 ansible_host=10.11.182.16

**Step 2**

Create a host variable file called ``host_vars/spine1.yaml``, then define the host, credentials, and transport.

::

    hostname: spine1
    ansible_ssh_user: xxxxx
    ansible_ssh_pass: xxxxx
    ansible_network_os: dellos10

**Step 3**

Create a playbook called ``create_vlan.yaml``.

::

  hosts: spine1
  connection: netconf
  gather_facts: no

  tasks:
  - name: "Create a vlan entry"
    netconf_config:
    host: 10.16.138.15
    username: admin
    password: admin
    hostkey_verify: false
    xml: |
        <config>
           <interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces" xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type" xmlns:dell-if="http://www.dellemc.com/networking/os10/dell-interface" xmlns:dell-eth="http://www.dellemc.com/networking/os10/dell-ethernet" xmlns:dell-lag="http://www.dellemc.com/networking/os10/dell-lag" xmlns:dell-lacp="http://www.dellemc.com/networking/os10/dell-lacp">
             <interface>
               <type>ianaift:l2vlan</type>
               <name>vlan106</name>
             </interface>
           </interfaces>
         </config>

**Step 4**

Run Dell EMC Ansible examples
****************************************

Use these sample Ansible playbooks to understand how to use Dell EMC Ansible modules.

Installation and setup
======================

1. Install `Ansible <http://docs.ansible.com/ansible/intro_installation.html>`_.

2. Clone the `Ansible-dellos-examples <https://github.com/Dell-Networking/ansible-dellos-examples>`_ repository in the control machine.

3. Update the ``inventory.yaml`` file to configure the device IP.

4. Update the corresponding host variables; use ``hosts_var/dellos10_sw1.yaml`` for device credentials.

OS6
---

dellos6_facts module that collects the facts from the OS6 device example.

::

    ansible-playbook -i inventory.yaml getfacts_os6.yaml

dellos6_command module that executes the show version command example.

::

    ansible-playbook -i inventory.yaml showver_os6.yaml

dellos6_config module that configures the hostname on the OS6 device example.

:: 

    ansible-playbook -vvv -i inventory.yaml hostname_os6.yaml

OS9
---

dellos9_facts module that collects the facts from the OS9 device example.

::

    ansible-playbook -i inventory.yaml getfacts_os9.yaml

dellos9_command module that executes the show version command example.

::

    ansible-playbook -i inventory.yaml showver_os9.yaml

dellos9_config module that configures the hostname on the OS9 device example.

::

    ansible-playbook -vvv -i inventory.yaml hostname_os9.yaml

OS10
----

dellos10_facts module that collects the facts from the OS10 device example.

::

    ansible-playbook -i inventory.yaml getfacts_os10.yaml

dellos10_command module that executes the show version command example.

::

    ansible-playbook -i inventory.yaml showver_os10.yaml

dellos10_config module that configures the hostname on the OS10 device example.

::

    ansible-playbook -vvv -i inventory.yaml hostname_os10.yaml

Run Ansible example
***********************************

Use this example to configure VLAN using CPS operations.

**Step 1**

Create an inventory file called ``inventory.yaml``, then specify the IP address.

::

    spine1 ansible_host=10.11.182.16

**Step 2**

Create a host variable file called ``host_vars/spine1.yaml``, then define the host, credentials, and transport.

::

    hostname: spine1
    ansible_ssh_user: xxxxx
    ansible_ssh_pass: xxxxx

**Step 3**

Create a file called ``create_vlan.yaml`, then define the CPS operations.

::
  
  - hosts: opx_cps
    tasks:
      - name: Create vlan
        opx_cps:
          module_name: "dell-base-if-cmn/if/interfaces/interface" 
          attr_data: "{{ attr_vlan }}"
          operation: "create" 
        environment:
          PYTHONPATH: "/usr/lib/opx:/usr/lib/x86_64-linux-gnu/opx"
          LD_LIBRARY_PATH: "/usr/lib/opx:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib:/lib"

**Step 4**

Run the playbook.

::

    ansible-playbook -i inventory.yaml create_vlan.yaml

Playbook using Ansible roles example
************************************

Use these examples to configure the switch using Ansible roles.

**Step 1**

Create an inventory file called ``inventory.yaml``, then specify the device IP address.

::

    spine1 ansible_host= <ip_address> 

**Step 2**

Create a host variable file called ``host_vars/spine1.yaml``, then define the host, credentials, and transport.

::

	---
	hostname: dellos9

        ansible_ssh_user: xxxxx
        ansible_ssh_pass: xxxxx
        ansible_become: yes
        ansible_become_method: enable
        ansible_become_pass: xxxxx
        ansible_network_os: dellos9

	
	dellos_interface:
		fortyGigE 0/32:
		  desc: "Connected to Spine1"
		  portmode:
		  switchport: False
		  mtu: 2500
		  admin: up
		  ipv6_and_mask: 2001:4898:5808:ffa2::5/126
		  suppress_ra : present
		  ip_type_dynamic: true
		  ip_and_mask: 192.168.23.22/24
		  class_vendor_identifier: present
		  option82: true
		  remote_id: hostname
		fortyGigE 0/20:
		  portmode:
		  switchport: False
		fortyGigE 0/64:
		  portmode:
		  switchport: True
		fortyGigE 0/60:
		  portmode:
		  switchport: True
		fortyGigE 0/12:
		  portmode:
		  switchport: True
		loopback 0:
		  ip_and_mask: 1.1.1.1/32
		  admin: up
		Port-channel 12:
		  switchport: True
	dellos_vlan:
		vlan 100:
		  name: "Mgmt Network"
		  description: "Int-vlan"
		  tagged_members:
			- port: fortyGigE 0/60
			  state: present
		  untagged_members:
			- port: fortyGigE 0/12
			  state: present
		  state: present

**Step 3**

Create a playbook called ``switch_config.yaml``.

::

	---
	- hosts: dellos9
	  gather_facts: no
	  connection: network_cli
	  roles:		
		- Dell-Networking.dellos-interface
		- Dell-Networking.dellos-vlan

**Step 4**

Run the playbook.

::

    ansible-playbook -i inventory.yaml switch_config.yaml
