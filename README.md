## Ansible - Prometheus 

For complete documentation on what these roles do, here is the manual process:

https://www.walkersblog.net/article/monitoring/ 

### Getting Started

This repository contains custom Ansible roles for deploying Promethues, Alertmanager, node_exporter, Thanos (query,sidecar,store), Grafana and Loki on a HA pair of servers.

Clone the repository:

```
git clone https://github.com/walkersblog/ansible-prometheus-stack.git
```

Create and activate a Python virtual environment (RHEL8 example):

```
cd ansible-prometheus-stack
python3.9 -m venv venv
source venv/bin/activate
```

Install ansible:

```
pip install ansible
```

Alternatively (on a RHEL8 system with subscription):

```
sudo subscription-manager repos --enable ansible-2.9-for-rhel-8-x86_64-rpms
sudo dnf install ansible -y
ansible --version
```

### Inventory / Environment

The inventory assumes two servers, this was tested using two bridged Libvirt Virtual Machines with IPAddresses `192.168.1.51` & `192.168.1.52`.

To update these addresses:

```
vi inventories/lab/hosts
vi inventories/lab/group_vars/monitoring_servers
```

The playbooks will target the group `monitoring_servers`.

### Prepare monitoring servers

Each server needs a user "ansible" with sudo access, SSH to each one, create the user and amend sudoers:

```
useradd ansible
passwd ansible
usermod -aG wheel ansible
visudo

    #%wheel ALL=(ALL)       ALL
    %wheel  ALL=(ALL)       NOPASSWD: ALL
```

From the client computer where this repository was cloned, copy the public key of the user being used to execute Ansible playbooks. (Use `ssh-keygen` to generate new key pair if required)

```
ssh-copy-id ansible@192.168.1.51
ssh-copy-id ansible@192.168.1.52
```

### Storage

This example uses NFS shared storage for the two monitoring hosts to share config.

### Smoke Test

With all the preparation in place, run the smoke test playbook which targets the monitoring_servers group, which should create a file `/tmp/smoke_test.yml` with the text as defined in `inventories/lab/group_vars/monitoring_servers`.

```
cd playbooks/lab
ansible-playbook smoke_test.yml 
```

### Deploy the stack

Ensure the group_vars/monitoring_servers variables reflect your environment and that a PostgreSQL is available and accessible. (See https://www.walkersblog.net/article/postgres/ on setting up a PostgreSQL instance)

```
cd playbooks/lab
ansible-playbook deploy_all.yml 
```

### Env Vars

Secret information is set using environment varibles, for example:

```
export ALERTMANAGER_SMTP_USERNAME=user@gmail.com
export ALERTMANAGER_SMTP_PASSWORD=changeme
export ALERTMANAGER_SEND_TO_EMAIL_ADDRESS=johndoe@gmail.com
```

### Versions

Override or update the versions found in `<ROLE>/defaults/main.yml`. 
Versions and further information can be found here:

* https://prometheus.io/download/
* https://github.com/thanos-io/thanos/releases
* https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1
* https://github.com/grafana/loki/releases
