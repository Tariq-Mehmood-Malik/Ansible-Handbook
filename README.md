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
   web1.example.com   # Host Ip or Domain name
   web2.example.com
   
   [databases]
   db1.example.com
   db2.example.com
   ```
   ### 2. Dynamic Inventory
   A dynamic inventory allows Ansible to automatically pull information about hosts from external sources, such as cloud providers like AWS, GCP, and others. This is particularly useful for cloud environments where hosts might be created or destroyed dynamically.
   
   ### Example: AWS
   For AWS, you can use the `ec2.py` script (provided by Ansible) to dynamically generate the inventory based on the EC2 instances running in your AWS account.
   
   You can configure a dynamic inventory in a configuration file or use an inventory plugin to interact with cloud APIs.
   
   ### Example Command:
   To list the inventory from a dynamic source, you can run the following command:
   
   ```bash
   ansible-inventory -i aws_ec2.yaml --list
   ```

# Setting Up Ansible Insfrastructure

   It can be divided into 3 parts:
   1. Configuring Ansible Controller
   2. Configuring Ansible hosts
   3. Setting up SSH 

## 1. Configuring Ansible Controller

First, we create the `ansible` user and provide it sudo privileges with the following commands (for Debian-based systems):
   
   **Creating user ansible:**
   ```bash
   sudo adduser ansible
   ```

   **Giving ansible user sudo privileges:**
   ```bash
   sudo usermod -aG sudo ansible
   ```

   **Switching to ansible user:**
   ```bash
   su ansible
   ```

   Now that we have created the ansible user on the controller, we need to install and configure Ansible on the controller with the following commands:
    **Installing Ansible:**
   ```bash
   sudo apt install software-properties-common
   sudo add-apt-repository --yes --update ppa:ansible/ansible
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
   sudo su
   ansible-config init -t all --disabled > ansible.cfg
   exit
   ```

   **Modify the configuration:**
   ```bash
   sudo nano /etc/ansible/ansible.cfg
   ```
   Add the following configuration settings in the `[defaults]` section:
   ```
   remote_user=ansible
   host_key_checking=False
   ```

   **Adding host group & host IPs in inventory:**
   Enter the host group & names in inventory
   ```bash
   sudo nano /etc/ansible/hosts
   ```

## 2. Configuring Ansible Hosts

   Create the `ansible` user with sudo privileges (ansible user will be use to execute commands received from ansible controller) on host systems with the following commands (for Debian-based        systems):
   
   ```bash
   sudo adduser ansible
   sudo usermod -aG sudo ansible
   su ansible
   ```

### Setting up Password-less Sudo Privileges on Host

   To set up password-less sudo for the `ansible` user on the host system, follow these steps:
   
   1. Open the sudoers file using `visudo`:
       ```bash
       sudo visudo
       ```
   
   2. Add the following line in the sudoers file:
       ```
       ansible   ALL=(ALL)		NOPASSWD: ALL
       ```

## 3. Setting up SSH

   In Ansible, SSH enables secure, passwordless communication between the control node and managed nodes. By using SSH keys, Ansible automates authentication, enhancing security and scalability, 
   while avoiding the need for manual passwords. This method also simplifies management and key rotation.
   
   **Creating SSH key for ansible user on the Ansible controller:**
   
   1. Generate the SSH key pair for the `ansible` user:
       ```bash
       ssh-keygen -t rsa
       ```
   
   2. **Viewing the Ansible controller public SSH key:**
       ```bash
       cat .ssh/id_rsa.pub
       ```
   
   **Sharing SSH key with the host:**
   
   Copy the SSH key to the host using the following command:
   ```bash
   ssh-copy-id ansible@host_ip # replace host_ip with host real IP
   ```
   
   **Verifying SSH sharing:**
   
   To verify the SSH key sharing is successful, run:
   ```bash
   ssh ansible@host_ip # replace host_ip with host real IP
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
