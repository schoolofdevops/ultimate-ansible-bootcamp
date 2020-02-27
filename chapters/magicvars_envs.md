# Magic Variables for Service Discovery, Multiple Environments

file: ansible.cfg
```
fact_caching = yaml
fact_caching_connection = /tmp/facts

```


file: roles/geerlingguy.haproxy/templates/haproxy.cfg.j2

```
{% for host in groups['app']  %}
    server {{ hostvars[host]['ansible_hostname'] }} {{ hostvars[host]['ansible_eth0']['ipv4']['address'] }}:80 cookie {{ hostvars[host]['ansible_hostname'] }} check
{% endfor %}

```


## Create staging env

production/staging

```
[app]
app2

[db]
app2

[staging:children]
app
db

```


file: group_vars/all.yml

```
---
  users:
    admin:
      uid: 5001
      shell: /bin/bash
      home: /home/admin
      state: present
    dojo:
      state: absent

  systems:
    packages:
      - ntp
      - tree
      - vim

```

file: group_vars/staging.yml
```
---

  app:
    version: 1.5
    env: staging

  fav:
    color: blue
    fruit: watermelon

  dbconn:
    host: 127.0.0.1
    user: devops
    pass: dfkl8d6msoYc0
    db: devopsdemo

  mysql_root_password: dfdvdHkst0ks72sY
  mysql_databases:
    - name: devopsdemo
      encoding: latin1
      collation: latin1_general_ci
  mysql_users:
    - name: devops
      host: "%"
      password: dfkl8d6msoYc0
      priv: "devopsdemo.*:ALL"

```




## Cleaning up

cleanup.yml
```
---
  - name: cleanup database server
    hosts: db
    become: true
    tasks:
      - name: stop mysql service
        service:
          name: mysqld
          state: stopped

      - name: uninstall mysql related packages  
        package:
          name: "{{ item }}"
          state: absent
        with_items:
          - mysql-server
          - mysql

```
