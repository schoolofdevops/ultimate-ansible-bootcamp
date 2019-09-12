# Setting up loadbalancer with a galaxy role

With this lab, you are going to setup HAProxy, a lightweight open source load balancer using a role available in Ansible Galaxy.  

Browse to [Ansible Galaxy](https://galaxy.ansible.com/) to browse or search for the roles available and understand how Galaxy works. Then proceed with the steps below to setup haproxy.

Fetch a role from galaxy as,

```
cd chap8
ansible-galaxy install geerlingguy.haproxy
```

Create a new playbook in the top level dir with name **lb.yml**

```
- hosts: lb
  become: yes
  roles:
    - { role: geerlingguy.haproxy }

```

Also add the custom vars that you would like to override and customize as per your setup.

e.g.

`file: group_vars/prod.yml`

```
haproxy_backend_httpchk: ''
haproxy_backend_servers:
  - name: app1
    address: 192.168.61.12:80
  - name: app2
    address: 192.168.61.13:80

```

And apply the playbook

```
ansible-playbook lb.yml
```

To validate, load the haproxy configurations now for load balancer which is now running on port 80 on your host.

http://IPADDRESS/app

You could also verify by checking the haproxy configs as

```
ssh devops@lb

cat /etc/haproxy/haproxy.cfg
```
You should see the following block configured in haproxy.cfg

```
backend habackend
    mode http
    balance roundrobin
    option forwardfor
    cookie SERVERID insert indirect
    server app1 192.168.61.12:80 cookie app1 check
    server app2 192.168.61.13:80 cookie app2 check
```

Which confirms that haproxy has picked up the web servers and load balancing traffic across it.

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
