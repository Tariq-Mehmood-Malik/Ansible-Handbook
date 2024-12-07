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
    Add the following configuration settings after the `[defaults]` section:
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