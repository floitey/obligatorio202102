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

- name: open firewall for httpd in Debian
  ufw:
    service: ufw allow httpd
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
