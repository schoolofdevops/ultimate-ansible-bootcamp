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


### INVOKING MODULES
