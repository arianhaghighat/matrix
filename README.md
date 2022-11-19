# matrix
How to install matrix using ansible

سرور را آپدیت کنید و فایل های لازم رو نصب کنید:

<pre>apt update && apt upgrade
apt install make
apt install git
apt install ansible</pre>

در این مرحله فایل های لازم را از گیت هاب دریافت کنید
<pre> git clone https://github.com/spantaleev/matrix-docker-ansible-deploy.git </pre>

در این مرحله دستورات لازم را به ترتیب اجرا کنید. توجه کنید به جای [domain] آدرس دامنه خودتون را وارد کنید.
<pre>
cd matrix-docker-ansible-deploy/
cd inventory/host_vars/
mkdir matrix.[domain]
cd matrix.[domain]
cp ../../../examples/vars.yml ./
nano vars.yml
</pre>

در فایل باز شده ۳ متغیر را می بایست تغییر دهید.
1) matrix_domain --> domian **main domain (without sub domain)
2) generate secret --> recomeded: using (pwgen -s 64 1)
matrix_homeserver_generic_secret_key: 
matrix_postgres_connection_password:
3)email address
matrix_ssl_lets_encrypt_support_email:

4) add
matrix_nginx_proxy_base_domain_serving_enabled: true


cd ../..
cp ../examples/hosts ./
nano hosts

[matrix_servers]
matrix.<your-domain> ansible_host=<your-server's external IP address> ansible_ssh_user=root

cd 
cd matrix-docker-ansible-deploy/
make roles

ansible-playbook -i inventory/hosts setup.yml --tags=setup-all
ansible-playbook -i inventory/hosts setup.yml --tags=start
