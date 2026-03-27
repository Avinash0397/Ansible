# Assignment-5
-Ansible Role for PostgreSQL Installation

**Objective**

This assignment focuses on creating an **Ansible Role** to install **PostgreSQL** on multiple servers.  
The role supports:
- Version-specific installation
- OS-independence (CentOS/Ubuntu)
- Jinja2 templating for configurations
- Handlers for service management (start/restart)
- Variables for flexible configurations

**Create Inventory File**
```bash
vi inventory.ini
```

**create playbook**
```bash
vi playbook.yml
```

![Screenshot 2025-04-29 005438](https://github.com/user-attachments/assets/42583adc-429b-426b-9fab-568c3b654b41)


**create role**
```bash
ansible-playbook -i inventory.ini playbook.yml
```

![Screenshot 2025-04-28 232810](https://github.com/user-attachments/assets/3cb46b4d-e159-4bb3-8b69-f6567f84cd00)


```bash
tree
```

![Screenshot 2025-04-28 232825](https://github.com/user-attachments/assets/d5a33564-3f6b-4691-b026-d5854dc9e177)

**Run ansible playbook**

```bash
ansible-playbook -i inventory.ini playbook.yml
```

![Screenshot 2025-04-29 001020](https://github.com/user-attachments/assets/8db18c29-8a78-4235-8208-e5550a6d3f14)


**check tool install on remote server 1**

```bash
sudo systemctl status postgresql
```

```bash
ps -aux | grep postgresql
```

![Screenshot 2025-04-29 001713](https://github.com/user-attachments/assets/c8cabe5f-7de7-454b-9ed4-03dfb2011942)


**check tool install on remote server 2**

```bash
sudo systemctl status postgresql
```

```bash
ps -aux | grep postgresql
```


![Screenshot 2025-04-29 001954](https://github.com/user-attachments/assets/bdd764b3-e07d-42e8-a137-f2e5ec97f2ea)







  
