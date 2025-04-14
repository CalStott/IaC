
# Ansible Instance Setup

Setting up an Ansible controller and one target node (TN) – allowing the controller to SSH into the target. We'll use the Hosts file to do this.

---

## Controller Instance

- Launch instances
- Name: e.g. `tech503-caleb-ansible-controller`
- Image: **Quick Start** → **Ubuntu 22.04**
- Instance type: `t2.micro`
- Key pair: AWS key
- Network settings: Default  
  - Security group: Any with port 22 (e.g. `app`)
- Launch

---

## Target Node (TN) Instance

- Create instance with the **same settings** as the controller

---

## Setting up Dependencies for Controller

1. **SSH into the controller VM**
2. **Run the following commands:**

   ```bash
   python3 --version  # Check Python is installed
   sudo apt update -y  # Update package list
   sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y  # Upgrade packages
   sudo DEBIAN_FRONTEND=noninteractive apt-add-repository ppa:ansible/ansible  # Add Ansible PPA
   sudo DEBIAN_FRONTEND=noninteractive apt install ansible -y  # Install Ansible
   ansible --version  # Verify Ansible installation
   ```

   > **Note:** You’ll need to press `Enter` for the repository command to work interactively. To automate this, consider using `yes "" | sudo apt-add-repository ...`.

3. **Give controller SSH access to the target node:**

   ```bash
   cd /etc/ansible
   ls  # Verify files
   cd ~
   ls -a
   cd .ssh
   sudo nano private-key.pem  # Paste your private key here
   ```

   - Copy your private key file from your local machine to this file
   - Save and exit

4. **Manually SSH into the target node from the controller to verify it works**

   ```bash
   sudo ssh -i ~/.ssh/private-key.pem ubuntu@<target-node-ip>
   ```

5. **Configure Ansible Hosts File:**

   ```bash
   sudo nano /etc/ansible/hosts
   ```

   - Add the following:

     ```ini
     [web]
     ec2-instance ansible_host=<placeholder-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/.ssh/<placeholder-key-name>
     ```

6. **Test connection with ping:**

   ```bash
   sudo ansible all -m ping
   ```

7. **Optional: Suppress warning messages by editing config:**

   ```bash
   sudo nano /etc/ansible/ansible.cfg
   ```

   - Add:

     ```ini
     [defaults]
     interpreter_python = auto_silent
     ```

---

## Handy Ansible Commands

- View help:
  ```bash
  ansible-inventory --help
  ```

- List hosts:
  ```bash
  ansible-inventory --list
  ```

- Graph hosts:
  ```bash
  ansible-inventory --graph
  ```

- Run an Ad hoc command on all target nodes:
  ```bash
  sudo ansible web -a "command"
  ```

- Better practice with module specified:
  ```bash
  sudo ansible web -m ansible.builtin.command -a "uname -a"
  ```

  - `-m` = module
  - This is safer and more specific

---

## Apt Update & Upgrade Using Ansible

```bash
# sudo apt update using ansible
sudo ansible web -m ansible.builtin.apt -a "update_cache=yes" --become

# sudo apt upgrade using ansible
sudo ansible web -m ansible.builtin.apt -a "upgrade=dist" --become
```
