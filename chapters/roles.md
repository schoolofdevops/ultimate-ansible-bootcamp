# Configuring app server environment with Roles

In the previous chapter, you have created and applied playbook for base systems configurations. Now is the time to start creating modular, reusable library of code for application configurations. In this chapter, we are going to write such modular code, in the form of roles and setup application server.  

We are going to create the roles with following specs,

apache role which will

  * Install **httpd** package
  * Start httpd service
  * Add a handler to restart service

**php** role to

  * install **php** and **php-mysql**
  * restart apache when packages are installed

We will also refactor  **systems.yml** and move all the tasks to its own role i.e. **systems**


### Creating Role Scaffolding for Apache  
  * Change working  directory to **chap6**

```
cd  chap6
```

  * Create roles directory

```
mkdir roles
```

  * Generate role scaffolding using ansible-galaxy

```
ansible-galaxy init --offline --init-path=roles  apache
```

  * Validate

```
tree roles/
```

[Output]

```
  roles/
    └── apache
        ├── defaults
        │   └── main.yml
        ├── files
        ├── handlers
        │   └── main.yml
        ├── meta
        │   └── main.yml
        ├── README.md
        ├── tasks
        │   └── main.yml
        ├── templates
        ├── tests
        │   ├── inventory
        │   └── test.yml
        └── vars
            └── main.yml

```


### Writing Tasks to Install and Start  Apache Web Service

We are going to create three different tasks files, one for each phase of application lifecycle
  * Install
  * Configure
  * Start Service

To begin with, in this part, we will install and start apache.

  *  To install apache, Create **roles/apache/tasks/install.yml**

```
---
  - name: install apache web server
    yum:
      name: httpd
      state: installed
```  


  * To start the service, create  **roles/apache/tasks/service.yml** with the following content  

```
---
  - name: start apache webserver
    service:
      name: httpd
      state: started
      enabled: true
```  


To have these tasks being called, include them into main task.

  * Edit roles/apache/tasks/main.yml

```
---
# tasks file for apache
  - import_tasks: install.yml
  - import_tasks: service.yml
```

### Create a role to install php

Generate roles scaffold
```
ansible-galaxy init --offline --init-path=roles  php
```

roles/php/tasks/install.yml
```
---
# install php related packages
  - name: install php
    package:
      name: "{{ item }}"
      state: installed
    with_items:
      - php
      - php-mysql
```  

file: roles/php/tasks/main.yml
```
---
# tasks file for php
- import_tasks: install.yml
```

#### Adding Notifications and Handlers   

  * Previously we have create a task to install php related packages.  After install these packages, to make it effective, its important to restart apache web server. To achieve this,  update the task to send a notification to the handler created above to restart  apache.  You simply have to add the line below which starts with **notify**

```
- name: install php
  package:
    name: "{{ item }}"
    state: installed
  with_items:
    - php
    - php-mysql
  notify: Restart apache service

```  



  * Create the notification handler by updating   **roles/apache/handlers/main.yml**  

```
---
# handlers file for apache
  - name: Restart apache service
    service:
      name: httpd
      state: restarted
```  


### Create and apply playbook to configure app servers

  * Create a playbook for app servers **app.yml** with following contents

```
  ---
  - hosts: app
    become: true
    roles:
      - apache
      - php
```

  * Apply app.yml with ansible-playbook

```
  ansible-playbook app.yml
```

[Output]

```
PLAY [Playbook to configure App Servers] *********************************************************************

TASK [setup] *******************************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [apache : Install Apache...] **********************************************
changed: [192.168.61.13]
changed: [192.168.61.12]

TASK [apache : Starting Apache...] *********************************************
changed: [192.168.61.13]
changed: [192.168.61.12]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=3    changed=2    unreachable=0    failed=0
192.168.61.13              : ok=3    changed=2    unreachable=0    failed=0
```


### Systems role, dependencies and nested roles

You have already written a playbook to define common systems configurations. Now, go ahead and refactor it so that instead of calling tasks from playbook itself, it goes into its own role, and then call on each server.

  * Create a base role with ansible-galaxy utility,  
