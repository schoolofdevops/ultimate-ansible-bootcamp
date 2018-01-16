
Modules  
## Topics Included:-  
* Desired State Configurations 
* Invoking Modules 
* Using Common Modules 
* Command Modules and Idempotence  
### Desired State Configuration  
**Desired State** :-             Once You decribe **WHAT** you want  using **desired state** configurations you need not to worry about **How** the state is acheived,**whether** to take an action on which action to take **It's Ansible's Job**  	
	user  <---------- Entity 	
	name = XYZ <----------Properties 	
	State = Present <----------  Desired State 	
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
	ansible app -m yum -s -a "name=ntp  state installed"   
* -m yum ---------> Module name  
* name=ntp ---------> key=value  
* state  installed ---------> arguments  
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
### Command Modules 
 **Command** :-  
* Does not use Shell 
* Does not have access to env 
* > < | ; & operators will not work 
* Secure and recommended  
**Shell** :-  
* Invokes /bin/sh 
* Has access to env, variables etc 
* > < | ; & operators will work 
* Use selectively
