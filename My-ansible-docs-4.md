
# Ansible Day 4

## get_url:


```python
#get_url module can be used to download a file from the internet.

get_url: 
  url: https://wordpress.org/wordpress-4.5.8.tar.gz
  dest: /tmp/wordpress.tar.gz
```

## Unarchive module


```python
#Unarchive module unzips/untars compressed files and put it in the destination directory of localhost.
# If the source and dest is remote server remote_src = yes parameter should also be given.

unarchive:
  src: /tmp/wordpress.tar.gz
  dest: /tmp/
  remote_src: yes
```


```python
#Below is a sample Wordpress conf file.

# vim wp-config.php.j2



<?php

define( 'DB_NAME', '{{mysql_database}}' );

define( 'DB_USER', '{{mysql_username}}' );

define( 'DB_PASSWORD', '{{mysql_password}}' );

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
#Playbook to download and install Wordpress in a remote Server.

# vim 05-01-wordpress-demo.yml

---
- hosts: amazon
  become: yes
  vars:
    mysql_username: wordpress
    mysql_password: wordpress123
    mysql_database: wordpress
  
  tasks:
    - name: 'Wordpress - Downloading.' 
      get_url:
        url: https://wordpress.org/wordpress-4.5.8.tar.gz
        dest: /tmp/wordpress.tar.gz
    
    - name: 'Wordpress - Extraction.'
      unarchive:
        src: /tmp/wordpress.tar.gz
        dest: /tmp/
        remote_src: yes
    
    - name: 'Wordpress - Copying To DocumentRoot.'
      shell: 'cp -r /tmp/wordpress/*  /var/www/html/'
    
    - name: 'Wordpress - Creating wp-config.php'
      template:
        src: wp-config.php.j2
        dest: /var/www/html/wp-config.php
```

# To remove LAMP Stack from remote machine


```python
ansible amazon -m shell -b  -a 'yum remove httpd php-* mariadb-* mariadb -y'
ansible amazon -m shell -b  -a 'rm -rf /var/www/html/* ; rm -rf /etc/httpd/ ; rm -rf /var/lib/mysql/* ; rm -rf /tmp/*'
```


```python
#To install Wordpress with LAMP Stack

# vim 05-02-wordpress-install.yml

---
- name: 'Wordpress Installation On Amazon Linux'
  hosts: all
  become: yes
  vars:
    domain: www.example.com
    mysql_root: mysqlroot123
    wordpress_user: wordpress
    wordpress_password: wordpress123
    wordpress_database: wordpress
      
  tasks:
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
        content: "<h1><center>info.html is working</center></h1>"
        dest: /var/www/html/{{domain}}/info.html      
          
    - name: 'LampStack - Creating info.php'
      copy:
        content: "<?php phpinfo(); ?>"
        dest: /var/www/html/{{domain}}/info.php
          
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
          
    - name: 'LampStack - Final Restarting'
      service:
        name: "{{item}}"
        state: restarted
      with_items:
        - mariadb
        - httpd
    
```
