# Setting up loadbalancer with a galaxy role

```
ansible-galaxy install geerlingguy.haproxy
```

lb.yml
```
- hosts: lb
  become: yes
  roles:
    - { role: geerlingguy.haproxy }

```

http://IPADDRESS/app


group_vars/prod.yml
```
haproxy_backend_httpchk: ''
haproxy_backend_servers:
  - name: app1
    address: 192.168.61.12:80
  - name: app2
    address: 192.168.61.13:80

```

http://IPADDRESS/app



## Exercises

### Nano Project: Setup MySQL Database
Now that you have configured load balancer along with the app servers, its time to setup database backend. And you could leverage ansible galaxy instead of writing all the code from scratch.  

Once you select and install role, create a playbook that would apply to **db** group from inventory.

In addition to setting up the mysql server, you would create
  * database: devopsdemo
  * user: devops
  * password: password-of-your-choice
  * root password: password-of-your-choice
and set grant permissions so that the **devops** user owns **devopsdemo** db. 
