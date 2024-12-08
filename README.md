# Ansible

Ansible is an open-source automation tool used to automate tasks in IT systems. It works as a remote controller for managing computers, servers, and other devices. Ansible can be used to set up software, configure systems, or run commands on each device.

It works by using simple ad-hoc commands or by using a book of instructions called playbooks written in a language called YAML. These playbooks tell Ansible what actions to perform on host devices.

It connects to target systems via SSH or WinRM. It's agentless, meaning no special software needs to be installed on the managed nodes. It is push-based, meaning the control node pushes out configurations or tasks to the host nodes when you execute an ad-hoc command/playbook.

It is known for its scalability, ease of use, and ability to manage diverse environments, including servers, cloud platforms, and network devices, all while ensuring consistency and reliability across systems.

### Key Terms of Ansible:
- **Ansible Controller**: The machine from which Ansible is run.
- **Module**: A unit of work Ansible uses to perform tasks.
- **Task**: A single action within a playbook.
- **Role**: A reusable set of tasks and configurations.
- **Fact**: Information about a system gathered by Ansible.
- **Play**: A section of a playbook targeting specific hosts with specific tasks.
- **Host**: A target machine managed by Ansible.
- **Inventory**: A file listing all hosts and groups of hosts for automation.

  ## Ansible Inventory
   An inventory is a list of managed nodes that Ansible communicates with. The inventory can be static (a file) or dynamic (automatically generated from cloud providers or other sources). 
   Inventory is used to define hosts and groups of hosts.
   ### 1. Static Inventory (INI format)
   A static inventory is a simple, static file that lists all the hosts that Ansible will manage. It is commonly written in the INI format, which contains a list of groups and the hosts in each group.
   **Example of a static inventory in INI format:**
   
   ```ini
   [webservers]       # Group name
   web1.example.com   # Host Ip or Domain address
   web2.example.com
   
   [databases]
   db1.example.com
   db2.example.com
   ```
   ### 2. Dynamic Inventory
   A dynamic inventory allows Ansible to automatically pull information about hosts from external sources, such as cloud providers like AWS, GCP, and others. This is particularly useful for cloud environments where hosts might be created or destroyed dynamically.
   

# Setting Up Ansible Insfrastructure

## Creating ansible user on Controller

First, we create the `ansible` user and provide it sudo privileges with the following commands(we will use this user to perform all task on host systems you can use your own user for it and can skip this step):
   
   **Creating user ansible:**
   ```bash
   sudo adduser ansible
   ```

   **Giving ansible user sudo privileges:**
   ```bash
   sudo usermod -aG sudo ansible
   ```

   **Switching to ansible user (for verification):**
   ```bash
   su ansible
   ```
## Creating ansible user on Hosts

   Create the `ansible` user with sudo privileges:
   
   ```bash
   sudo adduser ansible
   sudo usermod -aG sudo ansible
   su ansible
   ```
  ### Setting up Password-less Sudo Privileges on Host

   To set up password-less sudo access for the `ansible` user on the host system, follow these steps:
   Open the sudoers file using `visudo`:
       ```bash
       sudo visudo
       ```
   Add the following line in the end of sudoers file:
       ```
       ansible   ALL=(ALL)		NOPASSWD: ALL
       ```
## Installing Ansible Software on Controller

   Now that we have created the ansible user on the controller, we need to install and configure Ansible on the controller with the following commands:
    **Installing Ansible:**
   ```bash
   sudo apt update
   sudo apt install ansible
   ```
   **Verifying Ansible installation:**
   ```bash
   ansible --version
   ```
   **Creating default Configuration file:**
   ```bash
   cd /etc/ansible 
   sudo su                # switch to root user to create config file
   ansible-config init -t all --disabled > ansible.cfg
   exit                   # exit from root user
   ```
   **Modifying the configuration file:**
   ```bash
   sudo nano /etc/ansible/ansible.cfg
   ```
   Add the following configuration settings in the `[defaults]` section:
   ```
   remote_user=ansible      # can skip this if you are not creating ansible user
   host_key_checking=False
   ```
   **Adding host group & host IPs in inventory:**
   Enter the host group & names in inventory
   ```bash
   sudo nano /etc/ansible/hosts
   ```
  Example of inventory file with single group name `websever` with 2 hosts
   ```ini
   [webserver]          # Group name
   192.168.0.1          # 1st Host Ip 
   web2.example.com     # 2nd Host domain address
   ```
