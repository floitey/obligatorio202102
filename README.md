Rol simple para despliegue de stack LAMP.

1- @floitey configuración de repositorio floitey\obligatorio202102 e invitación a Luis y Fabricio como colaboradores
2- @floitey se agregan archivos "inventario" y "ansible.cfg" al repo Github+
3- @floitey se configuran todas las tasks necesarias para configurar el httpd tanto en entornos Redhat como Debian
[ansible@AnsibleServer tasks]$ cat install_httpd.yml
---
# These tasks install http and the php modules.

- name: Install apache for Redhat servers
  yum: name=httpd state=latest
  when: ansible_os_family == "RedHat"

- name: Install apache for Debian servers
  apt: name= httpd state=latest
  when: ansible_os_family == "Debian"


- name: open firewall for httpd in Redhat
  firewalld:
    service: http
    zone: public
    permanent: true
    state: reloaded
  when: ansible_os_family == "RedHat"
  notify: "reload firewalld"

- name: httpd service state
  service:
    name: httpd
    state: started
    enabled: yes
  when: ansible_os_family == "RedHat"
  notify: "start httpd"

- name: Configure SELinux to allow httpd to connect to remote database
  seboolean:
    name: httpd_can_network_connect_db
    state: true
    persistent: yes
  when: sestatus.rc != 0
  when: ansible_os_family == "RedHat"

- name: open firewall for http in Debian
  ufw:
    service: ufw
    zone: public
    permanent: true
    state: restarted
  when: ansible_os_family == "Debian"
  notify: "reload ufw"
  
  4- floitey se configuran los handlers involucrados para el servicio httpd.
[ansible@AnsibleServer handlers]$ cat main.yml
---
# Handler for the webtier: handlers are called by other plays.
# See http://docs.ansible.com/playbooks_intro.html for more information about handlers.


  name: start httpd
  service:
      name: httpd
      state: enabled
      enabled: yes

  name: reload firewall
  service:
      name: firewalld
      state: reloaded

  name: reload ufw
  service:
      name: ufw
      state: reloaded
 5- floitey se configuran tasks para MariaDB
 [ansible@AnsibleServer tasks]$ cat main.yml
---
# This playbook will install mysql and create db user and give permissions.

- name: install php on centos
  yum:
    name: php
    state: latest
    when: ansible_os_family == "RedHat"

- name: install mariadb server on centos
  yum:
    name: mariadb-server
    state: present
    when: ansible_os_family == "RedHat"

- name: install php on centos
    apt:
      name: php
      state: latest
      when: ansible_os_family == "Debian"

- name: install mariadb server on centos
    apt:
      name: mariadb-server
      state: present
      when: ansible_os_family == "RedHat"


- name: Create Mysql configuration file
  template:
    src: my.cnf.j2
    dest: /etc/my.cnf
  notify: restart mariadb

- name: Start Mysql Service
  service:
    name: mariadb
    state: started
    enabled: yes
    notify: restart mariadb

- name: open firewall for mariadb in Redhat
  firewalld:
     service: mariadb
     zone: public
     permanent: true
     state: reloaded
 when: ansible_os_family == "RedHat"
 notify: "reload firewalld"

- name: open firewall for mariadb in Redhat
    firewalld:
       service: mariadb
       zone: public
       permanent: true
       state: reloaded
       when: ansible_os_family == "Debian"
       notify: "reload ufw"

6- floitey se configuran handlers para mariaDB
[ansible@AnsibleServer tasks]$ cat main.yml
---
# This playbook will install mysql and create db user and give permissions.

- name: install php on centos
  yum:
    name: php
    state: latest
    when: ansible_os_family == "RedHat"

- name: install mariadb server on centos
  yum:
    name: mariadb-server
    state: present
    when: ansible_os_family == "RedHat"

- name: install php on centos
    apt:
      name: php
      state: latest
      when: ansible_os_family == "Debian"

- name: install mariadb server on centos
    apt:
      name: mariadb-server
      state: present
      when: ansible_os_family == "RedHat"


- name: Create Mysql configuration file
  template:
    src: my.cnf.j2
    dest: /etc/my.cnf
  notify: restart mariadb

- name: Start Mysql Service
  service:
    name: mariadb
    state: started
    enabled: yes
    notify: restart mariadb

- name: open firewall for mariadb in Redhat
  firewalld:
     service: mariadb
     zone: public
     permanent: true
     state: reloaded
 when: ansible_os_family == "RedHat"
 notify: "reload firewalld"

- name: open firewall for mariadb in Redhat
    firewalld:
       service: mariadb
       zone: public
       permanent: true
       state: reloaded
       when: ansible_os_family == "Debian"
       notify: "reload ufw"
       
7- floitey se configuran archivos ansible.cfg y hosts para relacionar los roles con la correcta sintaxis para ejecución
Ansible.cfg
inventory      = ./hosts

[ansible@AnsibleServer lamp_simple]$ cat hosts
[ubuntu]
UbuntuClient ansible_host=192.168.56.110

[centos]
CentosClient ansible_host=192.168.56.109

[linux:children]
ubuntu
centos

8- floitey se ejecuta pull final para consolidar los cambios y bajar un versionado actualizado en archivo .zip

 
