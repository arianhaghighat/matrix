
# Matrix Self-Hosted Server Setup (Using Ansible)

This guide explains how to install and run a Matrix server using [matrix-docker-ansible-deploy](https://github.com/spantaleev/matrix-docker-ansible-deploy) on your own VPS. It includes optional features like a Synapse admin panel and a Telegram bridge.

> 🖥️ **Note:** These instructions assume you're running commands *directly* on your server, which is why the `ansible_connection=local` setting is used.

---

## 🧱 Prerequisites

Start by updating your server and installing required packages:

```bash
apt update && apt upgrade -y
apt install make git ansible -y
```

Clone the official deployment repository:

```bash
git clone https://github.com/spantaleev/matrix-docker-ansible-deploy.git
```

---

## 📂 Step 1: Set Up Your Domain Configuration

Navigate to the inventory folder and create a directory for your domain:

```bash
cd matrix-docker-ansible-deploy/inventory/host_vars/
mkdir matrix.[your-domain]
cd matrix.[your-domain]
cp ../../../examples/vars.yml ./
nano vars.yml
```

In the `vars.yml` file, configure the following variables:

1. **Set your main domain** (without subdomain):

```yaml
matrix_domain: "[your-domain]"  # e.g. example.com
```

2. **Generate secure secrets** using:

```bash
pwgen -s 64 1
```

Then set these values in `vars.yml`:

```yaml
matrix_homeserver_generic_secret_key: "<secure-random-string>"
matrix_postgres_connection_password: "<secure-random-string>"
```

3. **Set your SSL email**:

```yaml
matrix_ssl_lets_encrypt_support_email: "your-email@example.com"
```

4. **Add the following extra configuration** to the bottom of the file:

```yaml
matrix_nginx_proxy_base_domain_serving_enabled: true
matrix_static_files_container_labels_base_domain_enabled: true
```

---

## 🗂️ Step 2: Configure Ansible Hosts

Navigate back and set up the `hosts` file:

```bash
cd ../..
cp ../examples/hosts ./
nano hosts
```

Replace the last line with the following, customized for your domain and IP:

```ini
matrix.[your-domain] ansible_host=<your-server-ip> ansible_connection=local ansible_become=true
```

> Replace `<your-server-ip>` with the actual IP address of your server.

---

## ⚙️ Step 3: Install Ansible Roles

Run the following commands to install the necessary Ansible roles:

```bash
cd
cd matrix-docker-ansible-deploy/
make roles
```

---

## 🚀 Step 4: Deploy Matrix Server

Run the full setup process (this will take a few minutes):

```bash
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all
```

Start the server:

```bash
ansible-playbook -i inventory/hosts setup.yml --tags=start
```

---

## 👤 Step 5: Register a New User

To create a new user, run:

```bash
ansible-playbook -i inventory/hosts setup.yml --extra-vars='username=<your-username> password=<your-password> admin=[yes|no]' --tags=register-user
```

Example:

```bash
ansible-playbook -i inventory/hosts setup.yml --extra-vars='username=arian password=secure123 admin=yes' --tags=register-user
```

You can now log in via [Element Web](https://app.element.io) using:

```
@<your-username>:<your-domain>
```

---

## 🔐 Optional: Enable Synapse Admin Panel

To enable the Synapse admin panel, edit your domain configuration again:

```bash
nano inventory/host_vars/matrix.[your-domain]/vars.yml
```

Add this line at the end:

```yaml
matrix_synapse_admin_enabled: true
```

Then apply the changes:

```bash
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all
ansible-playbook -i inventory/hosts setup.yml --tags=start
```

The panel will be available at:

```
https://matrix.[your-domain]/synapse-admin/
```

---

## 💬 Optional: Enable Telegram Bridge

To bridge Matrix with Telegram, enable and configure Mautrix-Telegram.

Edit `vars.yml`:

```yaml
matrix_mautrix_telegram_enabled: true
matrix_mautrix_telegram_api_id: "<your-telegram-app-id>"
matrix_mautrix_telegram_api_hash: "<your-telegram-api-hash>"
matrix_mautrix_telegram_config:
  homeserver:
    address: "http://matrix-synapse:8008"
    domain: "[your-domain]"
```

Then deploy the bridge:

```bash
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all
ansible-playbook -i inventory/hosts setup.yml --tags=start
```

> For full setup instructions of the Telegram bridge, refer to the [official bridge docs](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-playbook-bridge-mautrix-telegram.md).

---

## 📝 Notes

- Make sure your domain has DNS records set up correctly (e.g., `matrix.[your-domain]` pointing to your VPS IP).
- If running behind a firewall or cloud provider, open the required ports (typically 80 and 443).
- Use tools like [Element Web](https://app.element.io) or [Hydrogen](https://hydrogen.element.io/) to log in and chat.
