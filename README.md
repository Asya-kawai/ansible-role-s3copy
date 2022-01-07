# Ansible Role For Copy backup files to AWS S3.

Setup AWS credentials and s3 copy settings.

This role copies backup files(.tar.gz only) in `/var/` to AWS S3.

# Examples

Directory tree.

```
.
├── README.md
├── ansible.cfg
├── group_vars
│   ├── all.yml
│   ├── webserver_centos.yml
│   └── webserver_ubuntu.yml
├── inventory
└── webservers.yml
```

## Inventory file

```
[webservers:children]
webserver_ubuntu
webserver_centos

[webserver_ubuntu]
ubuntu ansible_host=10.10.10.10 ansible_port=22

[webserver_centos]
centos ansible_host=10.10.10.11 ansible_port=22
```

## Group Vars / Common settings(all.yml)

`all.yml` sets common variables.

```
# Common settings
become: yes
ansible_user: root

# Private_key is saved in local host only!
ansible_ssh_private_key_file: ""
```

## Group Vars / Ubuntu(webserver_ubuntu.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

This role references the `aws` variable to setup aws credentials,
and also use the `s3copy` variable to setup copy services.

The following example shows sets root user's aws credentials(stored in /root/.aws/credentials) and
copies backup files(.tar.gz format) to s3://backup as the root user every day at 00:00:00.

Note:

The format of `s3copy.on_calender` follows the `OnCalender` option of systemd service.

```
ansible_user: ubuntu
become: yes
ansible_become_password: 'ThisIsSecret!'

aws:
  credentials:
    - user: root
      content: |
        [default]
        aws_access_key_id = AKIAIOSFODNN7EXAMPLE
        aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

s3copy:
  user: root
  group: root
  on_calender: '*-*-* 00:00:00'
  dest: s3://backup
```

Of course, you can define it as a general setting in `all.yml`.

## Group Vars / CentOS(webserver_centos.yml)

`webserver_ubuntu.yml` is `webservers` host's children.

The following example shows that the root user copies files to s3://backup daily at 00:00:00.

```
# Use all.yml's settings.

aws:
  credentials:
    - user: root
      content: |
        [default]
        aws_access_key_id = AKIAIOSFODNN7EXAMPLE
        aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

s3copy:
  user: root
  group: root
  on_calender: '*-*-* 00:00:00'
  dest: s3://backup
```

## Playbook / Webservers(webservers.yml)

```
- hosts: webservers
  become: yes
  module_defaults:
    apt:
      cache_valid_time: 86400
  roles:
    - user
```

# How to DryRun and Apply

DryRun

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -CD webservers.yml --tags s3copy
```

Apply

```
ansible-playbook -i inventory --private-key="~/.ssh/your_private_key" -D webservers.yml --tags s3copy
```
