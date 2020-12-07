# Overlay Automation

The data model and set of roles included in this repo are designed to automate deployment of generic EVPN overlay configuration in a brownfield deployment. For greenfield deployments, arista.avd collection is recommended instead.

The intended result is for ansible to manage a configlet on CVP for each fabric device containing this configuration. This configlet can coexist with any other overlay or underlay configuration, as long as vni assignments, vlan numbers, etc doesn't overlap. It is meant to function as an easy, entry-level way of automating part (or all) of your overlay configuration with ansible.

There is also a mechanism included to deploy the configuration directly to the switches via API. However, this is less desireable since only additive changes can be deployed if you're not managing your entire device config with ansible.

# Roles

## eos_generate

Used to generate the cli configuration for the configlets. Mainly based on the data model included in the group_vars/leaf/overlay.yml file, host vars files and the group_vars/leaf1/leaf1.yml and ..../leaf2/leaf2.yml files. These files are all included as an example on how to build your own vars.

An example inventory file is also included.

The main template for this role is the templates/overlay.j2 template, which serves as a master index, and includes subtemplates located in templates/overlay/ directory.

The role loops through all hosts in the "leaf" host group, renders the overlay.j2 template for each device and puts the result in files/{{ inventory_hostname }}_overlay.

Playbook example: 

    ---
    - name: Generate arista overlay configs
    hosts: leaf
    gather_facts: false
    
    roles:
    - eos_generate

## cvp_deploy

This role is designed to take the raw config generated with eos_generate and put it into configlets, push the configlets to a CVP instance, attach the configlets to the proper devices and execute the resulting tasks, pushing the configuration to the fabric devices.

