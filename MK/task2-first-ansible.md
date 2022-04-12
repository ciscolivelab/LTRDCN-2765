In this section, we will create the first Ansible Playbook. In the playbook, we will configure VLANs on leaf switches, and assign VLANs to the server facing port.

You will learn create variables inside playbook, learn simple loop using “**with_items**”, learn simple logical using “**when**” and learn “**tags**” to isolate tasks from whole playbook.


---
### Step 1: Using "Atom" text Editor


* Open “Atom” text editor by double click the icon on desktop. Atom is the recommended text editor for this lab:

	![](pic/2-1-atom.png)

* After opening ATOM, **Click** `No, Never` to the ***“Register as default atom:// URI handler”*** message as show below:

	![](pic/2-1-2-atom.png)

* As an optional step: on ATOM you may close the Welcome, Welcome Guide, Telemetry Consent tabs.


---
### Step 2: Atom folder

* After opening ATOM, there should be a folder in the left pane named “EVPN-Ansible.”
* **Right click** the pre-configured project folder `EVPN-Ansible` and select `New File`

	![](pic/2-2-atom-folder.png)



*  Name the new file `vlan_provision.yml` and hit enter. This will create the new file:

	![](pic/2-2-2-atom-vlan.png)


*  Also, on the lower bar of the ATOM, verify that  file grammar of **YAML** is selected instead of default "**Plain Text**".  If **YAML** is not selected, then you should **choose** it from the listed options.

    ![](pic/2-2-3-atom-yaml.png)


---
### Step 3: Define variables, tasks for playbook

In this step, we are going to define the scope, variable for playbook and tasks

* In the opened window for ‘vlan_provision.yml’ file, enter the below content.
	*  **NOTE: YAML is space sensitive. Hence be careful with the spaces in below section.  You may copy and paste the full content (start to finish) from below to make sure that spaces are copied properly.**

~~~ YAML
---                     
#Task 2: Simple playbook assign VLAN to server facing port
  - hosts: leaf:jinja2_leaf

~~~

Further note:

*  “**hosts:**” defines the scope of this playbook applies to all switches in group ‘leaf’ and ‘jinja2_leaf’ (within the ***"hosts"*** file created in pervious task).
	*  **Note** that you can review the IP addresses of the three (2) “leaf” and one (1) “jinja2_leaf” in the “***hosts***” file (configured in previous steps).  The IP addresses are:
		*  jinja2_leaf: 198.18.4.104
		*  leaf: 198.18.4.101
		*  leaf: 198.18.4.103


---
### Step 4: VLAN tasks in playbook

* Continue to add below tasks on the same  playbook file:
    - **NOTE: YAML is space sensitive. Hence be careful with the spaces in below section.  You may copy and paste the full content (start to finish) from below to make sure that spaces are copied properly.**


~~~ YAML
	tasks:
	  - name: provision VLAN
	    cisco.nxos.nxos_vlans:
	      config:
	        - vlan_id: "{{item}}"
	          state: active
	    with_items:
	     -  140
	     -  141
	    tags: add vlans
~~~


Note:

* Multiple plays can be defined in one playbook under “**tasks**”, each starts with “-“ .
* This kind of play creates multiple VLANs using **cisco.nxos.nxos_vlans** module.

* At the end of this play, we use “**tags**” to name the play “**add vlans**”. This is useful to run a specific part of the configuration without running whole playbook.

* Below screenshot shows how playbook will look:

	![](pic/2-4-1.png)

	**NOTE: Formatting is extremely important when working with Ansible. Ansible playbook would return errors if the spaces are not properly aligned or formatting is not correct
**


* **Click** `File` and `Save` . This will save the playbook, and also ftp the playbook to Ansible server using pre-configured “**remote-sync**” package.

	![](pic/2-4-2.png)

	**NOTE: Once the Save button is pressed, then at the lower part of ATOM app, you will see message about connecting to Ansibe host (198.18.134.150) and saving the vlan_provision.yml file**



---
### Step 5: Execute playbook

After creating the playbook, it is now time to execute the playbook.
*  Before executing the playbook, we will verify the leaf switch if it has any vlan configuration present on it.
*  **Login** to leaf-3 switch using the Mputty client, or any leaf switch in previous playbook (remember **hosts:** variable in the file), and execute `show vlan brief` command.
	*  This command will show the vlans that currently exist on the leaf switch.  As you note from below screenshot, only the default VLAN (vlan number 1) is configured:

