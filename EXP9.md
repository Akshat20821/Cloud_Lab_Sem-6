\# exp9

\# Experiment 9 — Ansible: Configuration Management and Automation



\## Theory



\### Problem Statement



Managing infrastructure manually across multiple servers leads to configuration drift, inconsistent environments, and time-consuming repetitive tasks. Scaling from one server to hundreds becomes nearly impossible with manual SSH-based administration.



\### What is Ansible?



Ansible is an open-source automation tool for \*\*configuration management\*\*, \*\*application deployment\*\*, and \*\*orchestration\*\*. It follows an \*\*agentless architecture\*\* — using SSH for Linux and WinRM for Windows — and uses YAML-based \*\*playbooks\*\* to define automation tasks.



Ansible has become the standard choice among enterprise automation solutions because it is simple yet powerful, agentless, community-powered, predictable, and secure.



\### How Ansible Solves the Problem



| Problem | Ansible Solution |

|---|---|

| Agent installation on every server | Agentless — uses SSH only |

| Running playbooks twice breaks things | Idempotency — same result every time |

| Imperative scripts hard to read | Declarative YAML — describe desired state |

| Waiting for changes to propagate | Push-based — initiates from control node immediately |



\---



\## Key Concepts



| Component | Description |

|---|---|

| \*\*Control Node\*\* | Machine with Ansible installed — where you run commands |

| \*\*Managed Nodes\*\* | Target servers — no Ansible agent needed |

| \*\*Inventory\*\* | File listing all managed nodes (EC2 instances, servers, etc.) |

| \*\*Playbooks\*\* | YAML files containing a sequence of automation steps |

| \*\*Tasks\*\* | Individual actions in playbooks (e.g., installing a package) |

| \*\*Modules\*\* | Built-in functionality to perform tasks (e.g., `apt`, `yum`, `service`) |

| \*\*Roles\*\* | Pre-defined reusable automation scripts |



\### How Ansible Works



Ansible connects from the \*\*control node\*\* to the \*\*managed nodes\*\* via SSH, sending commands and instructions. The units of code it executes are called \*\*modules\*\*. Each module is invoked by a \*\*task\*\*, and an ordered list of tasks forms a \*\*playbook\*\*. The managed machines are listed in an \*\*inventory file\*\* grouped into categories.



```

Control Node (Ansible installed)

&#x20;       │

&#x20;       │ SSH

&#x20;       ├──────────── Managed Node 1

&#x20;       ├──────────── Managed Node 2

&#x20;       └──────────── Managed Node 3

```



No extra agents required on managed nodes — just a terminal and a text editor to get started.



\---



\## Part A — Hands-On Lab



\### Step 1: Install Ansible



\*\*Via pip (recommended for macOS/Linux):\*\*



```bash

pip install ansible

ansible --version

```



\*\*Via apt (Ubuntu/Debian):\*\*



```bash

sudo apt update -y

sudo apt install ansible -y

ansible --version

```



\*\*Post-installation check:\*\*



```bash

ansible localhost -m ping

```



\*\*Expected output:\*\*



```

localhost | SUCCESS => {

&#x20;   "changed": false,

&#x20;   "ping": "pong"

}

```



\---



\### Step 2: Create SSH Key Pair



Ansible uses SSH key-based authentication to connect to managed nodes without passwords.



```bash

ssh-keygen -t rsa -b 4096

\# Accept all defaults — keys saved to \~/.ssh/id\_rsa and \~/.ssh/id\_rsa.pub

```



Copy keys to the current working directory (needed for building the Docker image):



```bash

cp \~/.ssh/id\_rsa.pub .

cp \~/.ssh/id\_rsa .

```



\*\*Key placement explained:\*\*



| File | Location | Purpose |

|---|---|---|

| `id\_rsa` (Private Key) | Control node only | Used to authenticate when connecting — \*\*never share this\*\* |

| `id\_rsa.pub` (Public Key) | Remote server `\~/.ssh/authorized\_keys` | Grants access to anyone with the matching private key |



\---



\### Step 3: Create the Docker Image (Ubuntu SSH Server)



Create a `Dockerfile` that builds a custom Ubuntu image with OpenSSH pre-configured and our public key baked in:



