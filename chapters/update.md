# Updating Application

## Writing database schema update playbook

The following url has the links to the database dumps  

[Database SQL Dumps]( https://github.com/devopsdemoapps/devops-demo-app/tree/master/data)


Lets write tasks to update schema for database

file: db.yml
```
---
  - name: playbook to configure db servers
    hosts: db
    become: yes
    roles:
      - { role: geerlingguy.mysql }
    tasks:
      - name: download database schema
        get_url:
          url: https://raw.githubusercontent.com/devopsdemoapps/devops-demo-app/master/data/devops-demo-1.0.sql
          dest: /tmp/devops-demo-1.0.sql
          mode: 0444
        tags: schema

      - name: Schema Migrate
        mysql_db:
          name: "{{ dbconn.db }}"
          login_host: "127.0.0.1"
          login_password: "{{ mysql_root_password }}"
          login_user: "root"
          state: import
          target: /tmp/devops-demo-1.0.sql
        tags: schema

```
where,
  * we have added task to download the schema file from a remote uri
  * load the schema using **import** state

Its important to use **1.0** as the schema version first time you run as it needs the initial tables.

Run it for the first time,

```
ansible-playbook db.yml  --vault-id prod@~/.vault_prod --tags=schema
```

Once the initial schema apply is done, update the tasks to use **app.version** for schema update as well

```
---
  - name: playbook to configure db servers
    hosts: db
    become: yes
    roles:
      - { role: geerlingguy.mysql }
    tasks:
      - name: download database schema
        get_url:
          url: https://raw.githubusercontent.com/devopsdemoapps/devops-demo-app/master/data/devops-demo-{{ app.version }}.sql
          dest: /tmp/devops-demo-{{ app.version }}.sql
          mode: 0444
        tags: schema

      - name: Schema Migrate
        mysql_db:
          name: "{{ dbconn.db }}"
          login_host: "127.0.0.1"
          login_password: "{{ mysql_root_password }}"
          login_user: "root"
          state: import
          target: /tmp/devops-demo-{{ app.version }}.sql
        tags: schema

```



## Deployment Playbook

Strategy:
  * Rolling updates/zero downtime
  * Batch size = 1

Deployment strategy and sequence,
  * Update database schema
  * For each web server update
    * Disable load balancer traffic
    * Deploy new version of the application
    * Wait for the app to be available
    * Enable traffic from load balancer


### Pre tasks


file: deployment.yml
```
- hosts: app
  become: yes
  vars:
    app:
      version: 1.6
  serial: 1

  pre_tasks:
    - name: take the app server out
      haproxy:
        host: '{{ inventory_hostname }}'
        state: disabled
      delegate_to: lb

  roles:
     - frontend

  post_tasks:
    - name: Wait 300 seconds for port 80 to be available
      wait_for:
        port: 80
        host: '{{ inventory_hostname }}'
        delay: 5
        timeout: 300
      connection: local
    - name: add server to loadbalancer
      haproxy:
        host: '{{ inventory_hostname }}'
        state: enabled
      delegate_to: lb


```

Deploy a new version with

```
ansible-playbook deployment.yml

```