![](pic/2-5-1.png)

* Now, go to mputty, login or launch a new ssh into Ansible node (198.18.134.150)
* Use command `ansible-playbook vlan_provision.yml --tags "add vlans"` under folder "EVPN-Ansible" as shown below:

		[root@rhel7-tools EVPN-Ansible]# ansible-playbook vlan_provision.yml --tags "add vlans"


	![](pic/2-5-2.png)

	**Note: If the playbook fails first time, re-run the playbook again. Make sure to save all the changes in the playbook first before executing the playbook in Ansible.**

* After playbook is run successfully, login into leaf 3 again and check if vlan 140 and vlan 141 appears. There would also be a log message on the screen indicating a configuration change was pushed to the device:

 	![](pic/2-5-3.png)



---
### Step 6: Server port VLAN tasks in playbook

We have just tested our first playbook with basic configuration (i.e, by adding 2 VLANS), now we are going to add more tasks in our existing playbook “**vlan_provision.yml**” in this step:

* we will add new plays in the playbook to assign VLANs to server facing port. This time, we will configure VLAN towards the server facing ports
* Go back to **ATOM** and **add** the following plays to the **existing playbook**
    - *NOTE: YAML is space sensitive. Hence be careful with the spaces in below section.  You may copy and paste the full content (start to finish) from below to make sure that spaces are copied properly.*


~~~YAML
	  - name: configure server facing port to L2
	    cisco.nxos.nxos_interfaces:
	      config:
	        - name: eth1/3
	          mode: layer2
	  - name: configure VLAN for server port
	    when: ("101" in inventory_hostname) or ("103" in inventory_hostname)
	    cisco.nxos.nxos_l2_interfaces:
	      config:
	        - name: eth1/3
	          access:
	            vlan: 140
	      state: overridden
	  - name: configure VLAN for server port
	    when: ("102" in inventory_hostname) or ("104" in inventory_hostname)
	    cisco.nxos.nxos_l2_interfaces:
	      config:
	        - name: eth1/3
	          access:
	            vlan: 141
	      state: overridden
~~~

* **Click** `File` and `Save` on ATOM. This will save the playbook, and also ftp the playbook to Ansible server using pre-configured “**remote-sync**” package.


Note: In this new play, we used nxos module “**cisco.nxos.nxos_interfaces**” and “**cisco.nxos.nxos_l2_interfaces**”.

* “**cisco.nxos.nxos_interfaces**” provides the capability to manage the physical attributes of an interface
	* In this example, it is used to configure “layer 2” on interface Ethernet 1/3
* “**cisco.nxos.nxos_l2_interfaces**” provides the capability to manage the Layer 2 switchport attributes
	* In this example, it is used to configuration it is used to configure mode access on Ethernet ports 1/3
* We used “**when**” argument to provide little logic of the play.
	* In our example, the playbook assign VLAN 140 on leaf1 and leaf3 switches; assign VLAN 141 on leaf4 switch.

---
### Step 7: Execute playbook

Now, we are going to execute this playbook:

* Before executing the ansible playbook, you may log into switch (leaf1, leaf3 or leaf4) using MTPutty client, and check the existing configuration by executing the below command:
~~~  
show run interface ethernet1/3
~~~

* On MTPuttY, log back into (or launch a new ssh) into “Ansible” node

* Execute command `ansible-playbook vlan_provision.yml` in the “EVPN-Ansible” directory as shown below:
~~~		
[root@rhel7-tools EVPN-Ansible]# ansible-playbook vlan_provision.yml
~~~

Below screeenshot shows the execution of above command:

![](pic/2-7-1-new.png)

*  After we push the configuration, **login** to the **leaf-3** switch, and check if the server facing port has the access vlan configured with the below command:
~~~  
show run interface ethernet1/3
~~~

The output of above command on leaf-3 or leaf-1 is shown in below screenshot:

![](pic/2-7-2.png)

The output of above command on leaf-4 is shown in below screenshot:

![](pic/2-7-3.png)

---

**Congratulation! You have created your first ansible playbook, automatically provisioned new VLANs and assigned port to new created VLANs using Ansible. Next we are going to create VXLAN Fabric using Ansible.
**
