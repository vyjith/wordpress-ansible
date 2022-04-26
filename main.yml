---
- name: "Apache installation and site hosting"
  become: true
  hosts: amazon
  vars:
    apache:
      - httpd
      - mariadb-server
      - mariadb
      - MySQL-python
    http_port: 80
    http_host: wordpress.vyjithks.tk
    http_conf: wordpress.vyjithks.tk.conf
    mysql_root_password: "{{ (lookup('aws_secret', 'keyofmysql', region='ap-south-1') | from_json).mysql_root_password }}"
    mysql_db: wordpress
    mysql_user: wordpress
    mysql_password: "{{ (lookup('aws_secret', 'keyofmysql', region='ap-south-1') | from_json).mysql_password }}"

  tasks:

    - name: "Installing server packages."
      yum:
        name: "{{ apache }}"
        state: present
      tags: apache

    - name: "Enable php74 and epel respositories"
      shell: amazon-linux-extras install php7.4 -y

    - name: "Starting the services"
      service:
        name: "{{ item }}"
        state: restarted
        enabled: yes
      with_items:
        - mariadb

    - name: "Copying the website virtual host file to the remote machine"
      template:
        src: httpd.conf.tmpl
        dest: "/etc/httpd/conf.d/{{ http_conf }}"
        owner: root
        group: root

    - name: "Creating apache Document root"
      file:
        path: "/var/www/{{ http_host }}"
        state: directory
        owner: "apache"
        group: "apache"

    - name: "Set mysql root password"
      ignore_errors: yes
      mysql_user:
        login_host: 'localhost'
        login_user: 'root'
        login_password: ''
        name: 'root'
        password: '{{ mysql_root_password }}'
        state: present

    - name: "Creating database for wordpress"
      mysql_db:
        name: "{{ mysql_db }}"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: "Create mysql user for wordpress"
      mysql_user:
        name: "{{ mysql_user }}"
        password: "{{ mysql_password }}"
        priv: "{{ mysql_db }}.*:ALL"
        state: present
        login_user: root
        login_password: "{{ mysql_root_password }}"

    - name: "Download and upack latest Wordpress"
      unarchive:
        src: https://wordpress.org/latest.tar.gz
        dest: "/var/www/{{ http_host }}"
        remote_src: yes
        creates: "/var/www/{{ http_host }}/wordpress"

    - name: "Set ownership"
      file:
        path: "/var/www/{{ http_host }}"
        state: directory
        recurse: yes
        owner: apache
        group: apache

    - name: "Copying the website files"
      template:
        src: wordpress.conf.tmpl
        dest: /var/www/{{ http_host }}/wordpress/wp-config.php

    - name: "Restarting httpd service"
      service:
        name: httpd
        state: restarted
