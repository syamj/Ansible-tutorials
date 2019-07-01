
# Deploying a repo from Git.


```python
[ec2-user@ip-172-31-22-216 git-deployment]$ cat main.yml 
---
- name: Deploying website from Git repository
  hosts: all
  become: yes
  tasks:
    - name: Installing softwares
      yum: name=git,httpd state=present

    - name: Creating local code repository
      file: path=/var/git-repo state=absent

    - name: Creating local code repository
      file: path=/var/git-repo state=directory mode=700 owner=root group=root
    
    - name: Cloning latest commit from git
      git: repo=https://github.com/syamj/ansible-git.git dest=/var/git-repo
      register: repo_download

    - name: Copying contents to Docroot
      shell: 'cp -r /var/git-repo/* /var/www/html/'
      when: repo_download.changed == true

    - name: Ensure directories are set to apache.apache
      shell: find /var/www/html -exec chown apache:apache {} \;

    - name: Ensure directories are set to 755
      shell: find /var/www/html -type d -exec chmod 0755 {} \;

    - name: Ensure files are 644 
      shell: find /var/www/html -type f -exec chmod 0644 {} \;

    - name: restart softwares
      service: name=httpd state=restarted enabled=yes
      when: repo_download.changed
```

# Read value from csv file


```python
- hosts: localhost
  become: yes
  tasks:
    - debug: msg="The default shell of Syam is {{ lookup('csvfile', 'syam file=users.txt delimiter=,') }}"

[ec2-user@ip-172-31-22-216 playbooks]$ cat users.txt 
john,/bin/bash
doe,/bin/sh
sam,/sbin/nologin
syam,/bin/bash
fuji,/bin/sh
[ec2-user@ip-172-31-22-216 playbooks]$ 
```


```python
[ec2-user@ip-172-31-22-216 playbooks]$ ansible-playbook shell-check.yml 

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [debug] *************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "The default shell of Syam is /bin/bash"
}

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0   
```

# To Add multiple Users to a remote machine



```python

[ec2-user@ip-172-31-22-216 playbooks]$ cat multiple-user-add.yml 
- hosts: amazon
  become: yes
  tasks:
    - shell: cat users.txt
      register: content
      delegate_to: localhost
    
    - user: name="{{item.split(',')[0]}}" shell="{{item.split(',')[1]}}"
      with_items: 
        - "{{ content.stdout.split() }}"
[ec2-user@ip-172-31-22-216 playbooks]$ 
```
