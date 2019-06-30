
# Ansible


```python
#Ansible is a configuration management tool.

#Ansible is agentless, so we do not need to install anything in remote VM.

#Ansible is required only in control machine and it connects to remote machine via ssh.

#Passwordless ssh should be configured for ansible to run.
```

## To install Ansible in an Amazon AMI env

sudo amazon-linux-extras install ansible2

## Creating Inventory files

$ cat inventory.ini 

[amazon]
172.31.18.26 ansible_port=22 ansible_user='ec2-user' # ansible_ssh_private_key_file='path_to_ssh_key'

## How to check if a machine is reachable using Ansible(via ssh)


```python
#Ping module can be used to check if ansible is able to reach a remote server and execute code there.

ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m ping
172.31.18.26 | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "ping": "pong"
}
[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini all -m ping
172.31.18.26 | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "ping": "pong"
    }
```

## Shell module


```python
#Shell module is used to run shell commands in a remote VM using ansible

[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m shell -a 'free -m'
172.31.18.26 | SUCCESS | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            983          59         600           0         323         772
Swap:             0           0           0
```

## Different modules to run a shell command in ansible


```python
#We can run shell commands using 3 different modules.

1. Shell

[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m shell -a 'free -m'
172.31.18.26 | SUCCESS | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            983          59         600           0         323         772
Swap:             0           0           0

#This invokes a shell env in the remote vm and then executes the command and returns the result

2. Command Module

[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m command -a 'free -m'
172.31.18.26 | SUCCESS | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            983          59         600           0         323         772
Swap:             0           0           0

#Command module executes a script without being preceeded by a shell. So shell variables like $HOME etc will not be available 
# And also stream operations like  <, >, | and & will not work.
# Command module is more secure, because it will not be affected by the user’s environment.

3. RAW Module


[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m raw -a 'free -m'
172.31.18.26 | SUCCESS | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            983          48         612           0         323         784
Swap:             0           0           0
Shared connection to 172.31.18.26 closed.

# Executes a low-down and dirty SSH command, not going through the module subsystem.
# ie it basicall works like ssh remote-vm -e "free -m"
```

## Another example to depict between the differences of these 3 modules


```python
[ec2-user@ip-172-31-22-216 ~]$ ssh ec2-user@172.31.18.26 'ls /etc/*.conf | wc -l'
27
[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m shell -a 'ls /etc/*.conf | wc -l'
172.31.18.26 | SUCCESS | rc=0 >>
27

[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m command -a 'ls /etc/*.conf | wc -l'
172.31.18.26 | FAILED | rc=2 >>
ls: cannot access /etc/*.conf: No such file or directory
ls: cannot access |: No such file or directory
ls: cannot access wc: No such file or directorynon-zero return code

[ec2-user@ip-172-31-22-216 ~]$ ansible -i inventory.ini amazon -m raw -a 'ls /etc/*.conf | wc -l'
172.31.18.26 | SUCCESS | rc=0 >>
27
Shared connection to 172.31.18.26 closed.
```

## Resetting the default inventory files.


```python
# /etc/ansible/ansible.cfg is the default inventory file. We can add our own inventory file so that we donot want to mention 
# the inventory file where executing the commands each and every time.b

[ec2-user@ip-172-31-22-216 ~]$ grep inventory /etc/ansible/ansible.cfg 
#inventory      = /etc/ansible/hosts
inventory      = /home/ec2-user/inventory.ini
```

## Yum module


