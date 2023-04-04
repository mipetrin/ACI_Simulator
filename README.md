# Ansible Playbook to auto-provision an ACI Simulator that is deployed within your vCenter environment

Once an ACI Simulator is deployed, upon bootup, there is a start up script that needs to be completed every time the VM is started/rebooted. For NetDevOps purposes, I thought it would be better if I could automate this bring-up process, which is what is achieved via this Ansible playbook.

# Latest as of April 2023:
   - Made changes to support newer APIC startup script introduced in ACI 5.x and newer
   - Tested and validated that this works against both ACI 5.2 and ACI 6.0 simulator versions
   - Added demo video show casing the Ansible playbook in action.
      - NOTE: The video lasts for approx. 3 minutes whereas real-time the process takes approx. 10-15 minutes.

# Assumptions
   - You already have downloaded the Cisco ACI Simulator OVA from Cisco.com
   - You have already deployed the OVA to your vCenter environment
   - Update the Simulator_Inventory to match your environment and VM name for your crafted ACI Simulator from the Simulator

# Known Behaviour:
   - If using older simulator, there are only 3 nodes (101, 201, 102). Hence 202 (non existent) will fail during "wait for node to become ready"
   - I only change the details of IPv4, Default Gateway, Passwords - as the things that only need to be configured

# Requirements:
   - Ansible 2.9 or newer
   - pip modules; pyVmomi, jmespath
   - vSphere > 6.5 (to utilize vmware_guest_sendkey)
   - Assumes ACI Simulator already deployed via OVA and network connectivity correctly setup. Basic checks performed
      - port-group, promiscuous ports, mac forge, etc
      - Newer ACI Simulator versions, >5.1(3)
         - ACI 5.1 (option for topology setup = large/small Fabric (2spine/2leaf or 1spine/1leaf)
         - Make sure "newer_apic_sim" is set to yes in Inventory. Otherwise change to no for older Simulators
         - If doing large, need 16+ vCPUs and 64GB vRAM - Playbook checks and modifies VM accordingly

# Execution

Entire playbook
```YAML
ansible-playbook -i Simulator_Inventory ACI_Simulator_Setup.yml
```

Debug and test the VM portion
```YAML
ansible-playbook -i Simulator_Inventory ACI_Simulator_Setup.yml --tags vm_debug
```

If ACI Sim VM already on and waiting at the Startup Script
```YAML
ansible-playbook -i Simulator_Inventory ACI_Simulator_Setup.yml --tags startup_script
```

If ACI Sim VM already on and configured, you just want to perform Fabric Membership registration
```YAML
ansible-playbook -i Simulator_Inventory ACI_Simulator_Setup.yml --tags fabric_registration
```

If you want to skip the VMware Port-Group check (eg: forged transmits, MAC changes, and promiscuous)
```YAML
ansible-playbook -i Inventory.txt ACI_Simulator_Setup.yml --skip-tags "vmware_pg_check"
```

# Inside a virtual environment
pip install ansible
pip install pyVmomi
pip install "jmespath"

ansible-galaxy collection install cisco.aci
ansible-galaxy collection install community.vmware

May require to install Certs if fails on above:
/Applications/Python\ 3.7/Install\ Certificates.command

Possibly also require to update/upgrade/install xcode
xcode-select --install


Created by Michael Petrinovic 2021


WARNING:

These scripts are meant for educational/proof of concept purposes only - as demonstrated at Cisco Live and/or my other presentations. Any use of these scripts and tools is at your own risk. There is no guarantee that they have been through thorough testing in a comparable environment and I am not responsible for any damage or data loss incurred as a result of their use
