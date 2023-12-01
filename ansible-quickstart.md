# Ansible Quick Start Guide

#### You:
I have some Ubuntu VMs that I want to configure. Give me Ansible quick start.
I want to install and configure such things:
- java 17
- nginx
- open 8080 and 4443 ports
I have this VMs:
192.168.0.10
192.168.0.11
192.168.0.12
Put all required files to ~/ansible folder and show how to run them

#### ChatGPT:
To use Ansible for configuring your Ubuntu VMs with Java 17, Nginx, and to open the required ports, you need to follow these steps:

1. **Install Ansible**: Make sure Ansible is installed on your control machine (the machine you'll be running Ansible from).

2. **Inventory File**: Create an inventory file to list your VMs.

3. **Playbook**: Write an Ansible playbook to perform the required tasks.

4. **Run the Playbook**: Execute the playbook from your control machine.

Let's go through each step in detail.

### Step 1: Install Ansible on the Control Machine

You can install Ansible on a Linux machine using the package manager, for example:

```sh
sudo apt update
sudo apt install ansible -y
```

### Step 2: Create the Inventory File

Ansible uses an inventory file to track which servers it can interact with. Here's how you can set it up:

```ini
# ~/ansible/hosts
[webservers]
192.168.0.10
192.168.0.11
192.168.0.12
```

### Step 3: Write the Ansible Playbook

The playbook is where you define what you want Ansible to do on your servers. Below is an example playbook that installs Java 17, Nginx, and configures the firewall to open ports 8080 and 4443.

```yaml
# ~/ansible/main.yml
---
- hosts: webservers
  become: yes
  tasks:
    - name: Install Java 17
      apt:
        name: openjdk-17-jdk
        state: present

    - name: Install Nginx
      apt:
        name: nginx
        state: present
      notify:
        - Start Nginx

    - name: Open port 8080
      ufw:
        rule: allow
        port: '8080'
        proto: tcp

    - name: Open port 4443
      ufw:
        rule: allow
        port: '4443'
        proto: tcp

  handlers:
    - name: Start Nginx
      service:
        name: nginx
        state: started
```

Make sure the UFW firewall is enabled on your VMs before running the playbook. If not, you can add a task to enable it.

### Step 4: Run the Playbook

Navigate to the directory where your playbook and inventory file are located and run the following command:

```sh
ansible-playbook -i hosts main.yml
```

This command will execute your playbook against the hosts listed in the `hosts` file.

### Directory Structure

Your directory structure should look like this:

```plaintext
~/ansible/
├── hosts  # Inventory file
└── main.yml  # Playbook file
```

### Execution

To run the playbook, open a terminal and execute the following commands:

```sh
cd ~/ansible
ansible-playbook -i hosts main.yml
```

Make sure your control machine's SSH key is authorized on the target VMs, or alternatively, you can use username/password authentication (though not recommended for security reasons).

This is a quick start guide, and Ansible has a lot more to offer in terms of modules and complex tasks. You can refer to the Ansible documentation for more advanced usage and features.


#### You:
Now I have another VM
192.168.0.100
how to run my existing playbook only for this machine

#### ChatGPT:
To run your existing Ansible playbook on a new VM with the IP address `192.168.0.100`, you can use one of the following methods:

### Method 1: Add the New VM to the Inventory File

You can simply add the new VM to your existing inventory file under the same group or a new group. For example:

```ini
# ~/ansible/hosts
[webservers]
192.168.0.10
192.168.0.11
192.168.0.12
192.168.0.100
```

Then run the playbook as before:

```sh
ansible-playbook -i hosts main.yml
```

### Method 2: Run the Playbook Using the `-l` (Limit) Option

If you want to run the playbook only for the new VM without changing the inventory file, you can use the `-l` or `--limit` option followed by the IP address of the new VM. This will limit the playbook execution to the specified host(s).

```sh
ansible-playbook -i hosts main.yml -l 192.168.0.100
```

### Method 3: Use an Ad-Hoc Inventory

You can also provide an ad-hoc inventory directly at the command line without modifying the `hosts` file. To do this, specify the new VM's IP address after the `-i` option, and use a comma to denote it is a list:

```sh
ansible-playbook -i 192.168.0.100, main.yml
```

Note that the trailing comma is required when specifying a single host as an ad-hoc inventory list.

Choose the method that best fits your workflow. Method 2 is convenient if this is a one-time execution for the new VM. If the new VM is going to be a regular target for your playbooks, adding it to the inventory file (Method 1) is more maintainable.


#### You:
What if I don't have time to learn existing Ansible modules/features but just have sh commands to execute.
Give me a separate quick example how to execute one sh command to make things I want

#### ChatGPT:
If you have a shell command that you want to run on your remote machines without diving into the specifics of Ansible modules, you can use the `ansible` command-line tool with the `shell` or `command` module.

For instance, to install Java 17, Nginx, and open ports 8080 and 4443 using a shell command, you can run the following command from your Ansible control machine:

```sh
ansible all -i '192.168.0.100,' -m shell -a 'sudo apt update && sudo apt install -y openjdk-17-jdk nginx && sudo ufw allow 8080/tcp && sudo ufw allow 4443/tcp' -b
```

Let me break down the command for you:

- `ansible`: The Ansible command-line tool.
- `all`: Specifies the hosts pattern, in this case, we want to run the command on all hosts defined in the inventory. Replace `all` with the appropriate group if needed.
- `-i '192.168.0.100,'`: Specifies the inventory; the trailing comma tells Ansible that this is a list of hosts.
- `-m shell`: Specifies the module to use, in this case, the `shell` module which executes shell commands.
- `-a`: This is followed by the actual command you want to run in single quotes.
- `-b`: Tells Ansible to become another user, usually to escalate privileges, equivalent to `sudo`.

**Note**: It's important to be cautious when using the `shell` module because it pipes commands directly to the shell, which can be a security risk. Also, this approach does not take advantage of idempotency, which is one of the core benefits of using Ansible.

For a one-liner command like this, ensure you have SSH keys or passwordless sudo configured on the remote machine for the user you are connecting with, or else you'll need to provide a password interactively or through another method.


#### You:
How can I set some custom ssh commands like 'mkdir /etc/myservice' and also some custom command to verify it so it won't execute twice.
Show yaml example where it must be one shell command to execute an action and another ssh command to verify it's state

#### ChatGPT:
In Ansible, to execute a custom shell command and also have a verification step to ensure idempotency (that is, the command does not execute twice if not needed), you can use the `creates` parameter with the `command` module. The `creates` parameter tells Ansible not to run the command if a certain file or directory already exists.

Below is an example of an Ansible playbook with a task that creates a directory and checks for its existence to prevent re-execution:

```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Ensure the directory /etc/myservice exists
      command: mkdir /etc/myservice
      args:
        creates: /etc/myservice
```

This task will execute the `mkdir /etc/myservice` command only if the directory `/etc/myservice` does not already exist.

Here's a breakdown of the YAML file:

- `hosts: all` specifies that the playbook should run on all hosts in your inventory.
- `become: yes` indicates that tasks should be executed with privilege escalation (like sudo).
- `tasks` is a list of tasks to be executed on the hosts.
- `- name: ...` is a human-readable description of the task.
- `command: mkdir /etc/myservice` is the shell command to create the directory.
- `args` is a dictionary of arguments to the `command` module.
- `creates: /etc/myservice` tells Ansible to run the command only if the directory `/etc/myservice` does not exist.

To run this playbook, save it to a file (e.g., `create_directory.yml`), and run it with the `ansible-playbook` command:

```sh
ansible-playbook -i hosts create_directory.yml
```

This approach is one of the simplest ways to ensure a command is not run again if its changes are already in place, thus achieving a basic level of idempotency.


#### You:
what if my shell command is really custom so it can be verifyed only with another shell command, show an example for this

#### ChatGPT:
If your shell command must be verified by another shell command due to its custom nature, you would use the `register` directive to capture the output of a verification command, and then use a `when` condition to decide whether to run the main command. This is a bit more involved since Ansible does not inherently know the idempotent state of custom commands.

Here's an example playbook that illustrates this:

```yaml
---
- hosts: all
  become: yes
  tasks:
    - name: Check if the directory /etc/myservice exists
      command: stat /etc/myservice
      register: stat_result
      ignore_errors: true

    - name: Create directory /etc/myservice if it does not exist
      command: mkdir /etc/myservice
      when: stat_result is failed
```

Explanation of the YAML file:

- The first task uses the `stat` command to check if `/etc/myservice` exists. The `stat` command returns information about the file or directory if it exists or fails if it does not.
- `register: stat_result` captures the output and error code of the `stat` command.
- `ignore_errors: true` tells Ansible to continue with the next task even if the `stat` command fails (which it will if the directory does not exist).
- The second task actually creates the directory but will only run `when: stat_result is failed`, meaning it will only execute if the `stat` command indicates that the directory does not exist.

By using `stat` in the verification step, we utilize a command that's part of Linux's core utilities and is typically present on all systems. If you need to use a more complex verification command, simply replace the `stat` command with your custom verification command and adjust the `when` condition to match the expected output or error code that denotes the absence of the desired state.

To execute this playbook, save it to a file (e.g., `verify_and_execute.yml`) and run it with the `ansible-playbook` command:

```sh
ansible-playbook -i hosts verify_and_execute.yml
```

This approach ensures that your custom shell command is only executed when the verification command confirms that the action needs to be taken.
