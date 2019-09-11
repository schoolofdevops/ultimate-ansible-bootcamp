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

#### Problem Statement  3

##### Copy a file

- whose name is **"test.txt"** 
- to all app servers 
- at path **/tmp/test.txt** 
- whose permissions are set to **644**	

#

	module : copy
	src=test.txt  
	dest = /tmp/test.txt
	mode=644
	host pattern=app

	http://docs.ansible.com/ansible/copy_module.html

#

	ansible app -m copy -a "src=test.txt dest=/tmp/test.txt mode=644"

#

	app1 | SUCCESS => {
	     "changed": true,
	     "checksum": "dde78c32383a66378b6a8d7291deaed7ce4372bc",
	     "dest": "/tmp/test.txt",
	     "gid": 0,
	     "group": "root",
	     "md5sum": "ad80727e099718d25f8ece688b3c6efd",
	     "mode": "0644",
	     "owner": "root",
	     "size": 480,
	     "src": "/root/.ansible/tmp/ansible-tmp-1480603766.96-24842385908834 6/source",
	     "state": "file",     "uid": 0 
	} 
	app2 | SUCCESS => {
	     "changed": true,
	     "checksum": "dde78c32383a66378b6a8d7291deaed7ce4372bc",
	     "dest": "/tmp/test.txt",
	     "gid": 0,
	     "group": "root",
	     "md5sum": "ad80727e099718d25f8ece688b3c6efd",
	     "mode": "0644",
	     "owner": "root",
	     "size": 480,
	     "src": "/root/.ansible/tmp/ansible-tmp-1480603766.92-11939758265086 0/source",
	     "state": "file",
	     "uid": 0 
	}

#

	module : copy
	dest = /tmp/test.txt  
	host pattern = app  
	src = test.txt  
	mode = 644
 

#### COMMAND MODULES

It is recommended that you use a **specialized** **module** for the **entity** that you would like to manage or **action** you would want to take.......   

- copy a file <----------copy 
-  install a package on Redhat <-------yum 
-  manage vlan on Cisco NX OS <--------nxos_vlan 
-  create a  subnet on aws cloud<--------ec2_vpc__subnet

#

"however, there will be situations where you may not find **a module to take an action** you desire, or you may have a very complex script which you would like to invoke

- install an  application from source (make, make install)
- call an API for which there is no module 
- calling a ruby installer script for a third party application

to cover  such cases ansible offers you a group of **command modules**

##### CORE
**raw**
bypass modules subsystem and execute raw ssh command
#
**command**
execute a command on a remote node without shell
#
**shell**
execute a command on a remote node with /bin/sh
#
**script**
copy a script to the remote node and  also execute it
#
**expect**
execute a interactive command and auto respond to prompts

##### command

- does not use shell   
- does not have access to env >  <  |  ; &  operators will not work 
- secure and recommended

##### shell

- invokes /bin/sh 
- does not have access to env 
- >  <  |  ; &  operators will not work
- use selectively

#
Lets explore using command and shell modules

	ansible app -m command -a "free"

	ansible app -m command -a "free | grep -i swap"

	ansible app -m command -a "free" | grep -i swap

	ansible app -m shell -a "free | grep -i swap"

#
Learnthe difference between raw and shell

	ansible app -vvvv -m raw -a "free | grep -i swap"   
	ansible app -vvvv -m shell -a "free | grep -i swap"
	 
#### IDEMPOTENCE

Lets run a command to create a directory on remote nodes

	ansible app -m command -a "mkdir /tmp/dir1"

#

	ansible app -m command -a "mkdir /tmp/dir1"                                                                    
	
	app2 | SUCCESS | rc=0 >>
	app1 | SUCCESS | rc=0 >>
#
	Correct                

#
Lets run it again

	ansible app -m command -a "mkdir /tmp/dir1"
#
What is the result ?

	ansible app -m command -a "mkdir /tmp/dir1"                                                                    
	app1 | FAILED | rc=1 >>                                                
	mkdir: cannot create directory `/tmp/dir1': File exists                
                                                                       
	app2 | FAILED | rc=1 >>                                                
	mkdir: cannot create directory `/tmp/dir1': File exists                                                                                           

#
	Incorrect WHY ?
#

	ansible app -m command -a "mkdir /tmp/dir1"                                                                    
	app1 | FAILED | rc=1 >>                                                
	mkdir: cannot create directory `/tmp/dir1': File exists <-------file already exists                
                                                                       
	app2 | FAILED | rc=1 >>                                                
	mkdir: cannot create directory `/tmp/dir1': File exists 
#

- most shell utilities are not idempotent by nature 
- ansible's command modules simply invoke shell commands, and are not idempotent either

#### MAKING COMMANDS IDEMPOTENT
**HOW ?**
#
**CREATES**
#
- creates is a parameters/argument to the command modules
- checks for an existence of a file and decides to skip execution if the file is present

	creates=/tmp/dir1
#

	is file present ?----------SKIP
		|	   YES	
		|
		|
		|NO
		|
		|
	     EXECUTE

#
Run command with creates

#

	ansible app -m command -a "mkdir /tmp/dir1 creates=/tmp/dir1"

#
    

	app2 | SUCCESS | rc=0 >>                                                                        
	skipped, since /tmp/dir1 exists                                                                 
                                                                                                
	app1 | SUCCESS | rc=0 >>                                                                        
	skipped, since /tmp/dir1 exists   