```dockerfile

FROM ubuntu



RUN apt update -y

RUN apt install -y python3 python3-pip openssh-server

RUN mkdir -p /var/run/sshd



\# Configure SSH

RUN mkdir -p /run/sshd \&\& \\

&#x20;   echo 'root:password' | chpasswd \&\& \\

&#x20;   sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd\_config \&\& \\

&#x20;   sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd\_config \&\& \\

&#x20;   sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd\_config



\# Create .ssh directory and set proper permissions

RUN mkdir -p /root/.ssh \&\& \\

&#x20;   chmod 700 /root/.ssh



\# Copy SSH keys into the image

COPY id\_rsa /root/.ssh/id\_rsa

COPY id\_rsa.pub /root/.ssh/authorized\_keys



\# Set proper permissions

RUN chmod 600 /root/.ssh/id\_rsa \&\& \\

&#x20;   chmod 644 /root/.ssh/authorized\_keys



\# Fix for SSH login

RUN sed -i 's@session\\s\*required\\s\*pam\_loginuid.so@session optional pam\_loginuid.so@g' /etc/pam.d/sshd



\# Expose SSH port

EXPOSE 22



\# Start SSH service

CMD \["/usr/sbin/sshd", "-D"]

```



Build the image:



```bash

docker build -t ubuntu-server .

```



\---

<img width="1007" height="653" alt="image" src="https://github.com/user-attachments/assets/f0cc8944-550d-4b03-95c7-3a6cf518d7f5" />



\### Step 4: Launch 4 Server Containers



```bash

for i in {1..4}; do

&#x20;   echo -e "\\n Creating server${i}\\n"

&#x20;   docker run -d --rm -p 220${i}:22 --name server${i} ubuntu-server

&#x20;   echo -e "IP of server${i} is $(docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' server${i})"

done

```



\*\*Expected output:\*\*



```

Creating server1

IP of server1 is 172.17.0.2



Creating server2

IP of server2 is 172.17.0.3



Creating server3

IP of server3 is 172.17.0.4



Creating server4

IP of server4 is 172.17.0.5

```



\---



\### Step 5: Create Ansible Inventory



This script auto-generates `inventory.ini` with the real container IPs:



```bash

echo "\[servers]" > inventory.ini

for i in {1..4}; do

&#x20;   docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' server${i} >> inventory.ini

done



cat << EOF >> inventory.ini



\[servers:vars]

ansible\_user=root

ansible\_ssh\_private\_key\_file=\~/.ssh/id\_rsa

ansible\_python\_interpreter=/usr/bin/python3

EOF

```



Review the generated file:



```bash

cat inventory.ini

```



\*\*Expected `inventory.ini` content:\*\*



<img width="974" height="558" alt="image" src="https://github.com/user-attachments/assets/589eb69a-f85e-4816-8457-1a9a63c5b58f" />





\### Step 6: Test Connectivity



Manual SSH test to confirm keys work:



```bash

ssh -i \~/.ssh/id\_rsa root@172.17.0.2

```



Ansible ping test across all servers:



```bash

ansible all -i inventory.ini -m ping

```



\*\*Expected output:\*\*



```

172.17.0.2 | SUCCESS => {

&#x20;   "changed": false,

&#x20;   "ping": "pong"

}

172.17.0.3 | SUCCESS => {

&#x20;   "changed": false,

&#x20;   "ping": "pong"

}

172.17.0.4 | SUCCESS => {

&#x20;   "changed": false,

&#x20;   "ping": "pong"

}

172.17.0.5 | SUCCESS => {

&#x20;   "changed": false,

&#x20;   "ping": "pong"

}

```

<img width="988" height="385" alt="image" src="https://github.com/user-attachments/assets/b779199c-7bf3-427f-bdfe-5cdc0bd19572" />



For verbose output (useful for debugging):











\### Step 7: Create and Run Playbook (`update.yml`)





```yaml

\---

\- name: Update and configure servers

&#x20; hosts: all

&#x20; become: yes

&#x20; tasks:



&#x20;   - name: Update apt packages

&#x20;     apt:

&#x20;       update\_cache: yes

&#x20;       upgrade: dist



&#x20;   - name: Install required packages

&#x20;     apt:

&#x20;       name: \["vim", "htop", "wget"]

&#x20;       state: present



&#x20;   - name: Create test file

&#x20;     copy:

&#x20;       dest: /root/ansible\_test.txt

&#x20;       content: "Configured by Ansible on {{ inventory\_hostname }}"

```

<img width="1081" height="752" alt="image" src="https://github.com/user-attachments/assets/67475c38-3713-4a93-9ab7-105de58f426f" />



Run the playbook:



```bash

ansible-playbook -i inventory.ini update.yml

```



\*\*Expected output:\*\*



```

PLAY \[Update and configure servers] \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*



TASK \[Gathering Facts] \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

ok: \[172.17.0.2]

ok: \[172.17.0.3]

ok: \[172.17.0.4]

ok: \[172.17.0.5]



TASK \[Update apt packages] \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

changed: \[172.17.0.2]

changed: \[172.17.0.3]

...



TASK \[Install required packages] \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

changed: \[172.17.0.2]

...



TASK \[Create test file] \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

changed: \[172.17.0.2]

...



PLAY RECAP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

172.17.0.2  : ok=4  changed=3  unreachable=0  failed=0

172.17.0.3  : ok=4  changed=3  unreachable=0  failed=0

172.17.0.4  : ok=4  changed=3  unreachable=0  failed=0

172.17.0.5  : ok=4  changed=3  unreachable=0  failed=0

```



