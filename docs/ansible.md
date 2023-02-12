# Ansible Cheatsheet

### installation

```sh
yum install ansible
apt install ansible
pacman -S ansible
dnf install ansible
```

```sh
python3 -m pip install ansible
```

remote machine should have 'python' - 'gather_facts: False' or 'gather_facts: no' otherwise

### uninstall

```sh
rm -rf $HOME/.ansible
rm -rf $HOME/.ansible.cfg
sudo rm -rf /usr/local/lib/python2.7/dist-packages/ansible
sudo rm -rf /usr/local/lib/python2.7/dist-packages/ansible-2.5.4.dist-info
sudo rm -rf /usr/local/bin/ansible
sudo rm -rf /usr/local/bin/ansible-config
sudo rm -rf /usr/local/bin/ansible-connection
sudo rm -rf /usr/local/bin/ansible-console
sudo rm -rf /usr/local/bin/ansible-doc
sudo rm -rf /usr/local/bin/ansible-galaxy
sudo rm -rf /usr/local/bin/ansible-inventory
sudo rm -rf /usr/local/bin/ansible-playbook
sudo rm -rf /usr/local/bin/ansible-pull
sudo rm -rf /usr/local/bin/ansible-vault
sudo rm -rf /usr/lib/python2.7/dist-packages/ansible
sudo rm -rf /usr/local/lib/python2.7/dist-packages/ansible
```

### ansible configuration places

- path variable $Ansible_Config
- ~/.ansible.cfg
- /etc/ansible/ansible.cfg

```sh
ansible-config view
ansible-config dump
```

#### configuration for external roles

filename: ~/.ansible.cfg

```properties
[defaults]
roles_path = ~/repos/project1/roles:~/repos/project2/roles
```

#### check configuration

```sh
ansible-config view
```

### inventory

#### without inventory inline host ip

```
ansible all -i desp000111.vantage.zur, --user=my_user -m "ping" -vvv
```

#### without inventory with pem ssh private ssh key

generate PEM file

```sh
ssh-keygen -t rsa -b 4096 -m PEM -f my_ssh_key.pem
ll my_ssh_key.pem

ansible all -i desp000111.vantage.zur, --user=vitalii.cherkashyn -e ansible_ssh_private_key_file=my_ssh_key.pem -m "ping" -vvv
```

#### ini file

```properties
# example cfg file
[web]
host1
host2 ansible_port=222 # defined inline, interpreted as an integer

[web:vars]
http_port=8080 # all members of 'web' will inherit these
myvar=23 # defined in a :vars section, interpreted as a string
```

### execute with specific remote python version, remote python, rewrite default variables, rewrite variables, override variable

```
--extra-vars "remote_folder=$REMOTE_FOLDER ansible_python_interpreter=/usr/bin/python"
```

### execute ansible for one host only, one host, one remove server, verbosity

```sh
ansible-playbook -i "ubs000015.vantage.org , " mkdir.yaml

ansible-playbook welcome-message.yaml -i airflow-test-account-01.ini --limit worker --extra-vars="ACCOUNT_ID=QA01" --user=ubuntu --ssh-extra-args="-i $EC2_KEY" -vvv

ansible all -i airflow-test-account-01.ini --user=ubuntu --ssh-extra-args="-i $EC2_KEY" -m "ping" -vvv
ansible main,worker -i airflow-test-account-01.ini --user=ubuntu --ssh-extra-args="-i $EC2_KEY" -m "ping"
```

simple file for creating one folder

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

### execute ansible locally, local execution

```sh
# --extra-vars="mapr_stream_path={{ some_variable_from_previous_files }}/some-argument" \

ansible localhost \
    --extra-vars="deploy_application=1" \
    --extra-vars=@group_vars/all/vars/all.yml \
    --extra-vars=@group_vars/ubs-staging/vars/ubs-staging.yml \
    -m include_role \
    -a name="roles/labeler"
```

### execute ansible-playbook with external paramters, bash script ansible-playbook with parameters, extra variables, external variables, env var

```j2
# variable from env
{{ lookup('env','DB_VARIANT_USERNAME') }}
```

```sh
ansible-playbook -i inventory.ini playbook.yml --extra-vars "$*"
```

with path to file for external parameters, additional variables from external file

```sh
ansible-playbook -i inventory.ini playbook.yml --extra-vars @/path/to/var.properties
ansible-playbook playbook.yml --extra-vars=@/path/to/var.properties
```

### external variables inline

```sh
ansible-playbook playbook.yml --extra-vars="oc_project=scenario-test mapr_stream_path=/mapr/prod.zurich/vantage/scenario-test"
```

### check is it working, ad-hoc command

```sh
ansible remote* -i inventory.ini -m "ping"
ansible remote* -i inventory.ini --module-name "ping"
```

```sh
ansible remote* -i inventory.ini -a "hostname"
```

## Playbooks

```sh
ansible-playbook <YAML>                   # Run on all hosts defined
ansible-playbook <YAML> -f 10             # Run 10 hosts parallel
ansible-playbook <YAML> --verbose         # Verbose on successful tasks
ansible-playbook <YAML> -C                # Test run
ansible-playbook <YAML> -C -D             # Dry run
ansible-playbook <YAML> -l <host>         # Run on single host
```

### Run Infos

```sh
ansible-playbook <YAML> --list-hosts
ansible-playbook <YAML> --list-tasks
```

### Syntax Check

```sh
ansible-playbook --syntax-check <YAML>

## Remote Execution

ansible all -m ping
```

