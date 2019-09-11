# Creating a custiom Modules

## Writing module with bash

```
mkdir library
touch library/mymodule
```

file: library/mymodule
```
#!/bin/bash

display="This is a simple bash module.."

echo -e "{\"message\":\""$display"\"}"

```


file: custom_modules.yml

```
---
- hosts: local
  sudo: yes
  tasks:
   - name: test custom module
     mymodule:
     register: uptime

   - debug: var=uptime
```


Test
```
ansible-playbook custom_module.yml
```




## Accepting module options


file: custom_modules.yml

```
- name: check version
  printversion:
    app: java
    appv: 3.4
  register: printversion

- debug: var=printversion
```

file: library/printversion


```
#!/bin/bash
#
# This script accepts two inputs
# 1. app
# 2. appv
# and prints it as a message

changed="false"

source $1

display="Received app  $app with version as $appv"



if [ "$app" == "python" ]; then
  changed="true"
fi

printf '{"changed": %s, "msg": "%s"}' "$changed" "$display"

exit 0

```

Test
```
ansible-playbook custom_module.yml
```


### Trying out sample python module


```
cd library
wget -c https://gist.githubusercontent.com/initcron/88049b4fc3cbf4c53d17405efdd3a720/raw/fd2a4bccbe8fa895e3f6a6b517ec74abd1844df5/my_new_test_module
```

file: custom_modules.yml

```
- name: run the new module
  my_new_test_module:
    name: 'hello'
    new: true
  register: testout

- name: dump test output
  debug:
    msg: '{{ testout }}'

```


Test
```
ansible-playbook custom_module.yml
```
