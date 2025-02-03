<h1>FastAPI Loyihani Serverga deploy qilish</h1>
<h3>Qadamlar</h3>
<b>1. Serverga kirish:</b>
<br>
<i>Birinchi navbatda, serverga SSH orqali kirib, loyiha uchun foydalanish kerak bo'lgan katalogga o'ting. Misol uchun:
</i>
<br>

    $ cd /var/www

<b>2. Loyiha fayllarini yuklash:</b>
<br>
<i>Loyiha fayllarini serverga ko'chirish uchun, joylashgan fayl katalogiga o'tib, loyihangizning fayllarini yuklab oling. Agar fayllar kattaroq bo'lsa, <i style="color: green;">scp</i> yoki boshqa usullar bilan fayllarni ko'chirishingiz mumkin.</i>

<b>3. Loyiha konfiguratsiyasini sozlash:</b>
<br>
<i>Loyiha fayllarini serverga ko'chirgandan so'ng, loyiha konfiguratsiyasini sozlash lozim. Bu maqsadda, kerakli konfiguratsiya fayllarini o'zgartiring. Agar siz README.md faylini ochishga urinsangiz, faylda foydalanuvchining o'zgartirishlarni qanday bajarishini ko'rsatilgan bo'lishi mumkin.</i>

<b>4. Loyiha uchun virtual muhitni o'rnatish:</b><br>
<i>FastAPI loyihalari uchun o'zgartirilgan virtual muhit o'rnatish oson bo'lishi mumkin. Virtual muhit o'rnatish uchun, quyidagi kommandani ishga tushiring:</i>

    $ python3 -m venv venv

Bu, yangi virtual muhitni <b>venv</b> nomi bilan yaratadi.

<b>5. Virtual muhitni faollashtirish:</b><br>
<i>Yangi virtual muhitni faollashtirish uchun quyidagi kommandani ishga tushiring:</i>

    $ source venv/bin/activate

Virtual muhit faollashtirilgandan so'ng, terminalda muhit nomi (misol uchun (venv)) paydo bo'ladi.

<b>6. Qo'shimcha modullarni o'rnatish:</b><br>
<i>FastAPI loyihalari uchun kerakli qo'shimcha modullarni o'rnatish uchun:</i>

    $ pip install -r requirements.txt
    $ pip install gunicorn uvicorn


Bu, <b>'requirements.txt'</b> fayli orqali talab qilingan modullarni o'rnatadi va shu bilan bir qatorda loyiha serverda ishlab turishi uchun <b>gunicorn uvicorn</b> modullarini o'rnatadi.

<b>7. Systemd xizmat faylini yaratish:</b><br>
<i>Systemd xizmat faylini yaratish uchun, quyidagi kommandani ishga tushiring:</i>

    $ sudo nano /etc/systemd/system/myapp.service

<i>Bu kommanda, <b>myapp.service</b> nomli yangi faylni <b>nano</b> muhiti orqali ochadi, lekin siz boshqa muhitlarni ham ishlatishingiz mumkin.</i>

<b>8. Gunicorn va systemd sozlamalarini kiritish:</b><br>
<i><b>myapp.service</b> faylini ochib, quyidagi 
ma'lumotlarni kiritishni unutmang:</i>

    [Unit]
    Description=Gunicorn instance to serve MyApp
    After=network.target

    [Service]
    User=<username>
    Group=www-data
    WorkingDirectory=/var/www/myapp/src
    Environment="PATH=/var/www/myapp/venv/bin"
    ExecStart=/var/www/myapp/venv/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app

    [Install]
    WantedBy=multi-user.target

<i>Yuqorida, <b>username</b> o'rniga sizning foydalanuvchi nomingizni yozing.</i>

<b>9. Systemd xizmatini qo'shish:</b><br>
<i>Systemd xizmatini qo'shish uchun quyidagi kommandani ishga tushiring:</i>

    $ sudo systemctl daemon-reload
    $ sudo systemctl start myapp

