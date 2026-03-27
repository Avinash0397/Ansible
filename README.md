# Ansible Assignment-2  
## Automated Nginx and Apache Setup with Time-Based Website Rotation (Without Roles)

This project uses two Ansible playbooks and a static inventory to automate:

- Nginx installation and log management
- Hosting multiple websites with 2-hour rotation
- Apache installation
- Nginx as a reverse proxy to Apache
- Rolling updates on servers one by one


## Playbooks Overview

| Playbook           | Purpose                               |
|--------------------|---------------------------------------|
| `nginx_setup.yml`   | Install and configure Nginx, manage log size, rotate website content |
| `apache_proxy.yml`  | Install Apache, configure Nginx as reverse proxy to Apache |


## Prerequisites

- Ansible installed on control node
- SSH access setup to all managed nodes
- Correct permissions and `sudo` privileges
- Basic DNS or `/etc/hosts` entry for `team.opstree.com`

---

## How to Run

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/ansible-assignment-2.git
cd ansible-assignment-2
```

### 2. Update Inventory

Edit the `inventory` file and add your server IPs and users.

### 3. Run Nginx Setup

```bash
ansible-playbook -i inventory nginx_setup.yml
```

![Screenshot 2025-04-22 163033](https://github.com/user-attachments/assets/e8f7ceb9-2851-4e1a-bf30-1812a6e48906)


### 4. Run Apache and Reverse Proxy Setup

```bash
ansible-playbook -i inventory apache_proxy.yml
```

![Screenshot 2025-04-28 032044](https://github.com/user-attachments/assets/f5066e3e-e92b-4ddd-a3cd-0c3b74158652)


> Both playbooks are configured to update servers one-by-one using `serial: 1`.

---

## Webview
```bash
tanya.opstree.com
```
![Screenshot 2025-04-22 171433](https://github.com/user-attachments/assets/759b1a73-5b6d-4fcd-bbea-62fd56af1e15)

```bash
heena.opstree.com
```

![Screenshot 2025-04-22 171447](https://github.com/user-attachments/assets/6e13f120-cc5a-4af9-a714-3aa5c73a1b41)
