

# AWS deploy qilish.

### Serverga kirgandan keyin.
###### Birinchi bo'lib serverni update va bizga kerakli bo'lgan packeglarni o'rnatib olishimiz kerak.

- ```ubuntu@test-server:~# sudo apt update ```
- ```ubuntu@test-server:~# sudo apt install python3-venv python3.10-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl```

### DATABASE(Malumodlar ombori)ni yaratish va configuratsiya qilish.
###### Kamandalar terminalga yozilgandan so'ng Loyxamiz uchun DATABASE (Malumodlar obmori) yaratamiz.
- ``root@test-server:~#  sudo -u postgres psql ``
- `` CREATE DATABASE projectname;``
- `` ALTER USER postgres WITH PASSWORD '1'; ``
- `` ALTER ROLE postgres SET client_encoding TO 'utf8';``
- `` ALTER ROLE postgres SET default_transaction_isolation TO 'read committed';``
- `` ALTER ROLE postgres SET timezone TO 'UTC'; ``
- `` GRANT ALL PRIVILEGES ON DATABASE projectname TO postgres ;``
- `` \q `` chiqish uchun.
### Projectni githubdan clone qilishimiz kerak.
######  repositoryni projectni deploy qilish uchun shunchaki projectning repositoryining urlni olish yetarli buning uchun.
- ``root@test-server:~# git clone git@github.com:username/repository-name.git``

-----------
# Loyhani o'rnatish.
###### Bunign uchun biz birnichi bo'lib loyha turgan file ga kirib olishimiz kerak.
 - `` root@test-server:~# cd project/ `` project/ ning o'rniga sizda github repositoryning nomi bo'ladi.
 - `` root@test-server:~/project# python3 -m venv venv `` virtual enviroment(muhit) ni yaratib olamiz.
 - `` root@test-server:~/project# source venv/bin/activate `` virtual enviroment(muhit) ni active(faol)ashitirib olamiz.
 - `` (venv) root@test-server:~/project# pip3 install -r requirements.txt `` virtual enviroment(muhit) ni ichiga talab qilinga package(kutibxona)larni o'rnatib olishimiz kerak.
 - `` (venv) root@test-server:~/project# pip3 install gunicorn psycopg2-binary `` Gunicorn va psycopg2-binary ni o'rnatishimiz kerak.
 
# Loyhani sozlash.
###### biz loyhani qaysi url(manzil) da ishlashi va Production(Ishlab chiqarish) da Debug(Nosozliklarni tuzatish) kerak yani False(Yolg'on) ga o'zgaritishimiz kerak
- ``(venv) root@test-server:~/project# sudo nano root/settings.py``
- Debug = True
- ALLOWED_HOST da tog'ilash
- saqlash uchun va chiqib ketish `CTRL ` + `O` va `CTRL` + `X`
#### Loyhani DATABASE(malumodlar ombori) bilan bog'lanish uchun: 
- ``(venv) root@test-server:~/project# python3 manage.py makemigrations``
- ``(venv) root@test-server:~/project# python3 manage.py migrate``
#### DATABASE(malumodlar ombori) ga Admin panel orqali kirish uchun superuser yaratish:
- ``(venv) root@test-server:~/project# python3 manage.py createsuperuser``
#### Gunicorn loyhamiz uchun ishlayotganini tekshirishimiz kerak.
- ``(venv) root@test-server:~/project# gunicorn --bind 0.0.0.0:8000 loyhamiz_nomi.wsgi``
- Agar test muffaciyatli ishlasa ``CTRL`` + ``X`` orqali testni toxtatamiz.- 
- ``(venv) root@test-server:~/project# deactivate``
- --------------------------
#### Socket file yozishimiz shart Gunicorn uchun
###### Yani biz Gunicornni django loyhamiz bilan test qilib ko'rdik endi biz django loyhamiz server bilan ishlashi va toxtashi uchun yana ishonchli usuldan foydalanishimiz kerak yani systemd va socket dan.

```angular2html
sudo nano /etc/systemd/system/gunicorn.socket
```
filni ochganimizdan so'ng shu code o'zgarishsiz yozilishi kerak socket filega.
```angular2html
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

###### Biz endi systemd service yozishimiz kerak Gunicorn uchun 
```angular2html
sudo nano /etc/systemd/system/gunicorn.service
```
```angular2html
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/root/myprojectdir
ExecStart=/root//myprojectdir/venv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          root.wsgi:application