<i>Bu kommandalar, <b>systemd</b> xizmatini yangilab qayta yuklash va <b>myapp</b> xizmatini ishga tushirish uchun ishlatiladi.</i>

<b>10. Avtomatik ishga tushish:</b><br>
<i>Agar loyihangizni tizim boshlang'ich ishga tushishda avtomatik ravishda ishga tushirishni xohlaysiz, quyidagi kommandani ishga tushiring:</i>

    $ sudo systemctl enable myapp

<i>Bu, tizim boshida <b>myapp</b> xizmatini avtomatik ravishda ishga tushirish uchun sozlashni amalga oshiradi.</i>

<i>Sizning loyihangiz <b>systemd</b> xizmati orqali avtomatik ravishda ishga tushiriladi. Agar qandaydir muammo yuzaga kelsa, <b>journalctl</b> kommandasini ishlatib, xizmatning loglarini tekshirishingiz mumkin:</i>

    $ sudo journalctl -u myapp.service

<b>11. Nginx Konfiguratsiyasini Sozlash:</b><br>
<i>Nginx serverining konfiguratsiya faylini oching. Bu fayl katalogi quyidagicha bo'lishi mumkin:</i>

    $ sudo nano /etc/nginx/sites-available/myapp

<i>Ochilgan Faylga quidagi ma'lumotlarni qo'shing:</i>

    server {
        server_name <your-site-name> or <server-public-ip-address>;

        location / {
            include proxy_params;
            proxy_pass http://127.0.0.1:8000;
        }

        listen 80;  # Agar HTTP portdan foydalanmoqchi bo'lsangiz
        # listen 443;  # Agar HTTPS portdan foydalanmoqchi bo'lsangiz

        # SSL sozlamalari (agar HTTPS foydalanmoqchi bo'lsangiz)
        # ssl_certificate /path/to/your/certificate.crt;
        # ssl_certificate_key /path/to/your/private_key.key;
        # ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        # ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';
        # ssl_prefer_server_ciphers on;
        # ssl_session_timeout 5m;
        # ssl_session_cache shared:SSL:10m;
    }


<i>Yuqoridagi kodni o'zgartirib, <b>your-site-name</b> yoki <b>server-public-ip-address</b> ni loyihangizning ma'lumotlariga moslashtiring. Agar <b>HTTPS</b> orqali ulanmoqchi bo'lsangiz, shuningdek <b>SSL</b> sozlamalarini ham o'rnating. Keyin, barcha o'zgartirishlarni saqlang va faylni yopib chiqing.</i>

<b>12. Nginx sites-enabled faylini yaratish:</b><br>
<i>Mana shu yerga shunday ssilka yaratamiz:</i>

    $ sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled

<b>13. Nginx-ni qayta yuklash va xizmatni qayta ishga tushirish:</b><br>
<i>O'zgarishlar amalda ishga tushishi uchun <b>nginx</b> ni qayta ishga tushirish kerak bo'ladi:</i>

    $ sudo systemctl restart nginx

<b>14. NGINX to'g'ri ishlayotganligi tekshirish:</b><br>
<i>Endi nginx ishlayotganiga ishonch hosil qilish uchun quidagi komandani teramiz: </i>

    $ sudo nginx -t


<b>15. NGINX server loglarini tekshirish:</b></br>
<i>Endi, Nginx serveri loyihangizni 8000-portiga tushirib, uning orqasidan barcha so'rovlarini FastAPI loyihangizga yo'naltiradi. Nginx serverining loglarini tekshirish uchun quyidagi kommandani ishlatishingiz mumkin:</i>

    $ sudo tail -f /var/log/nginx/error.log

<h3>Umid qilamiz, bu qadamlar sizning FastAPI loyihangizni Nginx orqali serverda muvaffaqiyatli ishga tushirish uchun yordam bera oladi.</h3>

<h1 style="text-align: center;">E'tiboringiz uchun rahmat 😊</h1>
