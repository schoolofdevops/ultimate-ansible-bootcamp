# Templates and Variables

In this  tutorial, we  are going to make the roles that we created earlier dynamically by adding templates and defining variables.

## Variables

Variables are of two types

* Automatic Variables/ Facts
* User Defined Variables

Lets try to discover information about our systems by using facts.

### Finding Facts About Systems

* Run the following command to see to facts of db servers  
```
cd chap7
ansible db -m setup
```

[Output]

```
192.168.61.11 | SUCCESS => {
        "ansible_facts": {
            "ansible_all_ipv4_addresses": [
                "10.0.2.15",
                "192.168.61.11"
            ],
            "ansible_all_ipv6_addresses": [
                "fe80::a00:27ff:fe30:3251",
                "fe80::a00:27ff:fe8e:83e0"
.....
                "tz_offset": "+0100",
                "weekday": "Monday",
                "weekday_number": "1",
                "weeknumber": "36",
                "year": "2016"
            }
```

### Filtering facts

* Use filter attribute to extract specific data

```
ansible db -m setup -a "filter=ansible_distribution"
```

[Output]

```
192.168.61.11 | SUCCESS => {
  "ansible_facts": {
      "ansible_distribution": "CentOS"
  },
  "changed": false
}  
```

## Defining release versions with vars

Currently, while deploying the application, the versions of the artifacts as well as release directories are defined statically. This should change to vars so that the version can be defined from one place, and dynamically so.

Define the default vars

file: roles/frontend/defaults/main.yml
```
---
# defaults file for frontend
  app:
    version: 1.5

```

Update tasks to use the var defined above,

`wherever you see version number e.g. 1.1, replace that with {{ app.version }} var.`

file: roles/frontend/tasks/main.yml

```
- name: Download and extract the release
  unarchive:
    src: https://github.com/devopsdemoapps/devops-demo-app/archive/{{ app.version }}.tar.gz
    dest: /opt/app/release
    owner: apache
    group: apache
    creates: /opt/app/release/devops-demo-app-{{ app.version }}
    remote_src: yes

- name: create a symlink
  file:
    src: /opt/app/release/devops-demo-app-{{ app.version }}
    dest: /var/www/html/app
    owner: apache
    group: apache
    state: link
```

**Try This**:
  * Run playbook and check whether the above code works
  * Change the version e.g. 1.4 and check if it has any effect


## Creating  application configurations dynamically

Application configs are defined with **config.ini** file.  The version of config.ini as shipped with the application is as follows,

```
[database]
hostname = DBHOST
username = SQLUSER
password = SQLPASSWORD
dbname = SQLDBNAME

[environment]
environment = ENVNAME

[prefs]
color  = white
fruit  = apple
car    = fiat
laptop = dell
```

You should be able to customize these configs. In order to do that, you need to do split this into 2 things as follows,

  * **vars** which define the actual properties and allow you to change it from different places
  * a **jinja2 template** which will collect and process the vars on the fly and create the resulting configs dynamically


### Defining the vars for app config  

file: roles/frontend/defaults/main.yml
```
---
# defaults file for frontend
  app:
    version: 1.5
    env: LOCALDEV

  fav:
    color: white
    fruit: orange
    car: chevy
    laptop: toshiba

  dbconn:
    host: localhost
    user: root
    pass: changeme
    db: devopsdemo
```

Create directory and template file.  You could either use the commands below or directly create it from the graphical editor.

```
cd roles/frontend
mkdir templates
touch templates/config.ini.j2
```

file: roles/frontend/templates/config.ini.j2  
```

[database]
hostname = {{ dbconn['host'] }}
username = {{ dbconn['user'] }}
password = {{ dbconn['pass'] }}
dbname = {{ dbconn['db'] }}

[environment]
environment = {{ app['env'] }}

[prefs]
color  = {{ fav['color'] }}
fruit  = {{ fav['fruit'] }}
car    = {{ fav['car'] }}
laptop = {{ fav['laptop'] }}

```

### Adding task to generate  the  config from jinja2 template

file: roles/frontend/tasks/main.yml ( append the following code to the file)

```
- name: add application configs
  template:
    src: config.ini.j2
    dest: /var/www/html/app/config.ini
    owner: apache
    group: apache
    mode: 0644
```


Now, run the playbook, reload the application page and validate. You should also browse to  http://IPADDRESS:81/app/prefs.php to view if it prints the preferences you defined in the default vars.   

```
cd chap7
ansible-playbook  app.yml
```


## Beyond defaults - Playing with vars precedence

