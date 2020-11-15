+++
title = "Provision AWS Lightsail with Ansible"
slug = ""
categories = [
  "tech",
]
tags = [
  "aws",
  "ansible"
]
date = "2016-12-21T22:18:43+08:00"
draft = false

+++

Amazon has announced a new cloud service **Lightsail** recently aiming at DigitalOcean, with exact same price and same spec of node.  As a heavy DigitalOcean user, I am more than happy to try the alternative provided by AWS. Creating the first instance is not smooth, I got successfully created the first instance by AWS SDK after 3 weeks in and out mails with the support team. 

TL;DR

This post is a quick guide on provisioning the instance by Ansible.  Before that ,  some outlines:

- The default login account for Lightsail is pretty much depended on the image, while in DigitalOcean it is **root**.
- Lightsail is using key pair (.pem) for ssh login, you can either use default key pair or create a new pair. After successfully log into the instance, I added a pub ssh id by my preference.
- Everything in docker, as well as ansible

## Ansible playbook

provision.yml
```yml
---
- hosts: cloud 
  gather_facts: False
  tasks:
    - name: Update local known_hosts
      local_action: shell ssh-keyscan -H {{ hostvars[item].ansible_host }} >> ~/.ssh/known_hosts
      with_items: "{{ groups.cloud }}"

    - name: Install aptitude
      raw: test ! -e /usr/bin/aptitude && sudo apt-get install -qq aptitude || true

    - name: Install python 2.7
      raw: test ! -e /usr/bin/python && (sudo apt-get update -qq && sudo apt-get install -qq python2.7) || true

    - name: Install letsencrypt 
      raw: test ! -e /usr/bin/letsencrypt && (sudo apt-get update -qq && sudo apt-get install -qq letsencrypt) || true

- hosts: cloud 
  roles:
    - update-apt
    - user
    - swap

- hosts: cloud 
  roles:
    # https://github.com/angstwad/docker.ubuntu
    - role: angstwad.docker_ubuntu
      become: yes
      kernel_pkg_state: present
```

provision.ini
```ini
[hosts]
[YOUR_LIGHTSAIL_INSTANCE_NAME] ansible_host=[YOUR_IP] private_ip=[YOUR_PRIVATE_IP]

[cloud:children]
hosts

[cloud:vars]
ansible_connection=ssh
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python2.7
```


Looking at `provision.yml` , the first 3 common tasks are pretty straight forward:

* Adding remote instance IP to your local known_hosts
* Install Python
* Install letsencrypt for HTTPS

Followed by 3 common roles:

* Update and upgrade apt
* Create application user in sudo group
* Specify swap file (optional)

I would like to pick user role as example

ansible/roles/user/tasks/main.yml
```yml
---
- name: Ensure user exists
  become: yes
  user:
    name: appuser
    state: present
    shell: /bin/bash
    append: yes 
    groups: sudo

- name: Ensure authorized key exists
  become: yes
  authorized_key: user=appuser key="{{ lookup('file', '~/.ssh/id_rsa.pub') }}"

- name: Copy sudoers
  become: yes
  copy: src=./sudoers dest=/etc/sudoers
```

This role will create a user `appuser` in sudo group for application deploy.  So that we don’t need to use the default user `ubuntu` every time. Make sure you have added the id pub key into the instance’s `authorized_key` file

The last role is installing docker into the instance, after that you can use any docker command like `sudo docker` or `sudo docker-compose` in the instance and it’s armed with docker engine now.

OK now we have a provisioned instance and keep lego it with any application you write. I will suggest to use Ansible docker service as well to depoly your application based on docker image.





