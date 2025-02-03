# Django Loyihasini Serverga Deploy Qilish

## 1. Serverga kirish

Birinchi navbatda, serverga SSH orqali kiramiz:

```
$ ssh username@server-ip
```

Loyiha uchun katalogga o'tamiz:

```
$ cd /var/www
```

## 2. Loyiha fayllarini yuklash

Loyiha fayllarini serverga yuklash uchun quyidagi usullardan foydalanish mumkin:

- **SCP** orqali lokal kompyuterdan yuklash:

  \$ scp -r mydjangoapp username\@server-ip\:/var/www/

- **Git** orqali yuklash:

  \$ git clone [https://github.com/username/mydjangoapp.git](https://github.com/username/mydjangoapp.git)

## 3. Virtual muhit yaratish

Loyihaning muhitini izolyatsiya qilish uchun virtual muhit yaratamiz:

```
$ python3 -m venv venv
```

Yangi virtual muhitni faollashtiramiz:

```
$ source venv/bin/activate
```

## 4. Kerakli modullarni o‘rnatish

```
$ pip install -r requirements.txt
```

## 5. Django migratsiyalarni bajarish va statik fayllarni tayyorlash

```
$ python manage.py migrate
$ python manage.py collectstatic --noinput
```

## 6. Systemd xizmatini yaratish

Django loyihasini avtomatik ishga tushirish uchun systemd xizmat faylini yaratamiz:

```
$ sudo nano /etc/systemd/system/mydjangoapp.service
```

Quyidagi konfiguratsiyani kiriting:

```
[Unit]
Description=Django Application
After=network.target

[Service]
User=<username>
WorkingDirectory=/var/www/mydjangoapp
Environment="PATH=/var/www/mydjangoapp/venv/bin"
ExecStart=/var/www/mydjangoapp/venv/bin/gunicorn --workers 3 --bind unix:/var/www/mydjangoapp/mydjangoapp.sock mydjangoapp.wsgi:application
Restart=always

[Install]
WantedBy=multi-user.target
```

Faylni saqlab chiqamiz (`CTRL + X`, `Y`, `Enter`).

## 7. Systemd xizmatini ishga tushirish

```
$ sudo systemctl daemon-reload
$ sudo systemctl start mydjangoapp
$ sudo systemctl enable mydjangoapp
```

Xizmat holatini tekshirish:

```
$ sudo systemctl status mydjangoapp
```

## 8. Nginx sozlamalari

Django loyihasini internetga ulash uchun Nginx teskari proksi sifatida ishlaydi. Quyidagi konfiguratsiyani yaratamiz:

```
$ sudo nano /etc/nginx/sites-available/mydjangoapp
```

Quyidagi kodni kiriting:

```
server {
    server_name <your-site-name> or <server-ip>;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/mydjangoapp/mydjangoapp.sock;
    }

    listen 80;
}
```

Faylni saqlang va unga symbolic link yarating:

```
$ sudo ln -s /etc/nginx/sites-available/mydjangoapp /etc/nginx/sites-enabled
```

Nginx’ni qayta yuklang:

```
$ sudo systemctl restart nginx
```

## 9. Loglarni tekshirish

Agar muammo yuzaga kelsa, loglarni tekshiring:

```
$ sudo journalctl -u mydjangoapp -f
$ sudo tail -f /var/log/nginx/error.log
```

## 10. Django loyihasini avtomatik qayta ishga tushirish

Agar loyiha xatolik tufayli to‘xtab qolsa, qayta ishga tushirish uchun systemd xizmatini yaratganmiz.
Shuningdek, quyidagi buyrug‘ni ishga tushirib tekshirish mumkin:

```
$ sudo systemctl restart mydjangoapp
```

Shu bilan, Django loyihasi serverda barqaror ishlaydi. Omad! 🚀

