# LTRDCN-2765 - VXLAN EVPN Fabric and NetDevOpsautomation using Ansible  


---
- jinja2_fabric.yml: Deploy VXLAN EVPN Fabric using jinja2 template. Template with spine and leaf running OSPF for underlay RP and iBGP for EVPN control plane.
- nxos_fabric.yml: Deply VXLAN EVPN Fabric using NXOS modules.
- get_config.yml: Backup running configuration and compare with the standard config
- code_upgrade.yml: Compare running version with standard code, and upgrade if it's not compliant.
- verify_fabric.yml: day2 task for checking underlay and overlay
- vni_provision.yml: day2 task to deply new vnis
  
