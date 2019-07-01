
# Ansible Day 3

## Template module demo to modify wp.conf


```python
#wp-conf.j2 is the sameple template file. Variables mentioned in the yml file will be substituted here.

[ec2-user@ip-172-31-22-216 ~]$ cat wp-config.j2 
define( 'DB_NAME', '{{mysql_db_name}}' );

define( 'DB_USER', '{{mysql_username}}' );

define( 'DB_PASSWORD', '{{mysql_password}}' );


#The playbook to create a wp-config.php in the /tmp and write the config values.

[ec2-user@ip-172-31-22-216 ~]$ cat wpconfig-change.yml
---
- hosts: localhost
  vars: 
    mysql_username: uuseradmin
    mysql_password: password123
    mysql_db_name: testdb
   
  tasks:
    - name: substitute wpconf with exact values
      template:
        src: wp-config.j2
        dest: /tmp/wp-config.php
            
            
#Executing playbook.
    
[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook wpconfig-change.yml 

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [substitute wpconf with exact values] *******************************************************************************************************************
changed: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0   
    
#checking the /tmp/wp-config.php file.
#The variables are replacd with exact values.

[ec2-user@ip-172-31-22-216 ~]$ cat /tmp/wp-config.php 
define( 'DB_NAME', 'testdb' );

define( 'DB_USER', 'uuseradmin' );

define( 'DB_PASSWORD', 'password123' );
```

# Installing and configuring Mysql(Mariadb)


```python
[ec2-user@ip-172-31-22-216 ~]$ cat mariadb_install.yml 
---
- name: "MariaDB Installation and Configuration for Wordpress"
  hosts: amazon
  become: yes
#Variable declaration

  vars:
    mysql_username: mysqluser
    mysql_password: password123
    mysql_db_name: test_mysqldb
    mysql_root_passwd: root123


  tasks:
    - name: MariaDB Server Installation
      yum:
        name:
          - MySQL-python
          - mariadb-server
        state: present

    - name: "Restarting Mysql"
      service:
        name: mariadb
        state: restarted

    - name: "MariaDB Server Resetting root password"
      mysql_user:
        login_user: root
        login_password: ''
        name: root
        host_all: yes
        password: "{{mysql_root_passwd}}"

    - name: "Removing anonymous Users from Mysql DB"
      mysql_user:
        login_user: root
        login_password: "{{mysql_root_passwd}}"
        name: ''
        host_all: yes
        state: absent

    - name: "DB Creation"
      mysql_db:
        login_user: root
        login_password: "{{mysql_root_passwd}}"
        db: "{{mysql_db_name}}"
        state: present
    
    - name: "Creating User and granting privilege"
      mysql_user:
        login_user: root
        login_password: "{{mysql_root_passwd}}"
        name: "{{mysql_username}}"
        host: localhost
        password: "{{mysql_password}}"
        priv: "{{mysql_db_name}}.*:ALL"
[ec2-user@ip-172-31-22-216 ~]$ 
```

### Ignoring errors


```python
#To ignore errors while executing a task, ignore_errors: yes needs to be used inside a task.

#Example: In the above playbook, if a new user needs to be created, adding a line in for resetting mysql root password will help.

    - name: "MariaDB Server Resetting root password"
      ignore_errors: yes

        

PLAY [MariaDB Installation and Configuration for Wordpress] **************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [172.31.18.26]

TASK [MariaDB Server Installation] ***************************************************************************************************************************
ok: [172.31.18.26]

TASK [Restarting Mysql] **************************************************************************************************************************************
changed: [172.31.18.26]

TASK [MariaDB Server Resetting root password] ****************************************************************************************************************
fatal: [172.31.18.26]: FAILED! => {"changed": false, "failed": true, "msg": "unable to connect to database, check login_user and login_password are correct or /root/.my.cnf has the credentials. Exception message: (1045, \"Access denied for user 'root'@'localhost' (using password: NO)\")"}
...ignoring

TASK [Removing anonymous Users from Mysql DB] ****************************************************************************************************************
ok: [172.31.18.26]

TASK [DB Creation] *******************************************************************************************************************************************
changed: [172.31.18.26]

TASK [Creating User and granting privilege] ******************************************************************************************************************
changed: [172.31.18.26]

PLAY RECAP ***************************************************************************************************************************************************
172.31.18.26               : ok=7    changed=3    unreachable=0    failed=0 
```

### Reading file from external file


