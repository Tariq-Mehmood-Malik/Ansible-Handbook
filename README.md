# Ansible Handbook

# Ansible

Ansible is an open-source automation tool used to automate tasks in IT systems. It works as a remote controller for managing computers, servers, and other devices. Ansible can be used to set up software, configure systems, or run commands on each device.

It works by using simple ad-hoc commands or by using a book of instructions called playbooks written in a language called YAML. These playbooks tell Ansible what actions to perform on host devices.

It connects to target systems via SSH or WinRM. It's agentless, meaning no special software needs to be installed on the managed nodes. It is push-based, meaning the control node pushes out configurations or tasks to the host nodes when you execute an ad-hoc command/playbook.

It is known for its scalability, ease of use, and ability to manage diverse environments, including servers, cloud platforms, and network devices, all while ensuring consistency and reliability across systems.

## Key Terms of Ansible:
- **Ansible Controller**: The machine from which Ansible is run.
- **Module**: A unit of work Ansible uses to perform tasks.
- **Task**: A single action within a playbook.
- **Role**: A reusable set of tasks and configurations.
- **Fact**: Information about a system gathered by Ansible.
- **Play**: A section of a playbook targeting specific hosts with specific tasks.
- **Host**: A target machine managed by Ansible.
- **Inventory**: A file listing all hosts and groups of hosts for automation.

## Setting Up Ansible

It can be divided into 3 parts:
1. Configuring Ansible Controller
2. Configuring Ansible hosts
3. Setting up SSH 

### 1. Configuring Ansible Controller

1. First, we create the `ansible` user and provide it sudo privileges with the following commands (for Debian-based systems):

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

    **Basic Configuration:**
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
    ```bash
    sudo nano /etc/ansible/hosts
    ```

### 2. Configuring Ansible Hosts

Create the `ansible` user with sudo privileges and install Ansible on host systems with the following commands (for Debian-based systems):

```bash
sudo adduser ansible
sudo usermod -aG sudo ansible
su ansible
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt update
sudo apt install ansible
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

### 3. Setting up SSH

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

1. Copy the SSH key to the host using the following command:
    ```bash
    ssh-copy-id ansible@host_ip
    ```

**Verifying SSH sharing:**

1. To verify the SSH key sharing is successful, run:
    ```bash
    ssh ansible@host_ip
    ```

---

## Ansible Ad-Hoc Commands

In Ansible, an **ad-hoc command** is a one-time command that allows you to execute tasks on host systems without creating a full playbook. This is useful for quick, simple tasks. You can execute ad-hoc commands directly (without modules) with the `-a` flag.

### Format of ad-hoc commands (without module):

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
## Ansible Modules

You can use predefined **modules** in Ansible to perform tasks easily. Below are some examples:

### General Syntax:
```bash
ansible <hosts> -m <module> -a "<arguments>"
```
```bash
ansible all -m copy -a "src=/home/ansible/test.txt dest=/home/ansible/test.txt"
```

## Ansible Playbook

An **Ansible playbook** is a YAML file that defines a series of tasks to be executed on host systems. It allows you to run multiple tasks on multiple hosts in one go.

### Key Elements of a Playbook:
- **Target:** Defines the hosts and basic configurations for the playbook.
   ```yaml
---
- hosts: webservers
  become: yes  # Run tasks with sudo privileges
  ```
- **Tasks:** Each task defines a specific action to be performed. Tasks use Ansible modules to perform actions.
- **Variables:** Variables can be defined to customize the behavior of your playbook.
- **Handlers:** Special tasks that run only when notified by another task, usually for tasks like restarting services or reloading configurations after changes.

### Example of a Simple YAML Ansible Playbook:
```yaml
---
- hosts: webservers
  become: yes  # Run tasks with sudo privileges
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
