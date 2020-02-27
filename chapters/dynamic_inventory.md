## Using dynamic inventory with ec2

Download the script to setup dynamic inventory

```
wget -c  https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.py
```

create **ec2.ini** with crdentials to connect to aws.

**Best practice**: its recommended you create a read only user and use the iam keys for the same with ansible. For dynamic inventory, Ansible does need any additional access to make changes. Its a recommended security practice.

sample ec2.ini
```
[profile staging]
export AWS_ACCESS_KEY_ID='AKJKHKSDHFSJHJD73NEQ2Q'
export AWS_SECRET_ACCESS_KEY='aUy56Ksmw2bD/Aepmsdge3KsasnMSJIHls209NZpTc7'
```

its also recommeded you store this file somewhere securely with least privileges

e.g.
```
mv ec2.ini ~/.ec2.ini
chmod 400 ~/.ec2.ini

```

Set the path to ini file

export EC2_INI_PATH=~/.ec2.ini


Create a local ansible configuration

```
[defaults]
remote_user = ubuntu
inventory   = ec2.py
retry_files_save_path = /tmp
host_key_checking = False
log_path=ansible.log
```

Now test the dynamic inventory script

examples (update as per your profile and instance configs)

```
./ec2.py --list

./ec2.py --profile demo

/ec2.py --host 38.105.83.147
```



This should connect to aws, fetch information and display groups dynamically fetched.

You should now be ready to connect to the ec2 servers

e.g.
```
ansible all --list-hosts
ansible ec2 --list-hosts
ansible ec2 -m ping
ansible tag_env_demo -m ping
```

## Writing your own dynmaic inventory

**References:**

http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html 

http://docs.ansible.com/ansible/latest/dev_guide/developing_inventory.html

https://www.jeffgeerling.com/blog/creating-custom-dynamic-inventories-ansible
--list
--host <hostname>
