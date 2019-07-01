
# Ansible Day 5

## Stat module


```python
#Stat module is used to find the info of a file/directory.
#It can also be used to check if a file/folder exists or not.

[ec2-user@ip-172-31-22-216 playbooks]$ cat stat-demo.yml 
---
- hosts: localhost
  tasks:
    - name: 'Stat of /etc/passwd'
      stat: path=/etc/passwd
      register: passwdStatus

    - debug: var=passwdStatus

    - name: 'Stat of /etc'
      stat: path=/etc
      register: etcStatus
    
    - debug: var=etcStatus

    - name: 'Stat of /unknown'
      stat: path=/unknown
      register: unknownStatus

    - debug: var=unknownStatus
      
[ec2-user@ip-172-31-22-216 playbooks]$ 
```


```python
#Executing the above playbook

[ec2-user@ip-172-31-22-216 playbooks]$ ansible-playbook stat-demo.yml 

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Stat of /etc/passwd] ***********************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "passwdStatus": {
        "changed": false, 
        "failed": false, 
        "stat": {
            "atime": 1560498803.6619854, 
            "attr_flags": "", 
            "attributes": [], 
            "block_size": 4096, 
            "blocks": 8, 
            "charset": "us-ascii", 
            "checksum": "df1ec03c04e409d94def5bfe689a937f3f725362", 
            "ctime": 1560498803.6619854, 
            "dev": 51713, 
            "device_type": 0, 
            "executable": false, 
            "exists": true, 
            "gid": 0, 
            "gr_name": "root", 
            "inode": 8711679, 
            "isblk": false, 
            "ischr": false, 
            "isdir": false, 
            "isfifo": false, 
            "isgid": false, 
            "islnk": false, 
            "isreg": true, 
            "issock": false, 
            "isuid": false, 
            "md5": "155dc66f6c88851c2784e875e7f60e1e", 
            "mimetype": "text/plain", 
            "mode": "0644", 
            "mtime": 1560498803.6619854, 
            "nlink": 1, 
            "path": "/etc/passwd", 
            "pw_name": "root", 
            "readable": true, 
            "rgrp": true, 
            "roth": true, 
            "rusr": true, 
            "size": 1238, 
            "uid": 0, 
            "version": "18446744073442412900", 
            "wgrp": false, 
            "woth": false, 
            "writeable": false, 
            "wusr": true, 
            "xgrp": false, 
            "xoth": false, 
            "xusr": false
        }
    }
}

TASK [Stat of /etc] ******************************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "etcStatus": {
        "changed": false, 
        "failed": false, 
        "stat": {
            "atime": 1557345885.972871, 
            "attr_flags": "", 
            "attributes": [], 
            "block_size": 4096, 
            "blocks": 24, 
            "charset": "binary", 
            "ctime": 1560499512.5882022, 
            "dev": 51713, 
            "device_type": 0, 
            "executable": true, 
            "exists": true, 
            "gid": 0, 
            "gr_name": "root", 
            "inode": 8409184, 
            "isblk": false, 
            "ischr": false, 
            "isdir": true, 
            "isfifo": false, 
            "isgid": false, 
            "islnk": false, 
            "isreg": false, 
            "issock": false, 
            "isuid": false, 
            "mimetype": "inode/directory", 
            "mode": "0755", 
            "mtime": 1560499512.5882022, 
            "nlink": 81, 
            "path": "/etc", 
            "pw_name": "root", 
            "readable": true, 
            "rgrp": true, 
            "roth": true, 
            "rusr": true, 
            "size": 8192, 
            "uid": 0, 
            "version": "18446744071717444343", 
            "wgrp": false, 
            "woth": false, 
            "writeable": false, 
            "wusr": true, 
            "xgrp": true, 
            "xoth": true, 
            "xusr": true
        }
    }
}

TASK [Stat of /unknown] **************************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "unknownStatus": {
        "changed": false, 
        "failed": false, 
        "stat": {
            "exists": false
        }
    }
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=7    changed=0    unreachable=0    failed=0   

```


```python
#To check if a file/folder exists in remote server

[ec2-user@ip-172-31-22-216 playbooks]$ cat stat-demo.yml 
---
- hosts: localhost
  tasks:
    - name: 'Stat of /etc/passwd'
      stat: path=/etc/passwd
      register: passwdStatus

    - debug: var=passwdStatus.stat.exists

    - name: 'Stat of /etc'
      stat: path=/etc
      register: etcStatus
    
    - debug: var=etcStatus.stat.exists

    - name: 'Stat of /unknown'
      stat: path=/unknown
      register: unknownStatus

    - debug: var=unknownStatus.stat.exists

```