```python
#Below shows an amsible ad hoc command to install httpd on a remote computer.

[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -m yum -a 'name=httpd state=present'
172.31.18.26 | FAILED! => {
    "changed": false, 
    "failed": true, 
    "msg": "You need to be root to perform this command.\n", 
    "rc": 1, 
    "results": [
        "Loaded plugins: extras_suggestions, langpacks, priorities, update-motd\n"
    ]
}

#The above didn't work because we need to be root to install an package so, -b option is added(for being sudo)


[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m yum -a 'name=httpd state=present'
172.31.18.26 | SUCCESS => {
    "changed": true, 
    "failed": false, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "Loaded plugins: extras_suggestions, langpacks, priorities, update-motd\nResolving Dependencies\n--> Running transaction check\n---> Package httpd.x86_64 0:2.4.39-1.amzn2.0.1 will be installed\n--> Processing Dependency: httpd-tools = 2.4.39-1.amzn2.0.1 for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: httpd-filesystem = 2.4.39-1.amzn2.0.1 for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: system-logos-httpd for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: mod_http2 for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: httpd-filesystem for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: /etc/mime.types for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: libaprutil-1.so.0()(64bit) for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Processing Dependency: libapr-1.so.0()(64bit) for package: httpd-2.4.39-1.amzn2.0.1.x86_64\n--> Running transaction check\n---> Package apr.x86_64 0:1.6.3-5.amzn2.0.2 will be installed\n---> Package apr-util.x86_64 0:1.6.1-5.amzn2.0.2 will be installed\n--> Processing Dependency: apr-util-bdb(x86-64) = 1.6.1-5.amzn2.0.2 for package: apr-util-1.6.1-5.amzn2.0.2.x86_64\n---> Package generic-logos-httpd.noarch 0:18.0.0-4.amzn2 will be installed\n---> Package httpd-filesystem.noarch 0:2.4.39-1.amzn2.0.1 will be installed\n---> Package httpd-tools.x86_64 0:2.4.39-1.amzn2.0.1 will be installed\n---> Package mailcap.noarch 0:2.1.41-2.amzn2 will be installed\n---> Package mod_http2.x86_64 0:1.14.1-1.amzn2 will be installed\n--> Running transaction check\n---> Package apr-util-bdb.x86_64 0:1.6.1-5.amzn2.0.2 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package                 Arch       Version                Repository      Size\n================================================================================\nInstalling:\n httpd                   x86_64     2.4.39-1.amzn2.0.1     amzn2-core     1.3 M\nInstalling for dependencies:\n apr                     x86_64     1.6.3-5.amzn2.0.2      amzn2-core     118 k\n apr-util                x86_64     1.6.1-5.amzn2.0.2      amzn2-core      99 k\n apr-util-bdb            x86_64     1.6.1-5.amzn2.0.2      amzn2-core      19 k\n generic-logos-httpd     noarch     18.0.0-4.amzn2         amzn2-core      19 k\n httpd-filesystem        noarch     2.4.39-1.amzn2.0.1     amzn2-core      23 k\n httpd-tools             x86_64     2.4.39-1.amzn2.0.1     amzn2-core      87 k\n mailcap                 noarch     2.1.41-2.amzn2         amzn2-core      31 k\n mod_http2               x86_64     1.14.1-1.amzn2         amzn2-core     147 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package (+8 Dependent packages)\n\nTotal download size: 1.9 M\nInstalled size: 5.1 M\nDownloading packages:\n--------------------------------------------------------------------------------\nTotal                                              8.8 MB/s | 1.9 MB  00:00     \nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : apr-1.6.3-5.amzn2.0.2.x86_64                                 1/9 \n  Installing : apr-util-bdb-1.6.1-5.amzn2.0.2.x86_64                        2/9 \n  Installing : apr-util-1.6.1-5.amzn2.0.2.x86_64                            3/9 \n  Installing : httpd-tools-2.4.39-1.amzn2.0.1.x86_64                        4/9 \n  Installing : generic-logos-httpd-18.0.0-4.amzn2.noarch                    5/9 \n  Installing : mailcap-2.1.41-2.amzn2.noarch                                6/9 \n  Installing : httpd-filesystem-2.4.39-1.amzn2.0.1.noarch                   7/9 \n  Installing : mod_http2-1.14.1-1.amzn2.x86_64                              8/9 \n  Installing : httpd-2.4.39-1.amzn2.0.1.x86_64                              9/9 \n  Verifying  : apr-util-1.6.1-5.amzn2.0.2.x86_64                            1/9 \n  Verifying  : apr-util-bdb-1.6.1-5.amzn2.0.2.x86_64                        2/9 \n  Verifying  : httpd-tools-2.4.39-1.amzn2.0.1.x86_64                        3/9 \n  Verifying  : httpd-2.4.39-1.amzn2.0.1.x86_64                              4/9 \n  Verifying  : httpd-filesystem-2.4.39-1.amzn2.0.1.noarch                   5/9 \n  Verifying  : apr-1.6.3-5.amzn2.0.2.x86_64                                 6/9 \n  Verifying  : mod_http2-1.14.1-1.amzn2.x86_64                              7/9 \n  Verifying  : mailcap-2.1.41-2.amzn2.noarch                                8/9 \n  Verifying  : generic-logos-httpd-18.0.0-4.amzn2.noarch                    9/9 \n\nInstalled:\n  httpd.x86_64 0:2.4.39-1.amzn2.0.1                                             \n\nDependency Installed:\n  apr.x86_64 0:1.6.3-5.amzn2.0.2                                                \n  apr-util.x86_64 0:1.6.1-5.amzn2.0.2                                           \n  apr-util-bdb.x86_64 0:1.6.1-5.amzn2.0.2                                       \n  generic-logos-httpd.noarch 0:18.0.0-4.amzn2                                   \n  httpd-filesystem.noarch 0:2.4.39-1.amzn2.0.1                                  \n  httpd-tools.x86_64 0:2.4.39-1.amzn2.0.1                                       \n  mailcap.noarch 0:2.1.41-2.amzn2                                               \n  mod_http2.x86_64 0:1.14.1-1.amzn2                                             \n\nComplete!\n"
    ]
}



#Now if we run the same command again, ansible would do nothing because httpd is alread present in the remote VM.

[ec2-user@ip-172-31-22-216 ~]$ 
[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m yum -a 'name=httpd state=present'
172.31.18.26 | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "msg": "", 
    "rc": 0, 
    "results": [
        "httpd-2.4.39-1.amzn2.0.1.x86_64 providing httpd is already installed"
    ]
}
[ec2-user@ip-172-31-22-216 ~]$ 
```

