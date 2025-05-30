# Ansible
## ✅ What is ansible?
- Ansible is an open source **configuration management tool** that used to automate infrastructure management and configuration.
- Using ansible we can **manage thousands of servers with just a single command**.
- It is based on **push based mechanism**.
- **Agentless:** No need to install agents on managed nodes.
- It is developed by RedHat.
- Uses **SSH (Port 22)** for communication.
- **master slave architecture:**
   - **Control Node:** Runs Ansible commands and playbooks.
   - **Managed Nodes:** Remote servers managed by Ansible (no agents needed).
- /etc/ansible/  --> main directory of ansible.
- ansible.cfg --> A configuration file that controls Ansible’s behavior and default settings.
- hosts --> Inventory file defining groups and IPs of managed nodes
- ansible playbook (*.yml) --> A YAML file that defines a set of tasks to be executed on managed nodes.

![Screenshot (409)](https://github.com/Vaishnavi-M-Patil/AnsibleRepo/blob/main/assets/diagram.jpg)

### 🛠️ Ansible Concepts:
| **Concept**                      | **Description**                                                                                                                                       |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Modules**                      | Small programs that perform specific tasks such as installing software, managing files, or configuring services on remote hosts.                    |
| **Playbooks**                    | YAML files that define a series of tasks (using modules) to achieve a desired system state.                                                          |
| **Roles**                        | A way to organize and reuse playbooks by structuring tasks, handlers, variables, templates, and files into modular components.                      |
| **Inventory**                    | A list of managed hosts and their associated groups that Ansible uses to target specific servers or groups of servers.                              |
| **Tasks**                        | Individual units of action within a playbook, each specifying a module and its arguments.                                                            |
| **Ad-hoc Commands**              | Single, quick commands executed directly on a host, useful for testing or quick fixes.                                                               |
| **Ansible Vault**                | A tool for encrypting sensitive data like passwords and API keys within playbooks to ensure their security.                                          |
| **Ansible Galaxy**               | A community-driven repository where users can find and share roles and other Ansible content.                                                        |
| **Ansible Tower**                | A commercial product that adds features like web UI, job scheduling, and user management to Ansible.                                                 |
| **Configuration Management**     | The process of defining and managing the desired state of systems, which Ansible automates.                                                          |
| **CI/CD (Continuous Integration / Continuous Delivery)** | Ansible is often used in CI/CD pipelines to automate tasks like application deployment and infrastructure updates.            |

---

## ✅ yaml(yet another markup language):
- Easy to read and write
- YAML stands for yet another markup language or YAML ain’t markup language 
- Start with --- (Optional but good practice) and end with ... (optional Only used if you're combining multiple YAML documents in one file).
- .yaml or .yml file extension
- Comments can be identified with a pound or hash symbol (#).
- Use '-' for lists
- Key-value pairs with :
- Indentation is critical
   → Use spaces only, no tabs
   → Typically 2 spaces
- Strings with special characters need quotes
- Modules use dictionaries
```yaml
---
- name: Install and start Nginx          #Describes what the play or task does.
  hosts: webservers                      #Target group of servers defined in your inventory.
  become: yes                            #Run tasks with sudo/root privileges.

  tasks:                                #List of actions to perform.
    - name: Install Nginx
      apt:                                #Module used to manage packages (for Debian/Ubuntu).
        name: nginx                       
        state: present
        update_cache: yes                #Runs apt-get update before installation.

    - name: Start and enable Nginx
      service:                          #Module used to manage system services.
        name: nginx
        state: started
        enabled: yes                    #Enable service at boot	
...
```

--- 

## ✅ How ansible works?
1. Ansible uses playbook written in yaml syntax where we can define all tasks and configurations that we want to apply on all the servers.
2. Along with playbook we need **hosts file or inventory file** which will contains IP address of all servers on which we want to run playbook.  
![Screenshot](https://github.com/Vaishnavi-M-Patil/AnsibleRepo/blob/main/assets/inventory_file.png)  
   Here we used the private IP addresses of the instances because public IP addresses can change when an instance is stopped, whereas private IP addresses remain the same.
4. After editing playbook and host file, you can run the ansible command which will run playbook on all the servers defined the host file using ssh.

```
ansible -i hosts all -u ubuntu --private-key=./id_rsa -m shell -a hostname
```
The Ansible command is used for running the hostname command on all hosts listed in your inventory file using the shell module. 

**-i hosts-**	Specifies the inventory file named hosts.  
**all-**	Target group/hosts (here, all hosts in the inventory).  
**-u ubuntu-**	user name.  
**--private-key=./id_rsa-**	Path to the SSH private key for authentication.  
**-m shell-**	Use the shell module to run shell commands.  
**-a hostname-**	The argument to the shell module — in this case, the **hostname** command.  
![Screenshot](https://github.com/Vaishnavi-M-Patil/AnsibleRepo/blob/main/assets/ansible_cmd.png)


### 1stansible.yaml playbook(configuration file):
```yaml
- name: Message print
  hosts: all
  tasks:
  - name: Print message
    debug:
      msg: "Hello, World!"
```

### Command used to executes an Ansible playbook:
```
ansible-playbook 1stansible.yaml
```
| command | usecase|
| ----- | ------ |
| ansible | Ad-hoc commands are one-liner commands used to perform quick tasks on remote systems without writing a full playbook. |
| ansible-playbook | Run a YAML playbook with multiple tasks, logic, and roles |

---

## ✅ FQCN (Fully Qualified Collection Name):
- FQCN stands for Fully Qualified Collection Name.
- It is the complete path to a module or plugin in Ansible.
#### syntax:
```
<collection_namespace>.<collection_name>.<module_or_plugin_name>
```
#### example:
```
ansible.builtin.copy
ansible.builtin.fetch
```
#### Why Use FQCN?
- Avoid naming conflicts when multiple collections have modules with the same name.

---

## ✅ variables:
### local variable : 
- Defined within a task and used locally        
```yaml
- name: Install service
      apt:
        name: "{{ service }}"
        state: present
      vars:
        service: "nginx"
```
### Global variable:
Global variables are accessible across the entire playbook. 
```yaml
- name: Install and configure Nginx
  hosts: all
vars:
  service: "nginx"
tasks:
  - name: Install service
     apt:
        name: "{{ service }}"
        state: present
```
### Variable File:
External file that contains variables.
```yaml
- name: Install and configure apache2
  hosts: all
  vars_files:
    - ./var.txt
  tasks:
    - name: Install service
       apt:
          name: "{{ service }}"
          state: present
```
var.txt:
```
service: "apache2"
```

### command line variable:
```
ansible-playbook -e service=apache2 ansible.yaml
```

### prompt variable:
- Used to ask the user for input when the playbook runs.
- vars_prompt must be outside the **tasks:** block.
- Prompts are only available at playbook run time, not in ad-hoc commands.
- You can use input anywhere with {{ variable_name }}.
```yaml
- name: Prompt variable
  hosts: all
  vars_prompt:
    - name: movie
      prompt: Enter the movie name
      private: no                         # shows the input
  tasks:
    - name: print movie
       debug:
          msg: "This is {{ movie }}"
```

### register variable:
Stores the output of a command or task.
```yaml
- name: Register variable
  hosts: all
  tasks:
    - name: Get hostname
       shell: hostname
       register: host                           # store output of shell in host variable
    - name: print hostname
       debug: 
          msg: "This is {{ host.stdout }}"
```

### Pass variable from hosts file:
- In Ansible, you can pass variables directly from the hosts inventory file (usually hosts or inventory) by defining them next to the host entries or in group/host variable files.
- It contains least priority
```
192.168.1.10   movie = thar
```
```yaml
- name: Use variable from inventory
  hosts: all
  tasks:
    - debug:
        msg: "Movie name is {{ movie }}"
```

---

## ✅ Gathering facts:
The command **ansible all -m setup** is used to gather and display system facts (hardware, network, OS details, etc.) from all managed hosts.
```
ansible all -m setup
```
- **all:** Run on all hosts in your inventory.
- **-m setup:** Use the setup module, which collects facts about the system.
To view specific facts, use **-a** with a filter:
```
ansible all -m setup -a 'filter=ansible_hostname'
```
```yaml
- name: Print gathering facts
  hosts: all
  tasks:
    - debug:
        msg: "Hello this is {{ ansible_os_family }}"
```

---

## ✅ conditions:
Conditions in Ansible are used to **run tasks only when certain criteria are met**, using the *when* keyword.
```yaml
- name: Install Apache on Debian
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"
```

---

## ✅ Package module:
The package module is a generic way to manage software packages, regardless of the underlying package manager (like apt, yum, dnf etc.).
| State |	Description |
| ----- | ------ |
| present | Ensures the package is installed. Does nothing if already installed |
| absent |	Ensures the package is removed/uninstalled |
| latest | Ensures the latest version of the package is installed (if available) |
```yaml
- name: Print gathering facts
  hosts: all
  tasks:
    - name:  Install a package if not already installed
      package:
         name: nginx
         state: present
  
   - name: Remove package
     package:
        name: apache2
        state: absent

   - name: Install the latest version of a package
     package:
       name: curl
       state: latest
```

---

## ✅ privilege in ansible:
- In Ansible, privilege escalation allows you to run tasks as root user, even if you're connected as a regular user.
- If **become: yes** is set in the playbook, then all tasks in the playbook will run with elevated privileges.
```yaml
- name: Install nginx with root privileges
  hosts: webservers
  become: yes                                    # Enable sudo
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```
You can also use it per task if only some tasks need sudo:
```yaml
tasks:
  - name: Run a command as root
    command: apt update
    become: yes
```
You can configure privilege (become) globally using the **ansible.cfg** file.

Edit your /etc/ansible/ansible.cfg and add:
```
[defaults]
become = True
```
run an ad-hoc command with privilege:
```
ansible all -m apt -a "name=nginx state=present" -b                  # Here -b = --become
```

---

## ✅ service module:
- The service module in Ansible is used to start, stop, restart, or enable/disable services on target machines.
- On modern systems (like Ubuntu 18.04+, CentOS 7+), this works with systemd.

| Parameter |	Description |
| ------- |----------- |
| name | Name of the service (e.g., nginx, httpd) |
| state | Desired state: started, stopped, restarted, reloaded |
| enabled |	Whether the service should start at boot: yes / no |

```yaml
- name: Manage nginx service
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure nginx is started
      service:
        name: nginx
        state: started
        enabled: yes
```

### daemon_reload in Ansible's systemd Module:
- The daemon_reload option in the systemd module is used to tell systemd to reload its configuration. 
- Use daemon_reload: yes after:
   - Creating or modifying a systemd service file (/etc/systemd/system/myapp.service)
   - Changing unit file properties (ExecStart, Environment, etc.)
- **daemon_reload** does not restart services.
- It only refreshes systemd's in-memory configuration database.
```yaml
- name: Install and configure a custom service
  hosts: all
  become: yes
  tasks:
    - name: Copy custom systemd unit file
      copy:
        src: myservice.service
        dest: /etc/systemd/system/myservice.service
        mode: '0644'

    - name: Reload systemd daemon
      systemd:
        daemon_reload: yes

    - name: Start and enable the custom service
      systemd:
        name: myservice
        state: started
        enabled: yes
```

---

## ✅ Loops in Ansible: 
Loops in Ansible are used to repeat a task multiple times with different items.
```yaml
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - git
    - curl
```
Loop with dictionaries:
```yaml
- name: Create users with specific shells
  hosts: all
  become: yes
  tasks:
    - name: Add users with custom shells
      user:
        name: "{{ item.name }}"
        shell: "{{ item.shell }}"
        state: present
      loop:
        - { name: 'alice', shell: '/bin/bash' }
        - { name: 'bob', shell: '/bin/sh' }
```

---

## ✅ Copy module: [for more](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
The copy module is used to copy files from your Ansible control node (local machine) to the managed hosts.
```yaml
- name: Copy a file to remote server
  copy:
    src: /path/to/local/file.txt
    dest: /path/on/remote/file.txt
    # owner: www-data
    # group: www-data
    # mode: '0644'
```
| Option | Description |
| ------ | ---------- |
| src	| Path to the source file on the control node |
| dest | Path on the managed node to copy the file to |
| owner | Set the file owner on the remote system |
| group | Set the file group on the remote system |
| mode |	Set file permissions (e.g., '0644') |
| content |	Use this instead of src to directly write text into the destination file |
| backup |	Creates a backup before overwriting an existing file |

---

## ✅ fetch module:
- This module works like ansible.builtin.copy, but in reverse.
- It is used for fetching files from remote machines and storing them locally in a file tree, organized by hostname.
- Files that already exist at dest will be overwritten if they are different than the src.
- This module is also supported for Windows targets.
#### Example:
```yaml
- name: Store file into /tmp/fetched/host.example.com/tmp/somefile
  ansible.builtin.fetch:
    src: /tmp/somefile
    dest: /tmp/fetched
```
 if `dest=/tmp/fetched`, then `src=/tmp/somefile` on host `host.example.com`, would save the file into `/backup/host.example.com/etc/profile`. The host name is based on the inventory name.

---

## ✅ win_copy module :
- The win_copy module copies a file on the local box to remote windows locations.
- For non-Windows targets, use the ansible.builtin.copy module instead.
- It is recommended that backslashes \ are used instead of / when dealing with remote paths.

#### Notes:
   - This module is part of the ansible.windows collection (version 2.8.0).
   - You might already have this collection installed if you are using the ansible package. It is not included in ansible-core.
   - To check whether it is installed, run: `ansible-galaxy collection list`
   - To install it, use: `ansible-galaxy collection install ansible.windows`
   - To use it in a playbook, specify: `ansible.windows.win_copy`

#### Example:
```yaml
- name: Copy a single file
  ansible.windows.win_copy:
    src: /srv/myfiles/foo.conf
    dest: C:\Temp\renamed-foo.conf

- name: Copy a single file, but keep a backup
  ansible.windows.win_copy:
    src: /srv/myfiles/foo.conf
    dest: C:\Temp\renamed-foo.conf
    backup: true

- name: Copy a single file keeping the filename
  ansible.windows.win_copy:
    src: /src/myfiles/foo.conf
    dest: C:\Temp\

- name: Copy folder to C:\Temp (results in C:\Temp\temp_files)
  ansible.windows.win_copy:
    src: files/temp_files
    dest: C:\Temp

- name: Copy folder contents recursively
  ansible.windows.win_copy:
    src: files/temp_files/
    dest: C:\Temp

- name: Copy a single file where the source is on the remote host
  ansible.windows.win_copy:
    src: C:\Temp\foo.txt
    dest: C:\ansible\foo.txt
    remote_src: true

- name: Copy a folder recursively where the source is on the remote host
  ansible.windows.win_copy:
    src: C:\Temp
    dest: C:\ansible
    remote_src: true

- name: Set the contents of a file
  ansible.windows.win_copy:
    content: abc123
    dest: C:\Temp\foo.txt

```

---

## ✅ what is lineinfile, blockinfile, tags in ansible:
In Ansible, lineinfile and blockinfile are modules used to manage the contents of files on a target system, especially for configuration management. 

### lineinfile module:
The lineinfile module in Ansible is used to add, modify, or remove lines in a file. The lineinfile module looks for a specific line in a file and replaces it with a predefined regular expression.

### blockinfile module:
blockinfile inserts, modifies, or deletes one or several lines of text located between two marker lines in a file.

### tags:
Tags in Ansible allow you to run specific parts of a playbook rather than executing everything. 

```yaml
- name: add line in index.html
      lineinfile:
        path: /var/www/html/index.html
        line: "Hello this line is added in this file"
      tags: lineinfile

    - name: Add block in index.html
      blockinfile:
        path: /var/www/html/index.html
        block: |
          <h2>Hello world!</h2>
          <p>Hello world!</p>
        marker: "# {mark} ANSIBLE MANAGED BLOCK"
      tags: blockinfile
```
---

## ✅ Roles:
### What Are Roles in Ansible?
- Roles are a way to organize and reuse Ansible code.
- They split complex playbooks into smaller, manageable, reusable components.
- Roles help in structuring tasks, variables, files, handlers, templates, and more into a standard layout.

### Steps to Create a Role:
- Navigate to your Ansible project directory.
- Use the command:
```
ansible-galaxy init <role_name>
```
Example:
```
ansible-galaxy init apache
```
- Roles are stored in the `roles/` directory.
- A default role structure is created under `roles/apache`.

### Role directory structure:
An Ansible role has a defined directory structure with seven main standard directories. You must include at least one of these directories in each role. You can omit any directories the role does not use.
```
roles/
    common/               
        tasks/             # Main list of tasks
            main.yml      
        handlers/         #  Contains handlers that can be triggered by other tasks.
            main.yml      
        templates/        #  templates to use in the role or child roles
            ntp.conf.j2   
        files/            # Contains files that can be deployed by the role.
            bar.txt         
            foo.sh        
        vars/             # higher priority variables associated with this role 
            main.yml      
        defaults/         #  default lower priority variables for this role
            main.yml      
        meta/             # Metadata about the role and role dependencies
            main.yml       

```
- By default, Ansible will look in most role directories for a main.yml file.

### Using Role in a Playbook:
```yaml
- name: Setup Web Server
  hosts: web
  roles:
    - webserver
```

You can add other YAML files in some directories, but they won’t be used by default. They can be included/imported directly.For example, you can place platform-specific tasks in separate files and refer to them in the tasks/main.yml file:
```yaml
# roles/example/tasks/main.yml
- name: Install the correct web server for RHEL
  import_tasks: redhat.yml
  when: ansible_os_family == 'RedHat'

- name: Install the correct web server for Debian
  import_tasks: debian.yml
  when: ansible_os_family == 'Debian'

# roles/example/tasks/redhat.yml
- name: Install web server
  ansible.builtin.yum:
    name: "httpd"
    state: present

# roles/example/tasks/debian.yml
- name: Install web server
  ansible.builtin.apt:
    name: "apache2"
    state: present
```
---

## What is inventory_hostname?
- inventory_hostname is a special Ansible variable that refers to the name of the host as it is defined in your inventory file.
- If you don’t create an alias in your Ansible inventory file then inventory_hostname will simply be the IP address itself.

#### `hosts` file:
![inventory-name](https://github.com/Vaishnavi-M-Patil/Ansible/blob/main/assets/inventory-name-set.png)
In this case:
- inventory_hostname for 172.31.88.128 is ubuntu-server.
- inventory_hostname for 172.31.85.135 is ec2-server.
Even though ansible_host is the actual IP address of managed nodes, inventory_hostname refers to the alias (ubuntu-server, ec2-server).

#### In a playbook:
![playbook](https://github.com/Vaishnavi-M-Patil/Ansible/blob/main/assets/inventory-name-call.png)
This would generate:
![output1](https://github.com/Vaishnavi-M-Patil/Ansible/blob/main/assets/ec2-output.png)
![output1](https://github.com/Vaishnavi-M-Patil/Ansible/blob/main/assets/ubuntu-output.png)