```python
[ec2-user@ip-172-31-22-216 playbooks]$ ansible-playbook stat-demo.yml 

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Stat of /etc/passwd] ***********************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "passwdStatus.stat.exists": true
}

TASK [Stat of /etc] ******************************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "etcStatus.stat.exists": true
}

TASK [Stat of /unknown] **************************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "unknownStatus.stat.exists": false
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=7    changed=0    unreachable=0    failed=0  
```

## To delete the directory contents only


```python
#file module is used here to delete the entire directory and then create new directory with same name.

---
- hosts: localhost
  tasks: '
    - name: Deleting only /tmp/bin Contents'
      file: path=/tmp/bin state="{{item}}"
      with_items:
        - absent
        - directory
  
```

## To get the kernel status


```python
---
- hosts: localhost
  tasks:
    - name: 'Getting the kernel version'
      shell: 'uname -r'
      register: kernelStatus
    
    - debug: var=kernelStatus
```


```python
[ec2-user@ip-172-31-22-216 playbooks]$ ansible-playbook kernel-status.yml 

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Getting the kernel version] ****************************************************************************************************************************
changed: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "kernelStatus": {
        "changed": true, 
        "cmd": "uname -r", 
        "delta": "0:00:00.003425", 
        "end": "2019-06-20 03:53:00.860280", 
        "failed": false, 
        "rc": 0, 
        "start": "2019-06-20 03:53:00.856855", 
        "stderr": "", 
        "stderr_lines": [], 
        "stdout": "4.14.114-105.126.amzn2.x86_64", 
        "stdout_lines": [
            "4.14.114-105.126.amzn2.x86_64"
        ]
    }
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0   
```

# Ansible Roles


```python
#The concept of an Ansible role is simple; it is a group of variables, tasks, files, and handlers that are stored in a standardized file structure.
```

## Different parts of a playbook


```python
- tasks.
- Variables.
- Files.
- Jinja Templates.
- Handlers.
```


```python
#Default path of a role.

/etc/ansible/roles
```


```python
#Uncomment roles_path in ansible.cfg to enable ansible roles

[ec2-user@ip-172-31-22-216 wordpress-project]$ grep roles_path /etc/ansible/ansible.cfg 
roles_path    = /etc/ansible/roles
[ec2-user@ip-172-31-22-216 wordpress-project]$ 
```

## Creating an empty role


```python
[ec2-user@ip-172-31-22-216 playbooks]$ cd /etc/ansible/roles/
[ec2-user@ip-172-31-22-216 roles]$ ls



[ec2-user@ip-172-31-22-216 roles]$ sudo ansible-galaxy init lamp #Command to initialize a role named lamp
- lamp was created successfully # A directory named lamp will be created.
[ec2-user@ip-172-31-22-216 roles]$ ls
lamp
[ec2-user@ip-172-31-22-216 roles]$ cd lamp/
[ec2-user@ip-172-31-22-216 lamp]$ ls #default directories will be created and default files will also be automatically created.
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars

#Below is the tree of lamp directory.

[ec2-user@ip-172-31-22-216 lamp]$ tree
.
|-- defaults
|   `-- main.yml
|-- files
|-- handlers
|   `-- main.yml
|-- meta
|   `-- main.yml
|-- README.md
|-- tasks
|   `-- main.yml
|-- templates
|-- tests
|   |-- inventory
|   `-- test.yml
`-- vars
    `-- main.yml

8 directories, 8 files
```


```python
#sudo -i

#All the variables will be written inside the main.yml of vars directory

# vim lamp/vars/main.yml

---

domain: www.example.com
mysql_root: mysqlroot123
wordpress_user: wordpress
wordpress_password: wordpress123
wordpress_database: wordpress

```


```python
#Files which needs to Created should be placed inside the files directory.

#vim lamp/files/info.html

<h1><center>info.html is working</center></h1>


$vim lamp/files/info.php

<?php phpinfo(); ?>
```


```python
#Template files should be created under templates directory

#virtualhost.j2
#wp-config.php.j2
```


```python
#Tasks can be splitted and written seperate yml files.

#Here we create apache.yml for installing httpd and its requied files.

# vim lamp/task/apache.yml

---

- name: 'LampStack - Installation'
  yum:
    name:
      - httpd
      - php
      - php-mysql
      - mariadb-server
      - MySQL-python
    state: present
      
- name: 'LampStack - Restarting/Enabling'
  service:
    name: "{{item}}"
    state: restarted
    enabled: yes
  with_items:
    - mariadb
    - httpd
      
- name: 'LampStack - Creating Virtualhost'
  template:
    src: virtualhost.j2
    dest: /etc/httpd/conf.d/{{domain}}.conf
      
- name: 'LampStack - Creating Documentroot'
  file:
    path: /var/www/html/{{domain}}
    state: directory  
      
      
- name: 'LampStack - Creating info.html'
  copy:
    src: info.html
    dest: /var/www/html/{{domain}}/info.html      
      
