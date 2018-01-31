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
