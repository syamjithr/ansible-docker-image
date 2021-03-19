# ansible-docker-image
## This ansible playbook used to creating a docker image and push that image to the dockerhub registry.
vim docker-image.yaml
```
---
- name: Installing docker on ububtu
  hosts: container
  become: true
  tasks:
   - name: Install aptitude using apt
     apt: name=aptitude state=latest update_cache=yes force_apt_get=yes

   - name: Install required system packages
     apt: name=apt-transport-https,ca-certificates,curl,software-properties-common,python3-pip,virtualenv,python3-setuptools state=latest update_cache=yes

   - name: Add Docker GPG apt Key
     apt_key:
       url: https://download.docker.com/linux/ubuntu/gpg
       state: present

   - name: Add Docker Repository
     apt_repository:
       repo: deb https://download.docker.com/linux/ubuntu bionic stable
       state: present

   - name: Update apt and install docker-ce
     apt: update_cache=yes name=docker-ce state=latest

   - name: Install Docker Module for Python
     pip:
       name: docker

   - name: create build directory
     file:
       path: /root/demo-dockerfile
       state: directory
       owner: root
       group: root
       mode: '0755'

   - name: copy Dockerfile
     copy:
       src: ./Dockerfile
       dest: /root/demo-dockerfile/Dockerfile
       owner: root
       group: root
       mode: '0644'

   - name: build container image
     docker_image:
       build:
         path: /root/demo-dockerfile
       name: democontainer
       tag: v1
       source: build
       state: present

   - name: Log into DockerHub
     docker_login:
       username: dockerhub_username
       password: dockerhub_password

   - name: Push Docker image to Registry
     docker_image:
       name: democontainer
       build:
         path: /root/demo-dockerfile
         pull: true
       state: present
       tag: "v1"
       force_tag: yes
       repository: syamjith/democontainer:v1
       push: yes
       source: build

```
##### Dockerfile
vim ./Dockerfile
```
FROM httpd:2.2
EXPOSE 8080
CMD nc -l -p 8080
```
##### execution 
```
ansible-playbook -i inventory.txt docker-image.yaml 
```