[Install]
WantedBy=multi-user.target
```
###### Enable qilishimiz shart /run/gunicorn.sock file yaratish uchun
```angular2html
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

###### Gunicornning status(holat)ni tekshirib ko'rsak bo'ladi
```angular2html
sudo systemctl status gunicorn.socket
```

Natija: 
```angular2html
● gunicorn.socket - gunicorn socket
     Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: enabled)
     Active: active (listening) since Mon 2022-04-18 17:53:25 UTC; 5s ago
   Triggers: ● gunicorn.service
     Listen: /run/gunicorn.sock (Stream)
     CGroup: /system.slice/gunicorn.socket

Apr 18 17:53:25 django systemd[1]: Listening on gunicorn socket.
```

file /run/gunicorn.sockni yaratganini tekshirib ko'rsak bo'ladi.
```angular2html
file /run/gunicorn.sock
```
Natija: 
```
/run/gunicorn.sock: socket
```

Gunicornning Holatini tekshirishimiz kerak: 
```angular2html
sudo systemctl status gunicorn
```
Natija:
```angular2html
● gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 17:54:49 UTC; 5s ago
TriggeredBy: ● gunicorn.socket
   Main PID: 102674 (gunicorn)
      Tasks: 4 (limit: 4665)
     Memory: 94.2M
        CPU: 885ms
     CGroup: /system.slice/gunicorn.service
             ├─102674 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             ├─102675 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             ├─102676 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             └─102677 /root/myprojectdir/venv/bin/python3 /root/myprojectdir/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application

Sep 18 17:54:49 django systemd[1]: Started gunicorn daemon.
Sep 15 17:10 n:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Starting gunicorn 20.1.0
Sep 15 17:10:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Listening at: unix:/run/gunicorn.sock (102674)
Sep 15 17:10:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Using worker: sync
Sep 15 17:10:49 django gunicorn[102675]: [2022-04-18 17:54:49 +0000] [102675] [INFO] Booting worker with pid: 102675
Sep 15 17:10:49 django gunicorn[102676]: [2022-04-18 17:54:49 +0000] [102676] [INFO] Booting worker with pid: 102676
Sep 15 17:10:50 django gunicorn[102677]: [2022-04-18 17:54:50 +0000] [102677] [INFO] Booting worker with pid: 102677
Sep 15 17:10:50 django gunicorn[102675]:  - - [18/Apr/2022:17:54:50 +0000] "GET / HTTP/1.1" 200 10697 "-" "curl/7.81.0"
```

Gunicron ni qayta ishlastishimiz kerak:
```angular2html
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

---------------------------------
## Ngnix file ni Gunicron filega configuratsiya qilishimiz kerak.
Ngnixda yangi server yaratib olishimz kerak.
```angular2html
sudo nano /etc/nginx/sites-available/myproject
```
Va Ngnix ning yangi yaralgan ``myproject`` file ga
```angular2html
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /var/www ;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```

Va Ngnixni enable qilishimiz mumkin `sites-enabled` filga ga

```angular2html
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```
Ngnix ni test qilib olamiz
```angular2html
sudo nginx -t
```

Ngnixni restart qilamiz
```angular2html
sudo systemctl restart nginx
```
Ngnix serverimizga 80 porta ishlashga ruxsat berganimizdan keyin 8000 port ni olib tashlashimiz kerak bo'ladi.
```angular2html
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
```
Va 22 chi port uchun ham ruhsat berish kerak
```
sudo ufw allow 22
```

### Loyhamiz domain yoki ip da ishlab turibdi agar ishlmagan bo'lsa error log ni ko'rishimiz va docs ga qaytib errorni tog'rilashimz mumkin.
```angular2html
sudo tail -F /var/log/nginx/error.log
```
-----------------
### HTTPS ga otqizish
```
sudo apt-get install certbot python3-certbot-nginx
```
Va example.com orniga ozizni domenizni qoyasiz
```
sudo certbot --nginx -d example.com -d www.example.com
```