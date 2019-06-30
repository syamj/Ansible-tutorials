
## Facts Gathering Using Ansible


```python
#Ansible uses setup module to gather facts about the remote VM.
# Before a playbook/module is run at remote VM, ansible automatically run the setup module and grab details about the remote VM.

[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -m setup | grep "ansible_distribution"
        "ansible_distribution": "Amazon", 
        "ansible_distribution_file_parsed": true, 
        "ansible_distribution_file_path": "/etc/system-release", 
        "ansible_distribution_file_variety": "Amazon", 
        "ansible_distribution_major_version": "NA", 
        "ansible_distribution_release": "NA", 
        "ansible_distribution_version": "(Karoo)",
            
#We'll get a whole lot of info about the remote VM using the setup module.

[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -m setup | grep "ansible_os_family"
        "ansible_os_family": "RedHat", 
[ec2-user@ip-172-31-22-216 ~]$ 

[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -m setup | grep "memory" -A 15
        "ansible_memory_mb": {
            "nocache": {
                "free": 893, 
                "used": 90
            }, 
            "real": {
                "free": 762, 
                "total": 983, 
                "used": 221
            }, 
            "swap": {
                "cached": 0, 
                "free": 0, 
                "total": 0, 
                "used": 0
            }
            
[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -m setup | grep "ansible_all_ipv4_addresses" -A 2
        "ansible_all_ipv4_addresses": [
            "172.31.18.26"
        ], 
```

# Conditional Execution Using Facts and When Statement


```python
# In a case where we have VM's with different flavours of OS in our inventory.

# Our aim is to install httpd in all these VM's.

# While debian uses apt module to install packages, rhel uses yum package.

# We can use the when statement here.

---
- name: Playbook to install apazhe in different OS falvour.
  hosts: amazon
  become: yes
  tasks:
    - name: Install apache2 Debian
      when: ansible_os_family == "Debian"
      apt:
        name: apache2
        state: present
        update_cache: yes
            
    - name: Debian service restart
      when: ansible_os_family == "Debian"
      service:
        name: apache2
        state: restarted
        enabled: yes
            
    - name: Install httpd Red Hat
      when: ansible_os_distribution == "RedHat"
      yum:
        name: httpd
        state: present
        
    - name: Redhat service restart
      when: ansible_os_distribution == "RedHat"
      service:
        name: httpd
        state: restarted
        enabled: yes
            
            
            

            
            [ec2-user@ip-172-31-22-216 ~]$ ansible-playbook 03-01-httpd-on-redhat-debian.yml 

PLAY [Playbook to install apazhe in different OS falvour.] ***************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [172.31.18.26]

TASK [Install apache2 Debian] ********************************************************************************************************************************
skipping: [172.31.18.26]

TASK [Debian service restart] ********************************************************************************************************************************
skipping: [172.31.18.26]

TASK [Install httpd Red Hat] *********************************************************************************************************************************
ok: [172.31.18.26]

TASK [Redhat service restart] ********************************************************************************************************************************
changed: [172.31.18.26]

PLAY RECAP ***************************************************************************************************************************************************
172.31.18.26               : ok=3    changed=1    unreachable=0    failed=0   
```


```python
#Ansible will gather facts(run setup module) at the start of execution of every playbook.

#We can disable it for faster execution of playbook.

[ec2-user@ip-172-31-22-216 ~]$ cat 03-01-httpd-on-redhat-debian.yml 
---
- name: Playbook to install apazhe in different OS falvour.
  hosts: amazon
  become: yes
  gather_facts: no
  tasks:
```

## Include tasks


```python
#The playbook written in above example to install httpd in different OS flavour can be written in another way.

# We can split them into 3 different playbooks.


#Debian.yml which will have tasks only for debian falvoured OS
$vim debian.yml

---
- name: Install apache2 Debian
  apt:
    name: apache2
    state: present
    update_cache: yes
            
- name: Debian service restart
  service:
    name: apache2
    state: restarted
    enabled: yes

#Redhat.yml which will have tasks only for RHEL flavoured OS.
$vim redhat.yml

---
- name: Install httpd Red Hat
  yum:
    name: httpd
    state: present
        
- name: Redhat service restart
  service:
    name: httpd
    state: restarted
    enabled: yes

#Now we'll write a main playbook to execute the above 2 playbooks according to OS flavour.

$vim main.yml

---
- name: Install apache on multiple OS flavour
  become: yes
  hosts: amazon
  tasks:
    - include_tasks: debian.yml
      when: ansible_os_family == "Debian"
    - include_tasks: redhat.yml
      when: ansible_os_family == "RedHat"
        
#So basically we just have to execute this main.yml and it will automatically call debian or redhat.yml according to need.


```

