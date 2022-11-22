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
cd matrix-docker-ansible-deploy/cd inventory/host_vars/
mkdir matrix.[domain]
cd matrix.[domain]
cp ../../../examples/vars.yml ./
nano vars.yml
</pre>

در فایل باز شده ۳ متغیر را می بایست تغییر دهید.
1) جلوی matrix_domain آدرس دامنه خود را بدون ساب دامنه وارد کنید (example.com)
2)  جلوی دو متغییر زیر یک پسورد قوی وارد کنید. برای ساخت پسورد در لینوکس می توانید از <code> pwgen -s 64 1 </code> استفاده کنید. 
<pre> matrix_homeserver_generic_secret_key: 
matrix_postgres_connection_password: </pre>


3) جلوی <code> matrix_ssl_lets_encrypt_support_email: </code> ایمیل خود را برای گواهینامه ssl وارد کنید.
4) در پایین فایل عبارت زیر را اضافه کنید:
<pre> matrix_nginx_proxy_base_domain_serving_enabled: true </pre>

فایل را ببندید و کد های زیر را اجرا کنید:
<pre> cd ../..
cp ../examples/hosts ./
nano hosts </pre>

در فایل باز شده خط آخر را ویرایش کرده و آدرس دامنه و ip سرور خودتون را وارد کنید. 
اگر مستقیما دستورات را داخل سرور اجرا میکنید عبارت <code> ansible_connection=local </code> را نیز در اخر خط اضافه کنید. 

حالا دستورات زیر را وارد کنید
<pre> cd 
cd matrix-docker-ansible-deploy/
make roles </pre>

تنظیمات تمام است! حالا نوبت نصب اصلی برنامه است ابتدا دستور زیر را وارد کنید. توجه کنید این مرحله چندین دقیقه طول می کشد

<pre> ansible-playbook -i inventory/hosts setup.yml --tags=setup-all </pre>
سپس با این دستور فایل ها رو اجرا کنید
<pre> ansible-playbook -i inventory/hosts setup.yml --tags=start </pre>

به طور پیش فرض کاربر جدید امکان عضو شدن به سرور شما رو ندارد. با جرای دستور زیر می توانید کاربر جدید اضافه کنید. username و password را در دستور زیر ویرایش کنید و انتخاب کنید کاربر دسترسی ادمین داشته باشد یا خیر

<pre> ansible-playbook -i inventory/hosts setup.yml --extra-vars='username=[your-username] password=[your-password] admin=[yes|no]' --tags=register-user </pre>

