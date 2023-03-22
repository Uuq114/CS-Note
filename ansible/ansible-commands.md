[toc]

## installation

```bash
yum install ansible
apt install ansible
```

```bash
pip3 install ansible 
# for python2 - default installation 
pip install ansible
```

remote machine should have 'python' - 'gather_facts: False' or 'gather_facts: no' otherwise

gather_facts是用来收集各主机的facts信息，用来在playbook中引用的，设置为false就不会采集facts了



## ansible configuration places

* PATH variable: $Ansible_Config
* `~/.ansible.cfg`
* `/etc/ansible/ansible.cfg`

**config for external roles**

```cfg
[defaults]
roles_path = ~/repos/project1/roles:~/repos/project2/roles
```

**check config**

```bash
ansible-config view
```



## inventory

ini file:

```ini
# example cfg file
[web]
host1
host2 ansible_port=222 # defined inline, interpreted as an integer

[web:vars]
http_port=8080 # all members of 'web' will inherit these
myvar=23 # defined in a :vars section, interpreted as a string
```



## debug advice

set variable

```yaml
- name: Set Apache URL
  set_fact:
    apache_url: 'http://example.com/apache'
    
- name: Download Apache
  shell: wget {{ apache_url }}    
```

print variable

```
task.args
task.args['src']
vars()
```

debug message

```yaml
  - debug:
      msg: "print variable: {{  my_own_var }}"
```

```yaml
  - shell: /usr/bin/uptime
    register: result

  - debug:
      var: result
```



## example

**create folder, copy files**

```yaml
- hosts: all
  tasks:
    - name: Creates directory
      file:
        path: ~/spark-submit/trafficsigns
        state: directory
        mode: 0775
    - name: copy all files from folder
      copy: 
        src: "/home/projects/ubs/current-task/nodes/ansible/files" 
        dest: ~/spark-submit/trafficsigns
        mode: 0775

    - debug: msg='folder was amazoncreated for host {{ ansible_host }}'
```