- name: 'LampStack - Creating info.php'
  copy:
    src: info.php
    dest: /var/www/html/{{domain}}/info.php

```


```python
#Now we write another playbook to download and install Wordpress, configure mysql etc.

#So basically, different tasks can be splitted into different yml files for identification.

$vim lamp/tasks/mariadb.yml

---
- name: 'MariaDb - Resetting Root Password'
  mysql_user:
    login_user: root
    login_password: ''
    name: root
    host_all: yes
    password: "{{ mysql_root }}"
      
- name: 'MariaDb - Removing Anonymous Users'
  mysql_user:
    login_user: root
    login_password: "{{ mysql_root }}"
    name: ''
    host_all: yes
    state: absent
      
- name: 'MariaDb - Creating Wordpress Database'
  mysql_db:
    login_user: root
    login_password: "{{ mysql_root }}"
    db: "{{ wordpress_database }}"
    state: present
      
      
- name: 'MariaDb - Creating WordPress User and Privileges'
  mysql_user:
    login_user: root
    login_password: "{{ mysql_root }}"
    name: "{{ wordpress_user }}"
    host: localhost
    password: "{{wordpress_password}}"
    priv: "{{wordpress_database}}.*:ALL"
      
- name: 'Wordpress - Downloading'
  get_url:
    url: https://wordpress.org/wordpress-4.5.8.tar.gz
    dest: /tmp/wordpress.tar.gz
      
- name: 'Wordpress - Extraction'
  unarchive:
    src: /tmp/wordpress.tar.gz
    dest: /tmp/
    remote_src: yes

- name: 'Wordpress - Copying Wordpress To DocumentRoot'
  shell: 'cp -r /tmp/wordpress/*  /var/www/html/{{domain}}'
    
- name: 'Wordpress - Creating wp-config.php'
  template:
    src: wp-config.php.j2
    dest: /var/www/html/{{domain}}/wp-config.php
      
```


```python
$vim lamp/tasks/post-install.yml

---
- name: 'LampStack - Post Restarting'
  service:
    name: "{{item}}"
    state: restarted
  with_items:
    - mariadb
    - httpd
```


```python
:
$vim lamp/templates/wp-config.php.j2

<?php

define( 'DB_NAME', '{{wordpress_database}}' );
define( 'DB_USER', '{{wordpress_user}}' );
define( 'DB_PASSWORD', '{{wordpress_password}}' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );

define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

$table_prefix = 'wp_';

define( 'WP_DEBUG', false );


if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', dirname( __FILE__ ) . '/' );
}

require_once( ABSPATH . 'wp-settings.php' );
```


```python
#Now inside main.yml include_taks module is used to call different playbooks written in the tasks directory.


vim lamp/tasks/main.yml

---
- include_tasks: apache.yml
- include_tasks: mariadb.yml
- include_tasks: post-install.yml
```


```python
#Result

[ec2-user@ip-172-31-22-216 playbooks]$ ansible-playbook play-book-roles-demo.yml 

PLAY [amazon] ************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [172.31.18.26]

TASK [lamp : include_tasks] **********************************************************************************************************************************
included: /etc/ansible/roles/lamp/tasks/apache.yml for 172.31.18.26

TASK [lamp : LampStack - Installation] ***********************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : LampStack - Restarting/Enabling] ****************************************************************************************************************
changed: [172.31.18.26] => (item=mariadb)
changed: [172.31.18.26] => (item=httpd)

TASK [lamp : LampStack - Creating Virtualhost] ***************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : LampStack - Creating Documentroot] **************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : LampStack - Creating info.html] *****************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : LampStack - Creating info.php] ******************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : include_tasks] **********************************************************************************************************************************
included: /etc/ansible/roles/lamp/tasks/mariadb.yml for 172.31.18.26

TASK [lamp : MariaDb - Resetting Root Password] **************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : MariaDb - Removing Anonymous Users] *************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : MariaDb - Creating Wordpress Database] **********************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : MariaDb - Creating WordPress User and Privileges] ***********************************************************************************************
changed: [172.31.18.26]

TASK [lamp : Wordpress - Downloading] ************************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : Wordpress - Extraction] *************************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : Wordpress - Copying Wordpress To DocumentRoot] **************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : Wordpress - Creating wp-config.php] *************************************************************************************************************
changed: [172.31.18.26]

TASK [lamp : include_tasks] **********************************************************************************************************************************
included: /etc/ansible/roles/lamp/tasks/post-install.yml for 172.31.18.26

TASK [lamp : LampStack - Post Restarting] ********************************************************************************************************************
changed: [172.31.18.26] => (item=mariadb)
changed: [172.31.18.26] => (item=httpd)

PLAY RECAP ***************************************************************************************************************************************************
172.31.18.26               : ok=19   changed=15   unreachable=0    failed=0   
```


```python

```