\---

<img width="988" height="333" alt="image" src="https://github.com/user-attachments/assets/b2e418ed-75cf-48b9-a702-0ad54070cea3" />



\### Step 8: Create Advanced Playbook (`playbook1.yml`)



```yaml

\---

\- name: Configure multiple servers

&#x20; hosts: servers

&#x20; become: yes

&#x20; tasks:



&#x20;   - name: Update apt package index

&#x20;     apt:

&#x20;       update\_cache: yes



&#x20;   - name: Install Python 3 (latest available)

&#x20;     apt:

&#x20;       name: python3

&#x20;       state: latest



&#x20;   - name: Create test file with content

&#x20;     copy:

&#x20;       dest: /root/test\_file.txt

&#x20;       content: |

&#x20;         This is a test file created by Ansible

&#x20;         Server name: {{ inventory\_hostname }}

&#x20;         Current date: {{ ansible\_date\_time.date }}



&#x20;   - name: Display system information

&#x20;     command: uname -a

&#x20;     register: uname\_output



&#x20;   - name: Show disk space

&#x20;     command: df -h

&#x20;     register: disk\_space



&#x20;   - name: Print results

&#x20;     debug:

&#x20;       msg:

&#x20;         - "System info: {{ uname\_output.stdout }}"

&#x20;         - "Disk space: {{ disk\_space.stdout\_lines }}"

```



Run it:



```bash

ansible-playbook -i inventory.ini playbook1.yml

```



\---



\### Step 9: Verify Changes



Using Ansible to read the created file across all servers:



```bash

ansible all -i inventory.ini -m command -a "cat /root/ansible\_test.txt"

```

<img width="989" height="326" alt="image" src="https://github.com/user-attachments/assets/9303194a-fdbc-4bf2-8a35-67107e611cef" />



Using Docker exec directly:



```bash

for i in {1..4}; do

&#x20;   docker exec server${i} cat /root/ansible\_test.txt

done

```

<img width="1002" height="385" alt="image" src="https://github.com/user-attachments/assets/d9ce9644-0b7e-41c7-9011-675584dde448" />



\*\*Expected output on each server:\*\*



```

Configured by Ansible on 172.17.0.2

Configured by Ansible on 172.17.0.3

Configured by Ansible on 172.17.0.4

Configured by Ansible on 172.17.0.5

```



\---





\### Step 10: Cleanup



Stop all server containers:



```bash

for i in {1..4}; do

&#x20;   docker rm -f server${i}

done

```



\---







\## Complete Workflow Summary



```

1\. Setup SSH keys

&#x20;       ↓

2\. Build ubuntu-server Docker image

&#x20;       ↓

3\. Launch 4 containers (server1–server4)

&#x20;       ↓

4\. Generate inventory.ini with container IPs

&#x20;       ↓

5\. Test connectivity (ansible -m ping)

&#x20;       ↓

6\. Run playbook (ansible-playbook)

&#x20;       ↓

7\. Verify changes

&#x20;       ↓

8\. Cleanup (docker rm -f)

```



\---



\## Key Takeaways



1\. \*\*Agentless\*\* — Ansible only needs SSH on managed nodes, no extra software to install

2\. \*\*Idempotent\*\* — running the same playbook twice produces the same result, no unintended side effects

3\. \*\*Declarative\*\* — you describe \*what\* you want (nginx installed, file present), not \*how\* to do it step by step

4\. \*\*Inventory\*\* — the single source of truth for all managed nodes, supports groups and variables

5\. \*\*Modules\*\* — 3000+ built-in modules cover everything from `apt`/`yum` to AWS/Azure/GCP resources

6\. \*\*`register` + `debug`\*\* — capture command output and print it back, useful for auditing and troubleshooting

7\. \*\*Playbooks scale\*\* — the same playbook that ran on 4 Docker containers runs identically on 400 EC2 instances



\---



\## Screenshots



>  All screenshots are stored in the `screenshots/` folder.





\## References



\- \[Ansible Official Documentation](https://docs.ansible.com/)

\- \[Ansible Tutorial — Spacelift](https://spacelift.io/blog/ansible-tutorial)

\- \[Ansible Official Website](https://www.ansible.com/)

\- \[Ansible Tower GUI](https://ansible.github.io/lightbulb/decks/intro-to-ansible-tower.html)

\- \[Ansible Tower Tutorial — GeeksforGeeks](https://www.geeksforgeeks.org/devops/ansible-tower/)



\---



\*Experiment 9 | Containerization and DevOps Lab | UPES Dehradun\*