## Setting up Ansible for linux based Hosts
  ### Setting up SSH connection
   Ansible use SSH for linux based host to enables secure, passwordless communication between the controller and managed (host) nodes. By using SSH keys, Ansible automates authentication, enhancing security and scalability, 
   while avoiding the need for manual passwords.
   
   **Creating SSH key for ansible user on the Ansible controller:**
   
   1. Generate the SSH key pair for the `ansible` user:
       ```bash
       ssh-keygen -t rsa
       ```
   
   2. **Viewing the Ansible controller public SSH key:**
       ```bash
       cat .ssh/id_rsa.pub
       ```

   3. **Sharing SSH key with the host:**
   
   SHare the SSH key with host (linux based) using the following command:
   ```bash
   ssh-copy-id user_name@host_ip  
   ```
   Replace `user_name` with ansible if you have created ansible username on host system & `host_ip` with IP address of host
   
   4. **Verifying SSH sharing:**
   To verify the SSH key shared successfully, run:
   ```bash
   ssh user_name@host_ip
   ```

## Setting up Ansible for Windows Host
  ### Installing pywinrm on controller
  `pywinrm` is a Python library that allows Ansible to communicate with Windows hosts using Windows Remote Management (WinRM). WinRM is a Microsoft protocol that enables remote management of Windows machines, and pywinrm is used by Ansible to interact with Windows systems over WinRM. (Make sure python3 is pre-installed in system)
  ```bash
  pip3 install pywinrm
  ```
  If you are getting error try using following command
  ```bash
  sudo apt install python3-winrm
  ```
  ### Setting inventory file for windows host on controller
  winrm connection requires additional information in to establish connection. This information can be added in inventory as following:
   ```ini
  [windows]
  192.168.0.122            # window host IP

  [windows:vars]
  ansible_user=Administrator        #Host machine user name
  ansible_password=your_password    #Host machine password
  ansible_connection=winrm          
  ansible_winrm_server_cert_validation=ignore  # or 'validate' for secure connections
  ```
  ### Configure a Windows host for Ansible
  We can configure windows host for Ansible by running shell script in Poweshell.
  1. Download this [script](https://github.com/ansible/ansible-documentation/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1) from github.
  2. Open PowerShell as administartor
  3. Execute follwoing command in PowerShell
  ```shell
  powershell.exe -ExecutionPolicy ByPass path\to\file\ConfigureRemotingForAnsible.ps1
  ```
  4. After that run `win_ping` module on controller to test connection.
  ```bash
  ansible [group_name] -m win_ping
  ```
# Ansible Ad-Hoc Commands

   In Ansible, an **ad-hoc command** is a one-time command that allows you to execute tasks on host systems without creating a full playbook. This is useful for quick, simple tasks. You can       
   execute ad-hoc commands directly (without modules) with the `-a` flag.
   
   ### Format of ad-hoc commands (without module)
   
   ```bash
   ansible <hosts> -a "<shell command>"
   ```
   
   ### Some examples of ad-hoc commands:
   
   ```bash
   ansible host -a "uptime"
   ```
   ```bash
   ansible host -a "mkdir /home/ansible/test"
   ```

# Ansible Modules
   
   You can use predefined **modules** in Ansible to perform tasks easily. Below are some examples:
   
   ### General Syntax:
   ```bash
   ansible <hosts> -m <module> -a "<arguments>"
   ```
   ### Exaple of copy module:
   ```bash
   ansible all -m copy -a "src=/home/ansible/test.txt dest=/home/ansible/test.txt"
   ```
   Folllowing methos can be use to search details about any Ansible module:
   ## 1. Using the `ansible-doc` Command
   The `ansible-doc` command allows you to access documentation about any Ansible module directly from the command line.
   ### Basic Syntax:
   ```bash
   ansible-doc <module_name>
   ```
   ### Example
   To get details about the `copy` module, use:
   
   ```bash
   ansible-doc copy
   ```
   ## 2. Using `ansible-doc` with `-l` for Listing Modules
   If you are unsure about the module name, you can list all available modules and search for the one you're interested in.
   ### To list all modules:
   ```bash
   ansible-doc -l
   ```
   ## 3. Using the Ansible Documentation Website
   You can also find module documentation on the official Ansible website:
   
   **Ansible Module Index:** [https://docs.ansible.com/ansible/2.8/modules/list_of_all_modules.html](https://docs.ansible.com/ansible/2.8/modules/list_of_all_modules.html)
   
   This page lists all the available Ansible modules.


# Ansible Playbook

   An **Ansible playbook** is a YAML file that defines a series of tasks to be executed on host systems. It allows you to run multiple tasks on multiple hosts in one go.
   
   ### Key Elements of a Playbook:
   - **Target:** Defines the hosts and basic configurations for the playbook.
   ```yaml
   ---
   - hosts: webservers
     become: yes  # Run tasks with sudo privileges
   ```
   - **Tasks:** Each task defines a specific action to be performed. Tasks use Ansible modules to perform actions.
   ```yaml
   ---
     tasks:
       - name: Install nginx package
         apt:
           name: nginx
           state: present
       - name: Start nginx service
         service:
           name: nginx
           state: started
   ```
   - **Variables:** Variables can be defined to customize the behavior of your playbook.
   ```yaml
   ---
     vars:
       package_name: nginx
   
     tasks:
       - name: Install the specified package
         apt:
           name: "{{ package_name }}"
           state: present
   ```
   - **Handlers(notify):** Special tasks that run only when notified by another task, usually for tasks like restarting services or reloading configurations after changes.
   ```yaml
   ---
     tasks:
       - name: Install nginx package
         apt:
           name: nginx
           state: latest
         notify:
           - Restart nginx
   
     handlers:
       - name: Restart nginx
         service:
           name: nginx
           state: restarted
   ```
   
   ### Example: Playbook with Target, Task, Variables and Handlers
   
   This playbook installs or updates the `nginx` package, and if the package is changed, it notifies the handler to restart the `nginx` service.
   
   ```yaml
   ---
   - hosts: webservers
     become: yes  # Run tasks with sudo privileges
     vars:
       default_package: nginx
   
     tasks:
       - name: Install or update nginx package
         apt:
           name: "{{ package_name | default(default_package) }}"
           state: latest
         notify:
           - Restart nginx
         vars:
           package_name: apache2  # Task-specific variable overrides default
     
       - name: Ensure nginx is started
         service:
           name: "{{ default_package }}"
           state: started
   
     handlers:
       - name: Restart nginx
         service:
           name: nginx
           state: restarted
   ```
   
## Extra Features of a Playbook
   
   ### 1. Condition (`when`):
   The `when` statement in a playbook is used to execute a task only when a certain condition is met. The condition is typically a logical expression or variable value.
   
   **Example:**
   ```yaml
   - name: Install nginx if the condition is true
     ansible.builtin.yum:
       name: nginx
       state: present
     when: ansible_facts['os_family'] == 'RedHat'
   ```
   ### 2. Loops:
   You can use the `loop` directive to iterate over a list of items and apply the same task for each item.
   
   **Example:**
   ```yaml
   - name: Install multiple packages
     ansible.builtin.apt:
       name: "{{ item }}"
       state: present
     loop:
       - nginx
       - git
       - vim
   ```
   ### 3. Tags:
   Tags allow you to run specific parts of a playbook by assigning tags to tasks or plays. This helps you execute only certain parts of the playbook rather than running everything.
   
   **Example:**
   ```yaml
   tasks:
     - name: Install nginx
       ansible.builtin.yum:
         name: nginx
         state: present
       tags:
         - install
   ```

# Ansible Vault
   
   Ansible Vault is a feature that allows you to securely store sensitive data such as passwords or API keys in your Ansible projects. This data is encrypted and can be decrypted only with a password.
   
   ### 1. Creating a Vault File
   You can create an encrypted file using the `ansible-vault create` command.
   **Example:**
   ```bash
   ansible-vault create secrets.yml
   ```
   This command will prompt you for a password, which you will need to decrypt the file later or to perform other task on file.
   
   ### 2. Editing a Vault File
   To edit an encrypted vault file, use the `ansible-vault edit` command.
   
   **Example:**
   ```bash
   ansible-vault edit secrets.yml
   ```
   
   ### 3. Viewing an Encrypted File
   To view the contents of an encrypted file, use the `ansible-vault view ` command.
   
   **Example:**
   ```bash
   ansible-vault edit secrets.yml
   ```
   
   ### 4. Encrypting an Existing File
   If you have a plain text file that you want to encrypt, you can use the `ansible-vault encrypt` command.
   
   **Example:**
   ```bash
   ansible-vault encrypt plain_file.yml
   ```
   
   ### 5. Decrypting a Vault File
   To decrypt an encrypted file, use the `ansible-vault decrypt` command.
   
   **Example:**
   ```bash
   ansible-vault decrypt secrets.yml
   ```
   
   ### 6. Running Playbooks with Vaults
   When running a playbook that includes encrypted files, use the `--ask-vault-pass` option to prompt for the vault password or `--vault-password-file` to specify a password file.
   
   **Example:**
   ```bash
   ansible-playbook --ask-vault-pass playbook.yml
   ```
   Alternatively, to use a password file:
   ```bash
   ansible-playbook --vault-password-file=~/.vault_pass playbook.yml
   ```
