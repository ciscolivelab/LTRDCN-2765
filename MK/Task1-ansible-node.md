Your first task will be to build an Ansible node on a server running redhat CentOS operating system.  At the end of this task, you will have a fully operational Ansible node.

#### Step 1: Connect to lab using anyconnect VPN
You will connect to **dcloud-lon-anyconnect.cisco.com** using Cisco VPN AnyConnect client, as shown in below picture, with the username and password provided by the lab admin.

|Pod ID|Attendee Name|Anyconnect url|Anyconnect Username|Anyconnect Password|
|---|---|---|---|---|
POD  1 ||dcloud-lon-anyconnect.cisco.com||
POD  2 ||dcloud-lon-anyconnect.cisco.com||
POD  3 ||dcloud-lon-anyconnect.cisco.com||
POD  4 ||dcloud-lon-anyconnect.cisco.com||
POD  5 ||dcloud-lon-anyconnect.cisco.com||
POD  6 ||dcloud-lon-anyconnect.cisco.com||
POD  7 ||dcloud-lon-anyconnect.cisco.com||
POD  8 ||dcloud-lon-anyconnect.cisco.com||
POD  9 ||dcloud-lon-anyconnect.cisco.com||
POD  10 ||dcloud-lon-anyconnect.cisco.com||
POD  11 ||dcloud-lon-anyconnect.cisco.com||
POD  12 ||dcloud-lon-anyconnect.cisco.com||
POD  13 ||dcloud-lon-anyconnect.cisco.com||
POD  14 ||dcloud-lon-anyconnect.cisco.com||
POD  15 ||dcloud-lon-anyconnect.cisco.com||
POD  16 ||dcloud-lon-anyconnect.cisco.com||


**Note:** lab admin will furnish the credentials information to the participant.  If you don't have this information please ask the lab speakers.


![](pic/1-1-1-lon.png)

#### Step 2: Enter VPN credentials

After prompted for credentials, use the credentials provided by the lab admin. 		
•	Below is an example of user logging into a reference POD:

![](pic/1-2-vpn-credentials.png)

*  Hit accept when the prompt appears to accept the VPN connection login

![](pic/1-2-vpn-accept.png)


#### Step 3: RDP to workstation

In this step, you will connect to the workstation with RDP client on your machines.  Use below details for this RDP session:

*  Workstation: **198.18.133.36**
*  Username: **dcloud\demouser**
*  Password: **C1sco12345**

Below screenshot is only an example for this RDP connection:

![](pic/1-3-RDP.png)


#### Step 4: MTputty

Once you have the RDP session to the remote workstation, then you will use MTputty client to connect to all devices in this lab.  

MTputty is already installed on the Desktop of the workstation where you connected using RDP.  Run this application by clicking on the icon on the desktop:

![](pic/1-4-mtputty.png)

#### Step 5: SSH into Ansible node

 SSH into Ansible node (198.18.134.150) by double clicking the Ansible icon on the left pan with username **root** and password **C1sco12345**

![](pic/1-5-mtputty-open.png)


#### Step 6: Verify Python

Once successfully SSH into the ansible node, the very first thing we are going to do after logging into Ansible server is verify the python version by running `python --version` command - as shown below:

	[root@rhel7-tools ~]# python --version

It is an important step as we need minimum 2.7.5 version of python in order to install some features for ansbile.  The output of above command confirms this version.

Ansible can be run from any machine with Python 2 (versions 2.6 or 2.7) or Python 3 (versions 3.5 and higher) installed.

![](pic/1-6-python.png)

#### Step 7: Install PIP

After verifying we have the minimum version of python installed, we are now going to Install PIP python package using `easy_install pip` command as shown below:

	[root@rhel7-tools ~]# easy_install pip

Below screenshot shows the execution of above command:

![](pic/1-6-2-python-new.png)

Next, update pip to latest version by executing command `pip install --upgrade pip` as shown below:

	[root@rhel7-tools ~]# pip install --upgrade pip

**Note:** you can ignore the warning related to python version.  Below screenshot shows the execution of above command:

![](pic/1-6-3-python-new.png)


After installing PIP package, we are going to add the relevant packages that are needed for this Ansible based VXLAN lab. Below are the packages required for this lab.  

*  Paramiko
*  PyYAML
*  Jinj2
*  Httplib2

Run the command `pip install paramiko PyYAML jinja2 httplib2`, as shown below, to install these packages:

	[root@rhel7-tools ~]# pip install paramiko PyYAML jinja2 httplib2

