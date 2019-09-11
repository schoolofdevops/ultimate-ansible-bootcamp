## Why to use Vault
   * To maintain sensitive data e.g. passwords/creds, keys etc.
   * Version control encrypted files instead of plain text
   * ansible-vault utility

## How ?
  * Used AES Cipher
  * Symmetric Key

## What can be encrypted ?
   * Structured data (yaml, json)
   * Var files
     * group_vars/hostvars
     * include_vars or  var_files
     * var files passed at command line with "-e @file"
   * Tasks (however not very common)
   * Arbitory Files
   * Strings (newly added)

## What can not be encrypted ?
  * Templates


## How to encrypt/decrypt
  * Using --ask-vault-pass
  * Using --vault-password-file


## ansible-vault Operations
  * encrypt
  * decrypt
  * create
  * rekey
  * edit

### Examples

Running Playbooks with Vault

```
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt
```

Automating Rekeying Process
```
--new-vault-password-file=NEW_VAULT_PASSWORD_FILE

```
                       new vault password file for rekey




## Lab : Encrypting and decrypting with single key

```
mkdir vault
```

file: vault/api_keys
```
USER: devops
API_KEY: GUIFEHR3485y384H78435YYF89GUW03RIUWFHI
TOKEN: 8549JBHBHUPIYHSL602JHU
```

Encrypting file

```
cd vault
ansible-vault encrypt api_keys
cat api_keys
ansible-vault view api_keys

```

write a playbook to use encrypted file

file: test_vault.yml
```
---
  - name: testing ansible vault
    hosts: 'local:app'
    become: true
    tasks:
      - name: copy a file containing api keys
        copy:
          src: vault/api_keys
          dest: /root/.api_keys
          owner: root
          group: root
          mode: 0400

```

apply
```
ansible-playbook test_vault.yml
ansible-playbook test_vault.yml --ask-vault-pass

```

Using a password file

file ~/.vault
```
password1
```

profile passowrd file
```
ansible-playbook test_vault.yml --vault-password-file ~/.vault

```


## New Vault: Multiple vault ids and encrypting strings

create vault password file for vault id **prod**

file ~/.vault_prod
```
prodpassword
```


Create files to encrypt  

file: creds
```
mysql_root_password: password
```


create copies of it
```
cp creds staging
cp creds prod
```

encrypt
```
ansible-vault encrypt creds
ansible-vault encrypt staging --vault-id staging@prompt
ansible-vault encrypt prod --vault-id prod@~/.vault_prod
```

decrypt all

```
ansible-vault decrypt --vault-id staging@prompt staging --vault-id prod@~/.vault_prod --vault-id @prompt creds
```


creating individual vaules
```
ansible-vault encrypt_string --vault-id prod@~/.vault_prod 'password' --name 'mysql_root_password'
```
