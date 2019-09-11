# Control Structures

In this chapter, we will learn about the aspects of ansible that can change the  execution flow. This includes,

* Iterators/loops  
* Conditionals
* Tags
* Conditional inclusions


## Refactoring systems playbook

### Iterating over a list of items to install packages

* Create a list of packages  
* Let us create the following list of packages in base role.  
* Edit *group_vars/prod.yml* and put

```
  systems:
    packages:
      - ntp
      - tree
      - vim

```

* Also edit *roles/systems/tasks/main.yml* to iterate over this list of items and install packages

```
- name: install common systems packages
  package:  
    name:  "{{ item }}"
    state: installed
  with_items:
    - "{{ systems.packages }}"
```

apply

```
  ansible-playbook app.yml
```

### Iterating over a hash table/dictionary to create users

* This iteration can be done with using **with_dict** statement, let us see how.
* Edit *group_vars/all* file from the **parent directory** and define a dictionary of systems  users to be managed

```
users:
   admin:
     uid: 5001
     shell: /bin/bash
     home: /home/admin
     state: present
   dojo:
     state: absent
```

* **Update** the systems task to iterate over this dictionary

file:  *roles/systems/tasks/main.yml*

```
- name: create systems users
  user:
    name: "{{ item.key }}"
    uid:  "{{ item.value.uid  }}"
    shell: "{{ item.value.shell  }}"
    home: "{{ item.value.home    }}"
    state: "{{ item.value.state   }}"
  with_dict: "{{ users }}"
```


* Execute the *app* playbook to verify the output

```
ansible-playbook app.yml
```


### Troubleshooting : Setting defaults
The above playbook run fails as you have not defined all the fields for user **dojo** as part of the dictionary. You could either define it as part of vars, or better set defaults while invoking the vars, so that ansible automatically falls back to it.

file:  *roles/systems/tasks/main.yml*

```
- name: create systems users
   user:
     name: "{{ item.key }}"
     uid:  "{{ item.value.uid | default('none') }}"
     shell: "{{ item.value.shell | default('none') }}"
     home: "{{ item.value.home  | default('none')  }}"
     state: "{{ item.value.state  | default('none') }}"
   with_dict: "{{ users }}"

```

Apply and validate
```
ansible-playbook app.yml
```

## Refactoring apache playbook to add support for Ubuntu

Conditionals structures allow Ansible to choose an alternate path. Ansible does this by using *when* statements

When statement becomes helpful, when you will want to skip a particular step on a particular host


### Adding app3

Now lets update the inventory to add **app3**

file: environments/prod

```
[app]
app1
app2
app3 ansible_user=devops ansible_ssh_pass=codespaces

```

update package cache on app3
```
ansible app3 -m ping
ansible app3 -b -a "apt-get update"

```



### Selectively calling install tasks based on platform

Current configuration tasks that we have written is compatible with RedHat platform. It may fail on others. Lets selectively call it based on the platform family.

* Edit *roles/apache/tasks/main.yml*,

```
- import_tasks: config.yml
  when: ansible_os_family == 'RedHat'

```

* This will include *config.yml* only if the OS family is Redhat, otherwise it will skip the installation playbook


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
    package: httpd
    service:
      name: httpd
      state: started
```

file: roles/apache/vars/Debian.yml

```
---
  apache:
    package: apache2
    service:
      name: apache2
      state: started
```

Lets now selectively include those var files from tasks/main.yml .  Also selectively call configurations.

file: role/apache/tasks/main.yml

```
---
# tasks file for apache

  - include_vars: "{{ ansible_os_family }}.yml"

  - import_tasks: install.yml

  - import_tasks: service.yml

  - import_tasks: config.yml
    when: ansible_os_family == 'RedHat'

```


Update tasks and handlers to install and start the correct service

tasks/install.yml

```
---
- name: Install Apache...
  package:
    name: "{{ apache.package }}"
    state: latest

```

tasks/service.yml

```
---
- name: Starting Apache...
  service:
    name: "{{ apache.service.name }}"  
    state: "{{ apache.service.state }}"
```

handlers/main.yml

```
---
# handlers file for apache
- name: Restart apache service
  service:
    name: "{{ apache.service.name }}"  
    state: restarted
```


You also need to make sure  systems role to use **package** instead of yum module.  

apply playbook

```
ansible-playbook app.yml
```


### Selective execution by using tags

What all can be tagged,   
  * tasks
  * roles
  * plays
  * playbooks


Options to ansible-playbook  related to tags

```
--list-tags
--list-tasks
--tags=
--skip-tags=
```

Tag patterns

```
app
'all:!web'
'lb:db'
```

Lets tag the tasks

file: roles/apache/tasks/install.yml
```
---
- name: Install Apache...
  package:
    name: "{{ apache.package }}"
    state: latest

  tags:
    - apache
    - install
```

file: roles/apache/tasks/service.yml
```
---
- name: Starting Apache...
  service:
    name: "{{ apache.service.name }}"  
    state: "{{ apache.service.state }}"

  tags:
    - apache
    - service  
```


file: roles/apache/tasks/config.yml
```
---
  - name: copy apache config
    copy:
      src: httpd.conf
      dest: /etc/httpd.conf
      owner: root
      group: root
      mode: 0644
    notify: Restart apache service
    tags:
      - apache
      - config
```

Lets tag the roles and plays too,

file: app.yml
```
---
  - hosts: app
    become: true
    vars:
      fav:
        fruit: mango
    roles:
      - { role: apache, tags: www }
      - { role: php, tags: [ 'www', 'php' ] }
      - { role: frontend, tags: devopsdemo }
    tags:
      - frontend

```


and finally the playbooks,

file: site.yml

```
---
  # This is a sitewide playbook
  # filename: site.yml
  - import_playbook: lb.yml
    tags: lb

  - import_playbook: app.yml
    tags: app

  - import_playbook: db.yml
    tags: db

```

Now lets influence the tasks execution using tags,

```
ansible-playbook site.yml --list-tags
ansible-playbook site.yml --list-tasks
ansible-playbook app.yml  --tags=php
ansible-playbook app.yml  --tags='install:config'
ansible-playbook site.yml  --tags=frontend
ansible-playbook app.yml  --skip-tags=devopsdemo
ansible-playbook site.yml --tags='all:!db:!lb'


```

### Adding conditionals in Jinja2 templates

* Put the following content in *roles/frontend/templates/config.ini.j2*

```
[prefs]
{% if fav.color is defined %}
color  = {{ fav['color'] }}
{% endif %}

{% if fav.fruit is defined %}
fruit  = {{ fav['fruit'] }}
{% endif %}

{% if fav.car is defined %}
car    = {{ fav['car'] }}
{% endif %}

{% if fav.laptop is defined %}
laptop = {{ fav['laptop'] }}
{% endif %}

```


* In case if the var is not defined, it will not add the config to this file.

```
ansible-playbook site.yml
```

## Exercises

* Define dictionary of properties for a new user   in group_vars/prod. Observe if it gets created automatically.
* Define a hash/dictionary  of apache virtual hosts to be created,  and create a template which would iterate over that dictionary and create vhost configurations. You would additionally have to create a task to create the vhost configs.
* Learn about what else you could loop over, as well as how to do so by reading this document http://docs.ansible.com/ansible/playbooks_loops.html#id12