## Service Module


```python
#Service module is used to check status/start/stop/restart any specific service running in the remote VM.

#The below will restart httpd in the remote VM

[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m service -a 'name=httpd state=restarted'
172.31.18.26 | SUCCESS => {
    "changed": true, 
    "failed": false, 
    "name": "httpd", 
    "state": "started", 

    
   
    
#Below will check if httpd is started, if not will start it. If alread started, it will do nothing.    
    [ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m service -a 'name=httpd state=started'
172.31.18.26 | SUCCESS => {
    "changed": false, 
    "failed": false, 
    "name": "httpd", 
    "state": "started", 
```

## Copy module in ansible


```python
#Copy module is used to copy a file/text to a file in remote VM.

#This will copy index.html from the execution directory and copy it to /var/www/html/ of remote VM.

[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m copy -a 'src=index.html dest=/var/www/html/'
172.31.18.26 | SUCCESS => {
    "changed": true, 
    "checksum": "754b1df117c447f8caf63d0fff4aa002066a5fe0", 
    "dest": "/var/www/html/index.html", 
    "failed": false, 
    "gid": 0, 
    "group": "root", 
    "md5sum": "7ef01811d5b57b410f436204a04e416a", 
    "mode": "0644", 
    "owner": "root", 
    "size": 51, 
    "src": "/home/ec2-user/.ansible/tmp/ansible-tmp-1560501987.83-1848944088840/source", 
    "state": "file", 
    "uid": 0
}

#Now if we run the command again, nothing will change because both the files are exactly same.


[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m copy -a 'src=index.html dest=/var/www/html/'
172.31.18.26 | SUCCESS => {
    "changed": false, 
    "checksum": "754b1df117c447f8caf63d0fff4aa002066a5fe0", 
    "dest": "/var/www/html/index.html", 
    "failed": false, 
    "gid": 0, 
    "group": "root", 
    "mode": "0644", 
    "owner": "root", 
    "path": "/var/www/html/index.html", 
    "size": 51, 
    "state": "file", 
    "uid": 0
}

#After making changes in the index.html file, if we run the command, it will execute the copy and update remote VM with new file.


[ec2-user@ip-172-31-22-216 ~]$ vim index.html 
[ec2-user@ip-172-31-22-216 ~]$ 
[ec2-user@ip-172-31-22-216 ~]$ ansible amazon -b -m copy -a 'src=index.html dest=/var/www/html/'
172.31.18.26 | SUCCESS => {
    "changed": true, 
    "checksum": "10fc37f59d03f64a6cc8aa6cc59923f2bc812ee8", 
    "dest": "/var/www/html/index.html", 
    "failed": false, 
    "gid": 0, 
    "group": "root", 
    "md5sum": "012322d1b2bda728e68e82d8ef98380a", 
    "mode": "0644", 
    "owner": "root", 
    "size": 51, 
    "src": "/home/ec2-user/.ansible/tmp/ansible-tmp-1560502055.65-6558282353603/source", 
    "state": "file", 
    "uid": 0
}
[ec2-user@ip-172-31-22-216 ~]$ 
```

## Playbooks


```python
#Playbooks are written in yml format. 
#Intendation is really important for yml files.
# A yml file starts with ---


#Here we write the above examples of httpd install, restart service and copy index.html using a single playbook.

[ec2-user@ip-172-31-22-216 ~]$ cat httpd-install.yml 
---
- name: installing httpd on amazon linux
  hosts: amazon
  become: yes
  tasks: 
    - name: installing httpd
      yum: 
        name: httpd
        state: present
    - name: restarting httpd
      service:
        name: httpd
        state: restarted
    - name: creating index.html
      copy:
        src: index.html
        dest: /var/www/html/
   
[ec2-user@ip-172-31-22-216 ~]$ ansible-playbook httpd-install.yml 

PLAY [installing httpd on amazon linux] ******************************************************************************

TASK [Gathering Facts] ***********************************************************************************************
ok: [172.31.18.26]

TASK [installing httpd] **********************************************************************************************
ok: [172.31.18.26]

TASK [restarting httpd] **********************************************************************************************
changed: [172.31.18.26]

TASK [creating index.html] *******************************************************************************************
changed: [172.31.18.26]

PLAY RECAP ***********************************************************************************************************
172.31.18.26               : ok=4    changed=2    unreachable=0    failed=0   
```


```python

```
