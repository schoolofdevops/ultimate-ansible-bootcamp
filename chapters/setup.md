# Settig up Learning Environment

## Option 1: Setting up a Vagrant based environment



### Install VirtualBox and Vagrant

| TOOL  | VERSION  |  LINK |
|---|---|---|
| VirtualBox  |   5.1.26  |   https://www.virtualbox.org/wiki/Downloads |
| Vagrant  | 1.9.7   | https://www.vagrantup.com/downloads.html   |  




### Importing a  VM Template

```
vagrant box list

vagrant box add ansible  codespace-ansible-ubuntu1604-9.box

vagrant box list

```

### Provisioning Vagrant Nodes

Clone repo if not already

```
git clone https://github.com/schoolofdevops/lab-setup.git


```

Launch environments with Vagrant

```
cd lab-setup/ansible/codespace

vagrant up

```

Login to node


**Terminal**

```
cd lab-setup/ansible/codespace
vagrant ssh
sudo su

```


You could also visit  ![http://192.168.46.10:8000](http://192.168.46.10:8000) to access the codespaces env.


## Option 2: Setting up a codespaces environment  with Docker


  * Clone the git repo
```
git clone https://github.com/codespaces-io/codespaces.git
```

### Start Codespaces IDE

After installing Docker-Engine and Docker-Compose, change directory into the corresponding tool you want to learn. For example, let us assume that you want to learn puppet. In that case,

```
cd cs-ansible
```

Then all you need to do is to run

```
docker-compose up -d
```

This single command will initialize your Codespaces IDE.

### Use Codespaces IDE

To use Codespaces IDE,

  * Open your browser.
  * Visit your machine's IP with port 8000. (Ex. http://192.168.0.60:8080)
  * You will be asked for your e-mail address. Enter it and you are good to go.

![Email](https://github.com/codespaces-io/codespaces/blob/master/images/email.jpg?raw=true)

  * Now you will be presented with the Codespaces IDE console.

![Landing](https://github.com/codespaces-io/codespaces/raw/master/images/landing.jpg?raw=true)
