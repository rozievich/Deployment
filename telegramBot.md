# Telegram Botni Linux Serverga Deploy Qilish

## 1. Serverga kirish
Birinchi navbatda, serverga SSH orqali kiramiz:

    $ ssh username@server-ip

Loyiha uchun katalogga o'tamiz:

    $ cd /var/www

## 2. Loyiha fayllarini yuklash
Loyiha fayllarini serverga yuklash uchun quyidagi usullardan foydalanish mumkin:
- **SCP** orqali lokal kompyuterdan yuklash:

    $ scp -r mybot username@server-ip:/var/www/

- **Git** orqali yuklash:

    $ git clone https://github.com/username/mybot.git

## 3. Virtual muhit yaratish
Loyihaning muhitini izolyatsiya qilish uchun virtual muhit yaratamiz:

    $ python3 -m venv venv

Yangi virtual muhitni faollashtiramiz:

    $ source venv/bin/activate

## 4. Kerakli modullarni o‘rnatish

    $ pip install -r requirements.txt

## 5. Systemd xizmatini yaratish
Botni avtomatik ishga tushirish uchun systemd xizmat faylini yaratamiz:

    $ sudo nano /etc/systemd/system/mybot.service

Quyidagi konfiguratsiyani kiriting:

    [Unit]
    Description=Telegram Bot
    After=network.target

    [Service]
    User=<username>
    WorkingDirectory=/var/www/mybot
    Environment="PATH=/var/www/mybot/venv/bin"
    ExecStart=/var/www/mybot/venv/bin/python bot.py
    Restart=always

    [Install]
    WantedBy=multi-user.target

Faylni saqlab chiqamiz (`CTRL + X`, `Y`, `Enter`).

## 6. Systemd xizmatini ishga tushirish

    $ sudo systemctl daemon-reload
    $ sudo systemctl start mybot
    $ sudo systemctl enable mybot

Xizmat holatini tekshirish:

    $ sudo systemctl status mybot

## 7. Nginx sozlamalari (agar webhook ishlatilsa)
Agar bot webhook orqali ishlasa, Nginx teskari proksi sifatida ishlaydi. Quyidagi konfiguratsiyani yaratamiz:

    $ sudo nano /etc/nginx/sites-available/mybot

Quyidagi kodni kiriting:

    server {
        server_name <your-site-name> or <server-ip>;

        location / {
            proxy_pass http://127.0.0.1:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        listen 80;
    }

Faylni saqlang va unga symbolic link yarating:

    $ sudo ln -s /etc/nginx/sites-available/mybot /etc/nginx/sites-enabled

Nginx’ni qayta yuklang:

    $ sudo systemctl restart nginx

## 8. Loglarni tekshirish
Agar botda muammo yuzaga kelsa, loglarni tekshiring:

    $ sudo journalctl -u mybot -f
    $ sudo tail -f /var/log/nginx/error.log

## 9. Botni avtomatik qayta ishga tushirish
Agar bot xatolik tufayli to‘xtab qolsa, qayta ishga tushirish uchun systemd xizmatini yaratganmiz.
Shuningdek, quyidagi buyrug‘ni ishga tushirib tekshirish mumkin:

    $ sudo systemctl restart mybot

Shu bilan, Telegram bot serverda barqaror ishlaydi. Omad! 🚀