### Execute arbitrary commands

```sh
ansible <hostgroup> -a <command>
ansible all -a "ifconfig -a"

## Debugging

### List facts and state of a host

```sh
ansible <host> -m setup                            # All facts for one host
ansible <host> -m setup -a 'filter=ansible_eth*'   # Only ansible fact for one host
ansible all -m setup -a 'filter=facter_*'          # Only facter facts but for all hosts
```

### Save facts to per-host files in /tmp/facts

```sh
ansible all -m setup --tree /tmp/facts
```

## Ansible Modules

Ansible modules are standalone scripts that can be used inside an Ansible playbook. You can use these modules to run whatever commands it needs to get its job done. Ansible modules are categorized into various groups based on their functionality. There are hundreds of Ansible modules are available.

### Format

```yaml
---
- hosts: production
  remote_user: root
  tasks:
  - ···
```

> Place your modules inside `tasks`.

### Task formats

#### One-line

```yaml
- apt: pkg=vim state=present
```

#### Map

```yaml
- apt:
    pkg: vim
    state: present
```

#### Foldable scalar

```yaml
- apt: >
    pkg=vim
    state=present
```

## Modules

### Aptitude

#### Packages

```yaml
- apt:
    pkg: nodejs
    state: present # absent | latest
    update_cache: yes
    force: no
```

#### Deb files

```yaml
- apt:
    deb: "https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb"
```

#### Repositories

```yaml
- apt_repository:
    repo: "deb https://··· raring main"
    state: present
```

#### Repository keys

```yaml
- apt_key:
    id: AC40B2F7
    url: "http://···"
    state: present
```

### git

```yaml
- git:
    repo: git://github.com/
    dest: /srv/checkout
    version: master
    depth: 10
    bare: yes
```

See: [git module](https://devdocs.io/ansible/git_module)

### git_config

```yaml
- git_config:
    name: user.email
    scope: global # local | system
    value: hi@example.com
```

See: [git_config module](https://devdocs.io/ansible/git_config_module)

### user

```yaml
- user:
    state: present
    name: git
    system: yes
    shell: /bin/sh
    groups: admin
    comment: "Git Version Control"
```

See: [user module](https://devdocs.io/ansible/user_module)

### service

```yaml
- service:
    name: nginx
    state: started
    enabled: yes     # optional
```

See: [service module](https://devdocs.io/ansible/service_module)

## Shell

### shell

```yaml
- shell: apt-get install nginx -y
```

#### Extra options

```yaml
- shell: echo hello
  args:
    creates: /path/file  # skip if this exists
    removes: /path/file  # skip if this is missing
    chdir: /path         # cd here before running
```

#### Multiline example

```yaml
- shell: |
    echo "hello there"
    echo "multiple lines"
```

See: [shell module](https://devdocs.io/ansible/shell_module)

### script

```yaml
- script: /x/y/script.sh
  args:
    creates: /path/file  # skip if this exists
    removes: /path/file  # skip if this is missing
    chdir: /path         # cd here before running
```

See: [script module](https://devdocs.io/ansible/script_module)

## Files

### file

```yaml
- file:
    path: /etc/dir
    state: directory # file | link | hard | touch | absent

    # Optional:
    owner: bin
    group: wheel
    mode: 0644
    recurse: yes  # mkdir -p
    force: yes    # ln -nfs
```

See: [file module](https://devdocs.io/ansible/file_module)

### copy

```yaml
- copy:
    src: /app/config/nginx.conf
    dest: /etc/nginx/nginx.conf

    # Optional:
    owner: user
    group: user
    mode: 0644
    backup: yes
```

See: [copy module](https://devdocs.io/ansible/copy_module)

### template

```yaml
- template:
    src: config/redis.j2
    dest: /etc/redis.conf

    # Optional:
    owner: user
    group: user
    mode: 0644
    backup: yes
```

See: [template module](https://devdocs.io/ansible/template_module)

---

## Playbook snippets

### Using templating

```sh
     {{ MYSTR | b64decode }}        # base64 decode MYSTR
     {{ MYSTR.split('\n') }}        # string splitting
```

### Capture shell output

```sh
tasks:
- name: some shell
  register: sh_out
  ignore_errors: yes
  become_user: root
  shell: |
    find /

- name: "Print stdout"
  debug:
    msg: "{{ sh_out.stdout.split('\n') }}"
- name: "Print stderr"
  debug:
    msg: "{{ sh_out.stderr.split('\n') }}"
```

### Handling files

```sh
tasks:
- name: file operation
  file:
      path: <file path>
      state: file

      # optional attributes examples
      mode: '0755'
      owner: <owner>
      group: <group>
      modification_time: now
      access_time: '{{ "%Y%m%d%H%M.%S" | strftime(stat_var.stat.atime) }}'
```

### Handling directories

```sh
tasks:
- name: Change directory
  file:
    path: <dir path>
    state: directory

    # optional attributes
    recurse: yes              # apply owner, group, mtime, atime, mode... recursively to all childs too
```

### Deleting files & directories

```sh
tasks:
- name: rm
  file:
    path: <some path>
    state: absent
    recurse: yes        # optional
```

### Operate on multiple files

For example multi file fetching

```sh
tasks:
- name: register files
  find: paths="somesrcpath" recurse=no patterns="*"
  register: myfiles

- debug: myfiles="{{ myfiles.files }}"

- name: perform file fetch
  fetch: src={{ item.path }} dest="somedestpath"
  with_items: "{{ myfiles.files }}"
{% endraw %}
```
