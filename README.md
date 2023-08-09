# Kittygram
# Автор: fyurikon
### Описание
Лучшее место для добавления и хранения котиков вместе с их достижениями.
### Технологии
Django | djangorestframework | djoser

## Развёртывание проекта на удалённом сервере:
Клонируем репозиторий:
```
git clone git@github.com:fyurikon/infra_sprint1.git
```
Переходим в директорию проекта:
```
cd infra_sprint1/
```
Создаем файл .env и заполнить его по следующий образом:
SECRET_KEY = Секретный ключ Django-проекта
DEBUG = False или True. В зависимости от того в каком режиме мы хотим запустить проект.
ALLOWED_HOSTS = xxx.xxx.xxx.xxx 127.0.0.1 localhost. xxx.xxx.xxx.xxx - это внешний IP нашего сервера на котором мы развёртываем проект. Также через пробел можно добавить доменное имя.

Создаём директорию для хранения фоток котиков:
```
sudo mkdir /var/www/kittygram/media/
```
Чтобы сохранять картинки нужно назначить нашего пользователя владельцем директории:
```
sudo chown -R <имя_пользователя> /var/www/kittygram/media/
```
Далее создаём окружение:
```
python3 -m venv venv
```
И активируем его:
```
source env/bin/activate
```
Устанвливаем зависимости из requirements.txt:
```
pip install -r requirements.txt
```
Идём в директорию backend приложения и выполняем миграции:
```
cd backend/
```
```
python manage.py migrate
```
Создаём суперпользователя:
```
python manage.py createsuperuser
```
Собираем статику бэкенд-приложения:
```
python manage.py collectstatic 
```
Копируем директорию static_backend/ в директорию веб-сервера Nginx:
```
sudo cp -r static_backend/ /var/www/kittygram/
```
Далее переходим в директорию фронтенд приложения:
```
cd ../frontend/
```
И устанавливаем зависимости:
```
npm i
```
Собираем статику фронтенд-приложения:
```
npm run build 
```
Копируем всё что в директории build/ в директорию веб-сервера Nginx:
```
sudo cp -r build/. /var/www/kittygram/
```
Теперь настроем gunicorn server. Для этого создаём файл gunicorn_kittygram.service:
```
sudo vi /etc/systemd/system/gunicorn_kittygram.service
```
В файл копируем следующее
```
[Unit]
Description=kittygram_backend daemon 

After=network.target 

[Service]
User=<имя-пользователя-в-системе>

WorkingDirectory=/home/<имя-пользователя-в-системе>/infra_sprint1/backend/

ExecStart=/home/<имя-пользователя-в-системе>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target
```
Запускаем gunicorn_kittygram.service:
```
sudo systemctl start gunicorn_kittygram
```
Добавдяем процесс в автозапуск:
```
sudo systemctl enable gunicorn_kittygram
```
Проверяем статус:
```
sudo systemctl status gunicorn_kittygram
```
Конфигурируем nginx:
```
sudo vi /etc/nginx/sites-enabled/default 
```
Следующим образом:
```
server {
    server_name <ваш-ip> <ваш-домен>;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
    

    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }

    location / {
        root   /var/www/kittygram;
        index  index.html index.htm;
        try_files $uri /index.html;
    }
} 
```
Проверяем конфиг на ошибки:
```
sudo nginx -t 
```
Перезагружаем Nginx:
```
sudo systemctl reload nginx 
```
