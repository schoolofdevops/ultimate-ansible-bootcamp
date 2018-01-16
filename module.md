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

	user    <------------ **Entity**
	  name  = xyz <---------- properties
	  state = present <---------- <span style="color:green">desired state</span>
	  uid   = 5001    <---------- properties
	  group = admins  <---------- properties

### INVOKING MODULES