## Debug module and Register keyword


```python
#The below playbook is written to create a file /tmp/test-syam.txt and write "This is test file" to it.
#Note : **contend** is used inside copy module to write the text to to dest file.

#The logs of a task will not be visible b default, so we regiter it to a variable "var_tmp_write"

#To view the logs we use debug module. Syntax below:

$vim 03-03-register-demo.yml

---
- name: Register and debug test
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Creating a tmp file
      copy:
        content: "This is test file"
        dest: /tmp/test-syam.txt
      register: var_tmp_write
        
    - name: Debugging the copy using the register variable
      debug:
        vars: var_tmp_write
```


```python
#Below is the output of the abov playbook.


[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook 03-03-register-demo.yml 

PLAY [Register and debug test] *******************************************************************************************************************************

TASK [Creating a tmp file] ***********************************************************************************************************************************
changed: [localhost]

TASK [Debugging the copy using the register variable] ********************************************************************************************************
ok: [localhost] => {
    "var_tmp_write": {
        "changed": true, 
        "checksum": "857504db1244e427e1cce312af67ec65be34bf91", 
        "dest": "/tmp/test-syam.txt", 
        "diff": [], 
        "failed": false, 
        "gid": 1000, 
        "group": "ec2-user", 
        "md5sum": "8a813afddc95f11494a907466bd32e59", 
        "mode": "0664", 
        "owner": "ec2-user", 
        "size": 17, 
        "src": "/home/ec2-user/.ansible/tmp/ansible-tmp-1560744252.94-128626510313672/source", 
        "state": "file", 
        "uid": 1000
    }
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0   

#Now if we run the playbook again, it will give below result.
#No changes will be made because the file is already there and its contends is also there.
    
    
[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook 03-03-register-demo.yml 

PLAY [Register and debug test] *******************************************************************************************************************************

TASK [Creating a tmp file] ***********************************************************************************************************************************
ok: [localhost]

TASK [Debugging the copy using the register variable] ********************************************************************************************************
ok: [localhost] => {
    "var_tmp_write": {
        "changed": false, 
        "checksum": "857504db1244e427e1cce312af67ec65be34bf91", 
        "dest": "/tmp/test-syam.txt", 
        "diff": {
            "after": {
                "path": "/tmp/test-syam.txt"
            }, 
            "before": {
                "path": "/tmp/test-syam.txt"
            }
        }, 
        "failed": false, 
        "gid": 1000, 
        "group": "ec2-user", 
        "mode": "0664", 
        "owner": "ec2-user", 
        "path": "/tmp/test-syam.txt", 
        "size": 17, 
        "state": "file", 
        "uid": 1000
    }
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   

[ec2-user@ip-172-31-22-216 ~]$ 
```

## Playbook to restart service only when a change is applied.




```python
$vim 03-04-restart-only-when-task-changed.yml

---
- name: Restart httpd only when a change is applied.
  hosts: amazon
  become: yes
  tasks:
    - name: Installing LAMP
      yum:
        name:
          - httpd
          - php
          - php-mysql
        state: present
      register: installation_status
    
    - name: Creating PHP file
      copy:
        content: '</php phpinfo(); ?>'
        dest: /var/www/html/index.php
      register: php_content_status
        
    - name: Restarting httpd
      service:
        name: httpd
        state: restarted
      when: installation_status.changed == true or php_content_status.changed == true
```

## Jinja2 Templates (template module)


```python
#Here we use a template file(sample.j2) and create it in a remote VM using the name variable.

[ec2-user@ip-172-31-22-216 ~]$ cat 03-05-jinja-template-demo.yml 
---
- hosts: localhost
  vars:
    name: Syam
  tasks:
    - name: "Template Demo"
      template: 
         src: sample.j2
         dest: /tmp/{{name}}.txt
            
            

[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook 03-05-jinja-template-demo.yml 
 [WARNING]: Found variable using reserved name: name


PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Template Demo] *****************************************************************************************************************************************
changed: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0   

[ec2-user@ip-172-31-22-216 ~]$ 

```
