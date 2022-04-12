#Appendix B: Software compliance check and remediation

In this section, we will run software version compliance check using Ansible. For fabric switch that is not running on standard software version, we will perform software upgrade and bring all fabric switches into the standard version.  

In this playbook, we will use “nxos_facts” to find the software version on each fabric switch. Then we will compare with standard software version, 7.0(3)I7(4) in this lab. For fabric switch that is not running on standard version, the playbook will upgrade and reboot the switch.

The playbook will use **“nxos_file_copy”** module to copy image from remote repository to bootflash.

* On **Atom,** open up the project folder `EVPN-Ansible` and create new file under this folder (“EVPN-Ansible”). Name the new file **“code_upgrade.yml”.** and add below content (you may copy and paste):

```yaml
---
#Appendix code upgrade
  - hosts: spine,leaf
    vars:
      - standard: 7.0(3)I7(4)
      - image_file: nxos.7.0.3.I7.4.bin
    tasks:
      - name: "software complaince check"
        cisco.nxos.nxos_facts:
          gather_subset: all
      - name: "change to standard code"
        block:
          - debug: msg="{{ansible_net_hostname}} is not running standard {{standard}}"
          - cisco.nxos.nxos_feature:
              feature: scp-server
              state: enabled
          - name: "upload image file"
            cisco.nxos.nxos_file_copy:
              file_pull: True
              file_pull_timeout: 1200
              remote_file: "/root/downloads/{{image_file}}"
              remote_scp_server: "198.18.4.150"
              remote_scp_server_user: "root"
              remote_scp_server_password: "C1sco12345"
          - name: "change boot statement"
            cisco.nxos.nxos_config:
              lines: boot nxos bootflash:{{image_file}}
              save_when: modified
          - name: "reload switch"
            cisco.nxos.nxos_command:
              commands:
                - command: reload
                  prompt: '(y/n)?'
                  answer: 'y'
        when: ansible_net_version != standard
        rescue:
          - debug:
              msg: "{{ansible_net_hostname}} is reloading"
          - name: Wait For Device To Come Back Up
            wait_for:
              port: 22
              state: started
              timeout: 900
              delay: 60
              host: "{{ inventory_hostname }}"
        always:
          - debug:
              msg: "All devices are running {{standard}}"
```

* On the Ansible server, run the playbook for software compliance check and code upgrade by issuing the `ansible-playbook code_upgrade.yml` command as shown below:
~~~
[root@rhel7-tools EVPN-Ansible]# ansible-playbook code_upgrade.yml
~~~


------

<font style= "background-color:  yellow">**Note:** It is expected to see timeout error message when playbook reloads the switch.</font>

------

Below screenshot shows the output of above command:

![](pic/b2_1_0.png)

***
<font style= "background-color:  yellow">Switch will take 20 mins to bootup; up to this point, you have completed all tasks.* </font>

***



#Congratulations! You have completed the whole lab including the Optional (Appendix) sections. Well done!  
