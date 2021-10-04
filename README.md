# Ansible Omeka Classic
[![CI](https://github.com/supertarto/ansible-omeka-classic/workflows/CI/badge.svg?event=push)](https://github.com/supertarto/ansible-omeka-classic/actions?query=workflow%3ACI)

Install and configure Omeka Classic with Ansible.


## Requirements
A web server, php and MariaDB. You can use supertarto.apache, supertarto.mariadb and supertarto.php

## Tested plateform
* Debian 10 (Buster)
* Debian 11 (Bulleyes)

## Role variables
Force omeka update
```yml
omeka_classic_force_update: false
```
Define wich version to download, the download link, where to unarchive and the destination.
The "unarchive content dest" is the dfaukt folder name. It will be rename with the content of "omeka classic content dest" for consitency
```yml
omeka_classic_release_version: "2.7.1"
omeka_classic_download_url: "https://github.com/omeka/Omeka/releases/download/v{{ omeka_classic_release_version }}/omeka-{{ omeka_classic_release_version }}.zip"
omeka_classic_unarchive_dir: "/var/www"
omeka_classic_unarchive_content_dest: "{{ omeka_classic_unarchive_dir }}/omeka-{{ omeka_classic_release_version }}"
omeka_classic_content_dest: "{{ omeka_classic_unarchive_dir }}/omeka-classic"
```
The directory where your local configurations will be backed up. Used only with omekaS_force_update set to True
```yml
omeka_classic_backup_directory: /"usr/local/omeka-classic-bck"
```
The web user and group
```yml
omeka_classic_web_owner: www-data
omeka_classic_web_group: www-data
```
Used in db.ini
```yml
omeka_classic_db_user: omeka
omeka_classic_db_password: omekapass
omeka_classic_db_name: omekadb
omeka_classic_db_host: localhost
```

## Examples
```yml
- name: Somehost
  hosts: all
  pre_tasks:
    - name: Update apt cache.
      apt:
        update_cache: true
        cache_valid_time: 600
      when: ansible_os_family == 'Debian'
      changed_when: false

  roles:
    - role: supertarto.apache
    - role: supertarto.mariadb
    - role: supertarto.php
    - role: supertarto.omeka_classic

  vars:
    php_packages:
      - php7.3
      - php7.3-mysql
      - php7.3-exif

    apache_create_vhosts: true
    apache_vhosts_filename: "omeka.conf"
    apache_vhost_config:
      - listen_ip: "*"
        listen_port: 80
        server_name: host1
        documentroot: "{{ omeka_classic_content_dest }}"
        serveradmin: admin@localhost
        custom_param: |
          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
          LogLevel warn
        directory:
          - path: "{{ omeka_classic_content_dest }}"
            config: |
              AllowOverride All
              Order deny,allow
              allow from all

        mariadb_use_dump_script: false
        mariadb_databases:
          - name: "{{ omeka_classic_db_name }}"

        mariadb_users:
          - name: "{{ omeka_classic_db_user }}"
            host: "{{ omeka_classic_db_host }}"
            password: "{{ omeka_classic_db_password }}"
            priv: "{{ omeka_classic_db_name }}.*:SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,CREATE TEMPORARY TABLES,LOCK TABLES"

    omeka_classic_release_version: "2.7.1"
    omeka_classic_db_user: omeka
    omeka_classic_db_password: omekapass
    omeka_classic_db_name: omekadb
    omeka_classic_db_host: localhost
```

## Installation
```
ansible-galaxy install supertarto.omeka_classic
```
## License
GPL V3.0
