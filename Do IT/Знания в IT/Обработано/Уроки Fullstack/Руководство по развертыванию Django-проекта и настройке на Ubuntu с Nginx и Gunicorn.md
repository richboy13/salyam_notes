---
title: "Руководство по развертыванию Django-проекта и настройке на Ubuntu с Nginx и Gunicorn"
created: 2025-08-23
modified: 2025-08-23
---

# 📖 Руководство по развертыванию Django-проекта и настройке на Ubuntu с Nginx и Gunicorn

## 📋 Описание
Вот полная инструкция по [[развертывание|развертыванию]] [[Django]]-проекта и настройке на[[ Ubuntu]] с [[Nginx]] и [[Gunicorn]].

## 🎯 Цели урока
<!-- Что изучим -->

## 📚 Материалы
<!-- Что понадобится -->

## 📝 Основные темы

### Полная инструкция по развертыванию

1. **Подготовка проекта на сервере:**
   ```bash
   # Создаем директорию для проекта
   sudo mkdir -p /var/www/SITES/cargo4you
   
   # Устанавливаем права
   sudo chown -R nv_master:www-data /var/www/SITES/cargo4you
   sudo chmod -R 755 /var/www/SITES/cargo4you
   
   # Копируем файлы проекта в директорию
   # (предполагается, что файлы уже скопированы в /var/www/SITES/cargo4you)
   ```

2. **Настройка виртуального окружения:**
   ```bash
   cd /var/www/SITES/cargo4you
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ```

3. **Настройка конфигурации Django:**
   - Создайте файл `config.ini`:
     ```python
     [django]
     SECRET_KEY = ваш_секретный_ключ
     LANGUAGE_CODE = ru-ru
     TIME_ZONE = Asia/Almaty
     
     [telegram]
     TELEGRAM_BOT_TOKEN = ваш_токен_бота
     MAIN_ADMIN_ID = ваш_id_админа
     ```

3. **Настройка Gunicorn:**
   - Создайте файл `/etc/systemd/system/gunicorn.service`:
     ```python
     [Unit]
     Description=gunicorn daemon
     After=network.target

     [Service]
     User=nv_master
     Group=www-data
     WorkingDirectory=/var/www/SITES/cargo4you
     ExecStart=/var/www/SITES/cargo4you/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/var/www/SITES/cargo4you/gunicorn.sock core.wsgi:application

     [Install]
     WantedBy=multi-user.target
     ```
   - Активируйте и запустите сервис:
     ```bash
     sudo systemctl daemon-reload
     sudo systemctl start gunicorn
     sudo systemctl enable gunicorn
     ```

5. **Настройка Nginx:**
   - Создайте файл `/etc/nginx/sites-available/cargo4you`:
     ```nginx
     server {
         server_name cargo4you.kz;

         location = /favicon.ico { access_log off; log_not_found off; }

         location /static/ {
             alias /var/www/SITES/cargo4you/staticfiles/;
         }

         location /media/ {
             root /var/www/SITES/cargo4you;
         }

         location / {
             include proxy_params;
             proxy_pass http://unix:/var/www/SITES/cargo4you/gunicorn.sock;
         }
     }
     ```
   - Создайте символическую ссылку:
     ```bash
     sudo ln -s /etc/nginx/sites-available/cargo4you /etc/nginx/sites-enabled/
     ```
   - Проверьте конфигурацию:
     ```bash
     sudo nginx -t
     ```

6. **Настройка SSL с Certbot:**
   ```bash
   # Установка Certbot
   sudo apt install certbot python3-certbot-nginx
   
   # Получение SSL-сертификата
   sudo certbot --nginx -d cargo4you.kz
   ```
   - Certbot автоматически модифицирует конфигурацию Nginx, добавив SSL-настройки

7. **Настройка Django проекта:**
   ```bash
   cd /var/www/SITES/cargo4you
   source venv/bin/activate
   
   # Сбор статических файлов
   python manage.py collectstatic
   
   # Миграции базы данных
   python manage.py migrate
   ```

8. **Проверка и перезапуск сервисов:**
   ```bash
   # Перезапуск Gunicorn
   sudo systemctl restart gunicorn
   
   # Перезагрузка Nginx
   sudo systemctl reload nginx
   ```

9. **Проверка статуса:**
   ```bash
   # Проверка статуса Gunicorn
   sudo systemctl status gunicorn
   
   # Проверка статуса Nginx
   sudo systemctl status nginx
   ```

10. **Полезные команды для отладки:**
    ```bash
    # Просмотр логов Gunicorn
    sudo journalctl -u gunicorn
    
    # Просмотр логов Nginx
    sudo tail -f /var/log/nginx/error.log
    sudo tail -f /var/log/nginx/access.log
    ```

Важные замечания:
1. Убедитесь, что в `settings.py` Django проекта:
   - `DEBUG = False`
   - `ALLOWED_HOSTS` содержит ваш домен
   - `STATIC_ROOT` указывает на `/var/www/SITES/cargo4you/staticfiles/`

2. Проверьте права доступа:
   ```bash
   sudo chown -R nv_master:www-data /var/www/SITES/cargo4you
   sudo chmod -R 755 /var/www/SITES/cargo4you
   sudo chmod 777 /var/www/SITES/cargo4you/gunicorn.sock
   ```

3. Если возникают проблемы с правами доступа к сокету Gunicorn:
   ```bash
   sudo chmod 777 /var/www/SITES/cargo4you/gunicorn.sock
   ```

4. Для обновления сайта в будущем:
   ```bash
   cd /var/www/SITES/cargo4you
   source venv/bin/activate
   git pull  # если используете git
   pip install -r requirements.txt
   python manage.py migrate
   python manage.py collectstatic
   sudo systemctl restart gunicorn
   sudo systemctl reload nginx
   ```

## 💻 Практика
<!-- Практические задания -->

## 🔗 Связи
<!-- Связи с другими уроками -->

## 📌 Заметки
<!-- Дополнительные заметки -->

## 🏷️ Теги
#do_it #урок #django #python #nginx #gunicorn #ubuntu #развертывание #обучение

---
