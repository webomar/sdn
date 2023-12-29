# network-configuration.yml

- hosts: switches
  gather_facts: false
  collections:
    - arubanetworks.aoscx
  vars:
    ansible_python_interpreter: /usr/bin/python3
    vlan_ids:
      - 100
      - 200
      - 300
      - 400
      - 500
  tasks:
    - name: Configure New VLANs on Aruba Switches
      arubanetworks.aoscx.aoscx_vlan:
        vlan_id: "{{ item }}"
        vlan_name: "VLAN{{ item }}"
        state: present
      with_items: "{{ vlan_ids }}"
      delegate_to: localhost

    - name: Configure VLAN Interfaces on Cisco Router
      ios_config:
        lines:
          - "interface Vlan{{ item }}"
          - "ip address 192.168.{{ item }}.1 255.255.255.0"
        parents: "interface range GigabitEthernet0/0.{{ item }}"
      with_items: "{{ vlan_ids }}"
      when: "'router1' in inventory_hostname"
      delegate_to: localhost

    - name: Configure Access Ports on Aruba Switches
      arubanetworks.aoscx.aoscx_interface:
        name: "1/1/{{ item }}"
        vlan_id: "{{ item }}"
        mode: access
      with_items: "{{ vlan_ids }}"
      delegate_to: localhost