Below screenshot shows the execution of above command:

![](pic/1-6-4-python-new.png)

As a final step, we are going to install Ansible on this RHEL. Once the install is initiated by using `pip install ansible==2.10.6` command (as shown below).  It may take few minutes for it to download and install.

	[root@rhel7-tools ~]# pip install ansible==2.10.6

Below screenshot shows the execution of above command:

![](pic/1-6-5-ansible-new.png)

Starting from Ansible 2.9, plugins and modules for network platform are moved in to **collection**. To install NX-OS collection with Ansible Galaxy by using `ansible-galaxy collection install cisco.nxos`

	[root@rhel7-tools ~]# ansible-galaxy collection install cisco.nxos

Below screenshot shows the execution of above command:

![](pic/1-7-4-collection.PNG)

#### Step 8: Verify Ansible

After installation is complete, check Ansible version by executing command `ansible --version`, as shown below:

	[root@rhel7-tools ~]# ansible --version

Below screenshot shows the execution of above command:

![](pic/1-8-1-version-new.png)

**Note:**: Your output might show different version than the installed version. This is due to the separation of ansible-base (ansible-core) and community package starting from version 2.10. The above command shows ansible-base version. You can find more detail explanation from Ansible documentation page.
[https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

#### Step 9: Create Ansible Inventory

Now, we are going to create inventory, host variables and Configuration file. This is important as Ansible works  against multiple systems in the system by selecting portions of systems listed in Ansible inventory. Similarly, configuration settings in Ansible are adjustable via configuration file.

Create folder named EVPN-Ansible as working environment and verify that it’s empty:

	[root@rhel7-tools ~]# mkdir EVPN-Ansible && cd EVPN-Ansible
	[root@rhel7-tools EVPN-Ansible]# ls

Below screenshot shows the execution of above command:

![](pic/1-9-1-dir.png)

Next:

* Create Ansible inventory file to include Spine and Leaf switches.
* By default Ansible has inventory file saved in location /etc/ansible/hosts.
* In this lab we will create hosts file in the working environment. This file is created from ansible host prompt using `cat` command.   ***Note:*** you should copy and paste the complete section below i.e., starting from `cat` till `EOF` (also shown in subsequent screenshot):

		cat << EOF >> hosts
        #define global variables, groups and host variables
        [all:vars]
        ansible_connection = ansible.netcommon.network_cli
        ansible_network_os = cisco.nxos.nxos
        ansible_user=admin
        ansible_password=C1sco12345
        ansible_command_timeout=180
        gather_fact=no
        [jinja2_spine]
        198.18.4.202
        [jinja2_leaf]
        198.18.4.104
        [spine]
        198.18.4.201
        [leaf]
        198.18.4.101
        198.18.4.103
        [server]
        198.18.134.50 eth1=172.21.140.10 gw=172.21.140.1
        198.18.134.52 eth1=172.21.140.11 gw=172.21.140.1
        198.18.134.53 eth1=172.21.141.11 gw=172.21.141.1
		EOF


Below screenshot shows the output of above command:

![](pic/1-9a-1-hosts.png)

* Now you may verify the content of this file using below command:

		[root@rhel7-tools EVPN-Ansible]# more hosts

Below screenshot shows the execution of above command:
![](pic/1-9a-2-hosts.png)


*  Create Ansible config (ansible.cfg) using the same steps as above.  ***Note:*** you should copy and paste the complete section below i.e., starting from `cat` till `EOF`  (as shown in below screenshot):

		cat << EOF >> ansible.cfg
		[defaults]
		inventory = hosts
		host_key_checking = false
		record_host_key = true
		stdout_callback = debug
		deprecation_warnings = False
		EOF

* Now you may verify the content of this file using below command:

		[root@rhel7-tools EVPN-Ansible]# more ansible.cfg

Below screenshot shows the output of above commands:

![](pic/1-9a-3-ansible.png)



*  Do `ls` to verify the file that you just created under project folder EVPN-Ansible.   

	Below screenshot shows the execution of above command:

	![](pic/1-9-3-ls.png)


*  Create host variable folder named `host_vars` in folder `EVPN-Ansible` by using below command.  In this lab, we will use host_vars file to define the variables for various hosts (in next bullets):

		[root@rhel7-tools EVPN-Ansible]# mkdir host_vars && cd host_vars

*  Create host variable file for each host in inventory by using the cat command.  The variable file for a switch is created using the below cat command.  **Note:** you can copy and paste starting from `cat` till `EOF` (as shown in below screenshot).  **Note:** the spaces in the file are important so do not remove those:


~~~YAML
cat << EOF >> 198.18.4.101.yml
---
  hostname: leaf_1
  loopback0: 192.168.0.8
  loopback1: 192.168.0.18
  router_id: 192.168.0.8
EOF
~~~

Below screenshot shows the execution of above command:

![](pic/1-9a-4-101.png)

* Now you may verify the content of this file using `more 198.18.4.101.yml` as shown below:

		[root@rhel7-tools  host_vars]# more 198.18.4.101.yml

Below screenshot shows the execution of above command:

![](pic/1-9a-4-101-more.png)



*  Create a new host variable file for next host in inventory.   The variable file for a switch is created using the below cat command.  **Note:** you can copy and paste starting from `cat` till `EOF` (as shown in below screenshot).  **Note:** the spaces in the file are important so do not remove those:

~~~ YAML
cat << EOF >> 198.18.4.103.yml
---
  hostname: leaf_3
  loopback0: 192.168.0.10
  loopback1: 192.168.0.110
  router_id: 192.168.0.10
EOF
~~~

Below screenshot shows the execution of above command:

![](pic/1-9a-4-103.png)

* Now you may verify the content of this file using `more 198.18.4.103.yml` as shown below:

		[root@rhel7-tools  host_vars]# more 198.18.4.103.yml

Below screenshot shows the execution of above command:

![](pic/1-9a-4-103-more.png)


*  Create a new host variable file for next host in inventory. The variable file for a switch is created using the below cat command.  **Note:** you can copy and paste starting from `cat` till `EOF` (as shown in below screenshot).  **Note:** the spaces in the file are important so do not remove those:

~~~ YAML
cat << EOF >> 198.18.4.104.yml
---
  hostname: leaf_4
  loopback0: 192.168.0.11
  loopback1: 192.168.0.111
  router_id: 192.168.0.11
EOF
~~~

* Now you may verify the content of this file using `more 198.18.4.104.yml` as shown below:

		[root@rhel7-tools host_vars]# more 198.18.4.104.yml

Below screenshot shows the execution of above commands:

![](pic/1-9a-4-104.png)


*  Create a new host variable file for next host in inventory.  The variable file for a switch is created using the below cat command.  **Note:** you can copy and paste starting from cat till EOF (as shown in below screenshot).  **Note:** the spaces in the file are important so do not remove those:

~~~ YAML
cat << EOF >> 198.18.4.201.yml
---
  hostname: spine-1
  loopback0: 192.168.0.6
  loopback1: 192.168.0.100
  router_id: 192.168.0.6
EOF
~~~

* Now you may verify the content of this file using `more 198.18.4.201.yml` as shown below:

		[root@rhel7-tools host_vars]# more 198.18.4.201.yml

Below screenshot shows the execution of above commands:

![](pic/1-9a-4-201.png)


*  Create a new host variable file for next host in inventory.  The variable file for a switch is created using the below cat command.  **Note:** you can copy and paste starting from cat till EOF (as shown in below screenshot).  **Note:** the spaces in the file are important so do not remove those:


~~~ YAML
cat << EOF >> 198.18.4.202.yml
---
  hostname: spine-2
  loopback0: 192.168.0.7
  loopback1: 192.168.0.100
  router_id: 192.168.0.7
EOF
~~~

* Now you may verify the content of this file using `more 198.18.4.202.yml` as shown below:

		[root@rhel7-tools host_vars]# more 198.18.4.202.yml

Below screenshot shows the execution of above commands:

![](pic/1-9a-4-202.png)

#### Step 10: Ansible role structure

Role is very useful technique to manage a set of playbooks in Ansible.
In this lab, we will use two different playbooks to manage configuration for Spine and Leaf switches.

We will use role structure and manage the two plays into single playbook.
A role directory structure contains several directories of defaults, vars, files, handlers, meta, tasks and templates.

In this lab:

*  we will use vars, templates and tasks folders
*  main.yml file in `/vars`  folder contains dictionary of variables for this role
*  main.yml file in `/tasks` folder contains the Ansible playbook for this role

To proceed further with roles in subsequent Tasks:

•	Create roles directory in folder EVPN-Ansible by issuing below commands:

	[root@rhel7-tools host_vars]# cd /root/EVPN-Ansible
	[root@rhel7-tools EVPN-Ansible]# mkdir roles

Below screenshot shows the execution of above commands:

![](pic/1-10-1.png)

This will be used in the subsequent tasks in this lab.
