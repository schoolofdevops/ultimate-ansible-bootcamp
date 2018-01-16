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
such as,    
- packages files 
- services network interfaces 
- users & groups
- cron jobs 
- mount points
For Example,
	user 
	  name  = xyz 
	  state = present 
	  uid   = 5001 
	  group = admins

### INVOKING MODULES
