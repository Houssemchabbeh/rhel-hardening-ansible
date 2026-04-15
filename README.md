
# RHEL System Hardening with Ansible

A focused Ansible project demonstrating core **RHCE-level automation skills**:
- **User management** with **Ansible Vault** for secure credential handling.
- **SELinux** enforcement using official **RHEL System Roles**.
- **Firewalld** configuration strictly allowing HTTP/HTTPS.
- **Apache httpd** deployment featuring **handlers** and **Jinja2 templates**.

> 🧠 This is a ** mini project** designed to showcase Linux automation and security hardening in a clean, reusable, and professional way.

---

##  Project Structure

```text
rhel-hardening-ansible/
├── ansible.cfg
├── inventory
├── user_list.yml
├── vault.yml
├── users.yml
├── selinux.yml
├── site.yml
├── roles/
│   └── httpd/
│       ├── tasks/main.yml
│       ├── handlers/main.yml
│       ├── templates/index.html.j2
│       └── vars/main.yml
```

---

## ⚙️ Prerequisites & Setup

Ensure the following automation content is installed on your **Control Node** before execution:


### - RHEL System Roles
Required for official SELinux hardening. Install via DNF:
```bash
sudo dnf install rhel-system-roles
```
### - Required Collections
Required for `firewalld` management and `uri` verification:
```bash
ansible-galaxy collection install ansible.posix community.general
```

### - Role Path Configuration
Ensure your `ansible.cfg` is configured to recognize the system-installed roles:

---

##  1. User Management (Ansible Vault)

**Files used:**
- `user_list.yml`: Defines user attributes (UIDs, password expiry, and job roles).
- `vault.yml`: Encrypted passwords (`pass_developer`, `pass_manager`).
- `users.yml`: Main playbook orchestrating user lifecycles.

**User Data Example (`user_list.yml`):**
```yaml
users:
  - name: jack
    uid: 3333
    password_expire_days: 20
    job: manager
  - name: adam
    uid: 3334
    password_expire_days: 15
    job: developer
```

Users are created conditionally based on inventory groups:
| User | Group | Server Group |
| :--- | :--- | :--- |
| **adam** | opsdev | `[dev]` |
| **jack** | opsmgr | `[prod]` |

**Run the playbook:**
```bash
ansible-playbook -i inventory users.yml --vault-password-file=secret.txt
```

---

##  2. SELinux Enforcement

**Playbook:** `selinux.yml`
This project utilizes **RHEL System Roles** to ensure SELinux is consistently enforced across the infrastructure in `targeted` mode.

**Run:**
```bash
ansible-playbook -i inventory selinux.yml
```

---

##  3. Firewalld (HTTP/HTTPS)

Integrated within the **httpd role**. It ensures that the `firewalld` service is running and opens `http` and `https` ports **permanently** and **immediately** to allow web traffic.

---

##  4. Apache httpd with Handler & Template

**Role:** `roles/httpd/`
- Installs `httpd` and `firewalld` packages.
- Ensures services are started and enabled on boot.
- Deploys a customized `index.html` via **Jinja2 templates**.
- **Handlers:** Restarts the `httpd` service only when the template configuration changes, ensuring zero unnecessary downtime.

**Run the role:**
```bash
ansible-playbook -i inventory site.yml
```

---

##  Verification (Ad-Hoc Ansible Commands)

After running the playbooks, verify the system state across all managed nodes using these commands. This confirms the "Target State" was achieved:

```bash
# 1. Check SELinux status (Should be Enforcing)
ansible all -m command -a "getenforce"

# 2. Verify Firewall services (Should show http, https)
ansible all -m command -a "firewall-cmd --list-services"

# 3. Verify Users exist with correct UIDs
ansible all -m command -a "id adam"
ansible all -m command -a "id jack"

# 4. Check User Password Expiry configuration
ansible all -m command -a "chage -l adam"

```

---

##  RHCE Skills Demonstrated

| Skill | Implementation |
| :--- | :--- |
| **Ansible Vault** | Encrypted sensitive user passwords |
| **Conditionals** | `when: inventory_hostname in groups.dev` |
| **Handlers** | Triggered `httpd` restart on template change |
| **Templates** | Dynamic `index.html.j2` using Jinja2 |
| **RHEL System Roles** | SELinux enforcement (`rhel-system-roles.selinux`) |
| **Firewalld** | Managed via `ansible.posix.firewalld` module |
| **Account Security** | Automated password aging (`password_expire_days`) |
| **Ad-Hoc Validation** | Real-time state verification across nodes |

---

## 📌 Author
**Houssem Ben Chabbeh**
 *Junior aspiring Cloud & Infrastructure Security Engineer | RHCE | AWS SAA*

---
*🧩 This project is part of a learning journey—it reflects hands-on automation skills and a commitment to secure infrastructure management.*
```