```
  ansible-galaxy init --offline --init-path=roles systems
```  

  * Copy over the  tasks from **systems.yml** and lets just add it to   **/roles/base/tasks/main.yml**  

```
---
# tasks file for systems
  - name: remove user dojo
    user: >
      name=dojo
      state=absent

  - name: install tree utility
    yum: >
      name=tree
      state=present

  - name: install ntp
    yum: >
      name=ntp
      state=installed

```  

  * Define systems role as a dependency for  apache role,  
  * Update meta data for Apache by editing **roles/apache/meta/main.yml** and adding the following
```
---
dependencies:
 - {role: systems}
```  

Next time you run  **app.yml**, observe if the above tasks get invoked as well.




### Creating a Site Wide Playbook

We will create a site wide playbook, which will call all the plays required to configure the complete infrastructure. Currently we have a single  playbook for App Servers. However, in future we would create many.

  * Create **site.yml** in /vagrant/chap5 directory and add the following content

```
  ---
  # This is a sitewide playbook
  # filename: site.yml
  - include: app.yml

```  



  * Execute sitewide playbook as


```
ansible-playbook site.yml
```

[Output]

```

PLAY [Playbook to configure App Servers] ***************************************

TASK [setup] *******************************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : create admin user] ************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : remove dojo] ******************************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [base : install tree] *****************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [base : install ntp] ******************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [base : start ntp service] ************************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Installing Apache...] *******************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Starting Apache...] *********************************************
ok: [192.168.61.13]
ok: [192.168.61.12]

TASK [apache : Copying configuration files...] *********************************
ok: [192.168.61.12]
ok: [192.168.61.13]

TASK [apache : Copying index.html file...] *************************************
ok: [192.168.61.12]
ok: [192.168.61.13]

PLAY RECAP *********************************************************************
192.168.61.12              : ok=10   changed=0    unreachable=0    failed=0
192.168.61.13              : ok=10   changed=0    unreachable=0    failed=0
```


## Exercises

#### Nano Project: Deploy a PHP Application
devops-demo-app is an application written in  PHP. You have already setup the environment above with apache and php roles, to deploy this application.  Your job is to write the ansible code to  deploy this application on app servers. This code will be in the form of a role.

You have been tasked to create a **froentend**  role with the following specs,

  * Pull release packages from the github release page as provided in the resources below. Releases are in the form of an archive.
  * Multiple copies of releases will be maintained on the app servers for enabling rollbacks. To support this, every time to deploy a new version of the app, create a new directory for it inside the /opt/app
```
e.g.

opt
  | __ app
         \__ release
                 |
                 |____ devops-demo-app-1.0
                 |
                 |____ devops-demo-app-1.1

```


  * Create  a symlink from the current version path to  **/var/www/html/app**



```
e.g.

[root@app2 /]# ls -l /var/www/html/
total 8
lrwxrwxrwx 1 root root   36 Jan 16 13:28 app -> /opt/app/release/devops-demo-app-1.1

```


**Resources**:

  * **PHP App Source** : https://github.com/devopsdemoapps/devops-demo-app
  * **Releases**: https://github.com/devopsdemoapps/devops-demo-app/releases


Once deployed, visiting  ![http://IADDRESS:81/app](http://IADDRESS:81/app) for app1 or with port 82 for (app2) should show the web app deployed.


#### Nano Project: Create MySQL Role
  * Create a Role to install and configure MySQL server   
     *    Create role scaffold for mysql  using ansible-galaxy init  
     *   Create task to install "mysql-server" and "MySQL-python" packages using yum module   
     *    Create a task to start mysqld service   
     *   Manage my.cnf by creating a centralized copy in role and writing a task to copy it to all db hosts. Use helper/my.cnf as a reference. The destination for this file is /etv/my.cnf on db servers.
     *    Write a handler to restart the service on configuration change. Add a notification from the copy resource created earlier.
     * Add a dependency on base role in the metadata for mysql role.  
     *     Create  **db.yml** playbook for configuring all database servers. Create definitions to configure **db** group and to apply **mysql** role.   
