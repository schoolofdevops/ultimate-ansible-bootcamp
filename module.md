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