```python
# Create a new file with all variables

[ec2-user@ip-172-31-22-216 ~]$ cat mysql-vars.yml 
---
mysql_username: mysqluser
mysql_password: password123
mysql_db_name: test_mysqldb
mysql_root_passwd: root123

#In the playbook mention the filename under vars_files: module


  vars_files:
   - mysql-vars.yml

```

## Encrypting files using Ansible Vault


```python
#We can encrypt files with important data using ansible-vault

ec2-user@ip-172-31-22-216 ~]$ ansible-vault encrypt mysql-vars.yml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
[ec2-user@ip-172-31-22-216 ~]$ 
[ec2-user@ip-172-31-22-216 ~]$ 
[ec2-user@ip-172-31-22-216 ~]$ cat mysql-vars.yml 
$ANSIBLE_VAULT;1.1;AES256
62313362623265636564623930306464313661666635616130666565363362653566643831396134
3865343466376565393730336535373232653661636566650a336565383232623966376664383534
38333436336132323133303163393332303965336563366362363637376631623437356437316539
3261353035383239660a323032326466333435613230613362323331646564333462383538623337
66663963373435363931656463373239313363613163313662363430343662353737323230363763
62336537346333663034366462363730363832633134316335366638653335623934666131313830
36316664336235343739333561383461373565653032356232346661306130656166306230303232
63393531643464376337316135626139346439396461323435303561666161363033376666666466
36303039393839653834393236633738383730636133383531393163333634643264313331333061
3933623138643863373136376136646538303539616333383165

```


```python
#--ask-vault-pass should be used with playbook command to decrypt the file while executing playbook, else it will return error.

[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook mariadb_install.yml 
ERROR! Attempting to decrypt but no vault secrets found


#--ask-vault-pass should be used with playbook command and password should entered.

[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook --ask-vault-pass mariadb_install.yml 
Vault password: 

PLAY [MariaDB Installation and Configuration for Wordpress] **************************************************************************************************

```


```python
#To decrypt the encrypted file below command shoudl be used.


[ec2-user@ip-172-31-22-216 ~]$ ansible-vault decrypt mysql-vars.yml 
Vault password: 
Decryption successful
[ec2-user@ip-172-31-22-216 ~]$ cat mysql-vars.yml 
---
mysql_username: mysqluser
mysql_password: password123
mysql_db_name: test_mysqldb
mysql_root_passwd: root123
[ec2-user@ip-172-31-22-216 ~]$ 
```

# Ansible Handlers


```python
#Playbook to install LAMP using handlers.
#A notify is set after a task which will only be executed if a task return value is changed.
#Note: The handler task will only be executed after all the tasks are completed.

[ec2-user@ip-172-31-22-216 ~]$ cat handler.yml 
---
- hosts: amazon
  become: yes
  tasks:
    - name: 'Installing Httpd'
      yum:
        name: httpd
        state: present
      notify: restarting-httpd

    - name: 'Installing php'
      yum:
        name:
          - php
          - php-mysql
          - php-mbstring
        state: present
      notify: 
         - dummy-task1
         - dummy-task2
  
    - name: 'Creating phpsite'
      copy:
        content: '<h1>sample 1 content</h1>'
        dest: /var/www/html/index.html
      notify: restarting-httpd

  handlers:

    - name: 'restarting-httpd'
      service:
        name: httpd
        state: restarted

    - name: 'dummy-task1'
      debug:
        msg: 'executing dummy task1'

    - name: 'dummy-task2'
      debug:
        msg: 'executing dummy task2'
[ec2-user@ip-172-31-22-216 ~]$ 

[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook handler.yml 

PLAY [amazon] ************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [172.31.18.26]

TASK [Installing Httpd] **************************************************************************************************************************************
ok: [172.31.18.26]

TASK [Installing php] ****************************************************************************************************************************************
changed: [172.31.18.26]

TASK [Creating phpsite] **************************************************************************************************************************************
changed: [172.31.18.26]

RUNNING HANDLER [restarting-httpd] ***************************************************************************************************************************
changed: [172.31.18.26]

RUNNING HANDLER [dummy-task1] ********************************************************************************************************************************
ok: [172.31.18.26] => {
    "msg": "executing dummy task1"
}

RUNNING HANDLER [dummy-task2] ********************************************************************************************************************************
ok: [172.31.18.26] => {
    "msg": "executing dummy task2"
}

PLAY RECAP ***************************************************************************************************************************************************
172.31.18.26               : ok=7    changed=3    unreachable=0    failed=0   


    

```
