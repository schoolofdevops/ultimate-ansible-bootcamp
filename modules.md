# Modules  

## Topics Included:-  
* Desired State Configurations 
* Invoking Modules 
* Using Common Modules 
* Command Modules and Idempotence  

### Desired State Configuration  
**Desired State** :-             Once You decribe **WHAT** you want  using **desired state** configurations you need not to worry about **How** the state is acheived,**whether** to take an action on which action to take **It's Ansible's Job** 

  	user  <---------- Entity
 	name = XYZ <---------- Properties
 	State = Present <---------- Desired State
 	uid = 5001 <---------- Properties
 	group = admins <---------- Properties
### Module Categories

* System
* Cloud 
* Networking
* Utilities
* Files
* Database
* Inventory
* Messaging
* Remote management

### Module Types

* Core
* Extras

#### Core :-
  
* Maintained by ansible team
* Will always be shipped with ansible

#### Extras :-
  
* Maintained by community 
* Currently shipped with ansible 
* May be shipped separately in future  

### Invoking Modules  

##### Usage :-  Modules are typically executed as a part of ansible command.

  	ansible app -m yum -s -a "name=ntp  state=installed"   

* -m yum ---------> Module name
* name = ntp ---------> key=value  
* state = installed ---------> arguments  

Modules can also be called while writing tasks in **Playbook** using **YAML**

	- name : install ntp package
	 yum : >
         name: ntp
	 state: present  

**Output**
  	app1 | SUCCESS => { 
        "changed": false,
         "ping": "pong"
     }  
#### FINDING INFO

**ansible-doc** :- This utility helps you find list of modules, how to use those along with example snippets.
  	ansible-doc --help
 	ansible-doc --list | head
 	ansible-doc user
 	ansible-doc -s user  
**Invoking A module**
 
_Lets Invoke a Module for_  
* Installing Vim Utility 
* On **Load Balancer** 
* Whose OS is CentOS  

**Running Ad-hoc Command** (Do not Run This)
  	ansible lb -s -a "yum install -y vim"  

**Using Module**
  	ansible lb -s -m yum -a "name=vim state=present"
  
### Using Common Modules  

#### Common Modules :-
1. Packages
* yum
* apt   
* gem
* pip
2. Files
* copy    
* fetch    
* template   
3. System     
* user    
* group    
* cron    
* mount    
* ping   
4. Utilities     
* debug    
* assert    
* wait_for  
5. Commands     
* command   
* shell    
* expect 

## GE  

**Problem Statement 1** :-  

**Create a group**  
* on all prod servers 
* whose name is "**admin**" 
* whose gid is "**7045**"  

**Use this Command**
 	 	ansible prod -s -m group -a "name=admin gid=7045 state=present"   

**Thus it states**:-  
* module = group 
* name = admin 
* gid = 7045 
* state = present 
* host pattern = prod  

**Problem Statement 2** :-  

**Create a user**  
* On all prod servers 
* Except for db hosts 
* Whose name is "**abc**" 
* Whose uid is "**7001**" 
* Who belongs to "**admin**" group 
* Who has a home directory  

**Use this Command**
  	ansible 'prod:!db' -s -m user -a "name=abc uid=7001 group=admin state=present"  

**Thus it states**:-  
* module = **user** 
* name = abc 
* state = present 
* uid = 7001 
* group = admin 
* host pattern = prod:!db  

**Problem Statement 3** :-  

**Copy a File** :-  
* Whose name is "**test.txt**" 
* To all app servers 
* At path /tmp/test.txt 
* Whose permissions are set to 644  

**Use this Command**
  	ansible app -m copy-a "src=test.txt dest=/tmp/test.txt mode=644"  

**Thus it states**:-  
* module = **copy** 
* src = test.txt 
* dest = /tmp/test.txt 
* mode = 644 
* host pattern = app   

### Command Modules  

**Command** :-  
* Does not use Shell 
* Does not have access to env 
* This > < | ; & operators will not work 
* Secure and recommended  

**Shell** :-  
* Invokes /bin/sh 
* Has access to env, variables etc 
* This > < | ; & operators will work 
* Use selectively  

## GE  

**Lets explore using command and shell modules**
  	ansible app -m command -a "free"
  	ansible app -m command -a "free | grep -i swap"
  	ansible app -m command -a "free" | grep -i swap
  	ansible app -m shell -a "free | grep -i swap"

**Learn the difference between raw and shell**
  	ansible app -vvvv -m raw -a "free | grep -i swap"
  	ansible app -vvvv -m shell -a "free | grep -i swap"

**Lets run a command to create a directory on remote nodes**
  	ansible app -m command -a "mkdir /tmp/dir1"  

**Output**
	ansible app -m command -a "mkdir /tmp/dir1"
	app1 | SUCCESS | rc=0 >>
	app2 | SUCCESS | rc=0 >> 
