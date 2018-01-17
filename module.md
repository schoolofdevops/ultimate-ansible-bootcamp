# MODULES
### TOPICS
- Desired State Configurations
- Invoking Modules  
- Using Common Modules  
- Command Modules and Idempotence
### DESIRED STATE
#### PROCEDURAL
	useradd -m  abc
#### DESIRED STATE
Modules allow us to  manage independent components or entities of our infrastructure
#

such as,    

- packages files 
- services network interfaces 
- users & groups
- cron jobs 
- mount points
#

For Example,

	user    <------------ Entity
	  name  = xyz <---------- properties
	  state = present <---------- desired state
	  uid   = 5001    <---------- properties
	  group = admins  <---------- properties

Once you describe **WHAT** you want

  using **desired state** configurations

  you need not worry about **how** the state is achieved, 
**whether** to take an action and **which** action to take
  
Its **Ansible's** Job

#### MODULE CATAGORIES

- SYSTEM CLOUD 
- COMMAND NETWORKING 
- PACKAGING CLUSTERING 
- DATABASE FILES 
- INVENTORY 
- MESSAGING 
- NOTIFICATION 
- SOURCE CONTROL 
- REMOTE MANAGEMENT
####MODULES TYPES
##### CORE 
- maintained by ansible team 
- will always be shipped with ansible  
##### EXTRA
- maintained by community 
- currently shipped with ansible 
- may  be shipped separately in future

### INVOKING MODULES
##### USAGE
Modules are typically executed as a part of ansible command

	ansible app  -m yum -s -a  "name=ntp     state=installed"
	             ------------  --------     ---------------
	            module name    key = value  arguments  params
##### USAGE
Modules can also be called while writing tasks in **Playbooks** using **YAML**

	- name: install ntp package 
	  yum: >     
		name: ntp     
	  state: present
#
Module output is JSON data

	app1 | SUCCESS => {     
        "changed": false,     
        "ping": "pong" 
	}

This is useful when you write your own modules, as you **need not stick to python** to create a custom module. 
Only requirement is you take inputs and outputs in the format Ansible recognizes.

#### FINDING INFO

#### FINDING INFO 

**ansible-doc**

This utility helps you find list of modules, how to use those along with example snippets.

	ansible-doc --help
  
	ansible-doc --list | head
  
	ansible-doc user
  
	ansible-doc -s user

##### INVOKING A MODULE

Lets use a module for

- Installing **vim** utility 
- on load **balancer** 
- whose OS is **CentOS**

##### PROCEDURAL VS DESIRED STATE

Running Ad Hoc Command

	ansible lb -s -a "yum install -y vim"
#
#### USING MODULE
	ansible lb  -s -m yum -a "name=vim  state=present"
                     module          attributes
                                    (properties)

##### INVOKING A MODULE

	ansible db -s -m yum -a "name=vim  state=present"
#
 
	db | SUCCESS => { 
            "changed": true,
            "msg": "",
            "rc": 0,
            "results": [
                "Loaded plugins: fastestmirror, ovl\nSetting up Install Process
        \nLoading mirror speeds from cached hostfile\n * base: mirror.fibergrid
       .in\n * extras: mirror.fibergrid.in\n * updates: mirror.fibergrid.in\nR

#### RERUN 

 	ansible db -s -m yum -a "name=vim  state=present"

#

	db | SUCCESS => {
	      "changed": true,
	      "msg": "",
	      "rc": 0,
	      "results": [
	          "Loaded plugins: fastestmirror, ovl\nSetting up Install Process
	  \nLoading mirror speeds from cached hostfile\n * base: mirror.fibergrid 
	 .in\n * extras: mirror.fibergrid.in\n * updates: mirror.fibergrid.in\nR


Thats **idempotence** in action  Ansible modules(most)  are idempotent.  Ansible modules will compare the desired state (policy we set)  with the current state of the entity on the system, and decide whether the action is necessary.

#### USING COMMON MODULES

##### COMMON MODULES

**PACKAGES**

- yum 
- apt 
- gem
- pip

**Files** 

- copy 
- fetch 
- template  

**System** 

- user 
- group 
- cron 
- mount 
- iptables 
- ping

**Utilities** 

- debug
-  assert 
- wait_for  

**Commands** 

- command 
- shell 
- expect


#### COMMAND MODULES

It is recommended that you use a **specialized** **module** for the **entity** that you would like to manage or **action** you would want to take.......


- copy a file <----------copy
-  install a package on Redhat <-------yum
-  manage vlan on Cisco NX OS <--------nxos_vlan
-  create a  subnet on aws cloud<--------ec2_vpc__subnet

##### Problem Statement  1

##### Create a group 

-  on all prod servers
-  whose name is "admin" 
- whose gid  is "7045"

	
	module:group
	name=admin
	state=present
	git=705
	host pattern=prod
	
	http://docs.ansible.com/ansible/group_module.html

#### APPROACH

- Find the module which could manage the entity in question
- Observe the usage documentation and learn about the parameters and examples
- Determine the desired state  
- Determine the parameters to use and the values to assign


	ansible prod -s -m group -a "name=admin gid=7045 state=present"

#

	lb | SUCCESS => {
	     "changed": true,
	     "gid": 7045,
	     "name": "admin",
	     "state": "present",
	     "system": false 
	} 
	app1 | SUCCESS => {
	     "changed": true,
	     "gid": 7045,
	     "name": "admin", 
	     "state": "present",
	     "system": false 
	} 
	db | SUCCESS => { 
	    "changed": true,
	     "gid": 7045,
	     "name": "admin",
	     "state": "present",
	     "system": false
	 } 
	app2 | SUCCESS => {
	     "changed": true,
	     "gid": 7045,
	     "name": "admin",
	     "state": "present",
	     "system": false 
	}

#

	module :  group  
	state = present  
	gid = 7045  
	host pattern = prod  
	name = admin

#### Problem Statement  2

- on all prod servers
- except for db hosts
- whose name is **"abc"** 
- whose uid  is **"7001"** 
- who belongs to **admin** 
- group who has a home directory

#

	module=user
	name=abc
	stae=present
	uid=7001
	group=admin
	host pattern='prod:!db'

	http://docs.ansible.com/ansible/user_module.html

#

	ansible 'prod:!db' -s -m user -a "name=abc uid=7001 group=admin state=present"

#

	app2 | SUCCESS => {
	     "changed": true,
	     "comment": "",
	     "createhome": true,
	     "group": 7045,
	     "home": "/home/abc",
	     "name": "abc",
	     "shell": "/bin/bash",
	     "state": "present",
	     "system": false,
	     "uid": 7001 
	} 
	lb | SUCCESS => {
	     "changed": true,
	     "comment": "",
	     "createhome": true,
	     "group": 7045,
	     "home": "/home/abc",
	     "name": "abc",
	     "shell": "/bin/bash",
	     "state": "present",
	     "system": false,
	     "uid": 7001 
	} 
	app1 | SUCCESS => {
	     "changed": true,
	     "comment": "",
	     "createhome": true,
	     "group": 7045,
	     "home": "/home/abc", 
	     "name": "abc",
	     "shell": "/bin/bash",
	     "state": "present",
	     "system": false,
	     "uid": 7001 
	}
#

	module=user
	name=abc
	state=present
	uid=7001
	group=admin
	host pattern='prod:!db'	

	


	
