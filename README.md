### use below command

'''
ansible-playbook -i inventory.ini fetch_backup.yml
'''

![image](https://github.com/user-attachments/assets/61232de4-2c2b-4f01-9f69-173e272b7ab6)

![image](https://github.com/user-attachments/assets/a5d82a86-fd6b-40f8-976c-892bda1d84fd)



## From vm
It looks like there are a few issues and misunderstandings with the commands and configurations. Let’s break down and resolve each one step-by-step.

### **1. Fix Ansible Inventory File**

Your `inventory.ini` file is currently set up incorrectly for Ansible commands. Ansible expects `.ini` files to use the `[group]` format. You have a `[remote]` group but are referencing `master1` in your playbook, which does not exist. 

**Fix `inventory.ini`:**

```ini
[master1]
192.168.0.1 ansible_user=optit
```

Now, `master1` is correctly defined as a group containing `192.168.0.1`.

### **2. Fix `ansible-playbook` Commands**

You should run `ansible-playbook` with the playbook filename and use `inventory.ini` as the inventory file. You do not need to pass `inventory.ini` to `--syntax-check`. Instead, just run:

```bash
sudo ansible-playbook fetch_backup.yml --syntax-check
```

### **3. Correct `ansible-playbook` Command to Execute Playbook**

To run the `fetch_backup.yml` playbook, use:

```bash
sudo ansible-playbook fetch_backup.yml
```

### **4. Verify and Fix `ansible.cfg` File**

Your `ansible.cfg` file appears to be correct, but there are some common pitfalls:

- Make sure the `ansible.cfg` file is not empty or malformed.
- Ensure the path to `private_key` is correct and that the file has the correct permissions:

```bash
sudo chmod 600 /etc/ansible/myansible/cleanup_backup_check_diskusage/private_key
```

### **5. Ensure Correct `fetch_backup.yml` Configuration**

Your `fetch_backup.yml` file refers to a group `master1` that you have not defined correctly. Update `fetch_backup.yml`:

```yaml
---
- name: Download the latest Jenkins backup file from remote server
  hosts: master1
  tasks:
    - name: Find the latest backup zip file
      shell: ls -t /var/lib/jenkins/jenkins_backup/*.zip | head -1
      register: latest_file
      failed_when: "'No such file or directory' in latest_file.stderr"

    - name: Set the remote file path
      set_fact:
        remote_file_path: "{{ latest_file.stdout.strip() }}"

    - name: Download the latest backup file
      ansible.builtin.fetch:
        src: "{{ remote_file_path }}"
        dest: "~/Downloads/"
        flat: yes

    - name: Print the status
      debug:
        msg: "File downloaded successfully: {{ '~/Downloads/' + latest_file.stdout.split('/')[-1] }}"
```

### **6. Check and Add SSH Key Permissions**

Ensure the SSH private key has the right permissions:

```bash
sudo chmod 600 /etc/ansible/myansible/cleanup_backup_check_diskusage/private_key
```

### **7. Ensure Ansible Version Compatibility**

Ensure you are using a compatible version of Ansible. You can check the version with:

```bash
ansible --version
```

### **8. Running Playbooks**

After fixing the inventory and playbook, run the playbook with the correct inventory file:

```bash
sudo ansible-playbook -i inventory.ini fetch_backup.yml
```

### **9. Additional Troubleshooting**

If you still face issues, use the following commands to diagnose the problem:

#### **Verify SSH Connection**

```bash
ssh -i /etc/ansible/myansible/cleanup_backup_check_diskusage/private_key optit@192.168.0.1
```

#### **Check Ansible Connection**

```bash
ansible master1 -m ping -i inventory.ini -vvv
```

This command will provide verbose output to help diagnose connection issues.

### **10. Sample Directory Structure**

Ensure your directory structure is correct:

```
/etc/ansible/myansible/cleanup_backup_check_diskusage/
│
├── ansible.cfg
├── inventory.ini
├── fetch_backup.yml
├── private_key
```

### **Example Configuration and Execution**

Here's how everything should look and how to execute the commands:

#### **`ansible.cfg`**

```ini
[defaults]
inventory = inventory.ini
remote_user = optit
private_key_file = /etc/ansible/myansible/cleanup_backup_check_diskusage/private_key
host_key_checking = False  # Disable host key checking to avoid prompt issues
```

#### **`inventory.ini`**

```ini
[master1]
192.168.0.1 ansible_user=optit
```

#### **`fetch_backup.yml`**

```yaml
---
- name: Download the latest Jenkins backup file from remote server
  hosts: master1
  tasks:
    - name: Find the latest backup zip file
      shell: ls -t /var/lib/jenkins/jenkins_backup/*.zip | head -1
      register: latest_file
      failed_when: "'No such file or directory' in latest_file.stderr"

    - name: Set the remote file path
      set_fact:
        remote_file_path: "{{ latest_file.stdout.strip() }}"

    - name: Download the latest backup file
      ansible.builtin.fetch:
        src: "{{ remote_file_path }}"
        dest: "~/Downloads/"
        flat: yes

    - name: Print the status
      debug:
        msg: "File downloaded successfully: {{ '~/Downloads/' + latest_file.stdout.split('/')[-1] }}"
```

#### **Commands to Run**

```bash
sudo ansible-playbook -i inventory.ini fetch_backup.yml --syntax-check
sudo ansible-playbook -i inventory.ini fetch_backup.yml
```

### **Conclusion**

These steps will help you diagnose and resolve issues related to Ansible configuration and SSH connectivity. If issues persist, please share the verbose output of the commands for further troubleshooting.