Lets define the variables from couple of other places, to learn about the Precedence rules. We will create,
* group_vars
* playbook vars

Since we are going to define the variables using multi level hashes,  define the way hashes behave when defined from multiple places.

Update `chap7/ansible.cfg` and add the following,

```
hash_behaviour=merge
```

Lets create group_vars and create a group **prod** to define vars common to all prod hosts.

```
cd chap7
mkdir group_vars
cd group_vars
touch prod.yml
```

Edit **group_vars/prod.yml** file and add the following contents,

```
---
  fav:
    color: yellow
    fruit: guava
```

Lets also add vars to playbook. Edit app.yml and add vars as below,

```
---
  - hosts: app
    become: true
    vars:
      fav:
        fruit: mango
    roles:
      - apache
      - php
      - frontend
```

Execute the playbook and check the output

```
ansible-playbook app.yml
```

If you view the content of the html file generated, you would notice the following,

```
<h3> color     : yellow </h3>
<h3> fruit     : mango </h3>
<h3> car       : chevy </h3>
<h3> laptop    : toshiba </h3>
```


| fav item | role defaults     | group_vars     | playbook_vars |
| :------------- | :------------- | :------------- | :------------- |
| color | white | **yellow** |   |
| fruit | orange      |  guava     | **mango** |
| car | **chevy**       |        |  |
| laptop | **toshiba**       |        |  |

* value of color comes from group_vars/all.yml
* value of fruit comes from playbook vars
* value of car and laptop comes from role defaults

## Registered  Variables

Lets create a playbook to run a shell command, register the result and display the value of registered variable.

Create **register.yml** in chap7 directory

```
---
  - name: register variable example
    hosts: local
    tasks:
      - name: install net tools to make ifconfig command available
        package:
          name: net-tools
          state: installed

      - name: run a shell command and register result
        shell: "/sbin/ifconfig eth0"
        register: result

      - name: print registered variable
        debug: var=result
```

Execute the playbook to display information about the registered variable.

```
ansible-playbook  register.yml
```

## Adding support for Ubuntu

Apache role that we have developed supports only RedHat based systems at the moment. To add support for ubuntu (app2), we must handle platform specific differences.

e.g.

|   | RedHat | Debian     |
| :------------- | :------------- | :------------- |
| Package Name | httpd       | apache2       |
| Service Name | httpd       | apache2       |

OS specific configurations can be defined by creating role vars and by including those in tasks.

file: roles/apache/vars/RedHat.yml

```
---
apache:
  package:
    name: httpd
  service:
    name: httpd
    status: started
```

file: roles/apache/vars/Debian.yml

```
---
apache:
  package:
    name: apache2
  service:
    name: apache2
    status: started
```

Lets now selectively include those var files from tasks/main.yml .  Also selectively call configurations.
file: role/apache/tasks/main.yml

```
---
# tasks file for apache
  - include_vars: "{{ ansible_os_family }}.yml"
  - include: install.yml
  - include: service.yml
  - include: config_{{ ansible_os_family }}.yml  
```

We are now going to create two different config tasks. Since the current config is applicable to RedHat, lets rename it to config_RedHat.yml

```
mv roles/apache/tasks/config.yml roles/apache/tasks/config_RedHat.yml
```

We will now create a new config for Debian

file: roles/apache/tasks/config_Debian.yml

```
- name: Copying index.html file...
  template: >
    src=index.html.j2
    dest=/var/www/html/index.html
    mode=0777
```

Update tasks and handlers to install and start the correct service

tasks/install.yml

```
---
  - name: install httpd on centos
    package: >
      name={{ apache['package']['name']}}
      state=installed
```

tasks/service.yml

```
---
  - name: start httpd service
    service: >
      name={{ apache['service']['name']}}
      state={{ apache['service']['status']}}
```

handlers/main.yml

```
---
# handlers file for apache
  - name: restart apache service
    service: >
      name={{ apache['service']['name']}}
      state=restarted
```

Now add host app3 to the inventory

`file: environments/prod`

```
[app]
app1
app2
app3 ansible_password=codespaces
```

and apply playbook

```
ansible-playbook app.yml
```

## Exercises

* Create host specific variables in host_vars/HOSTNAME for one of the app servers, and define some variables values specific to the host. See the output after applying playbook on this node.
* Generate MySQL Configurations dynamically using templates and modules.
  * Create a template for my.cnf.  Name it as roles/mysql/templates/my.cnf.j2
  * Replace parameter values with templates variables
  * Define variables in role defaults.
