![Main kittygram workflow](https://github.com/antonata-c/kittygram_final/actions/workflows/main.yml/badge.svg?event=push)
# Kittygram
***
### Описание:
Kittygram - социальная сеть для обмена фотографиями любимых питомцев, а именно, котиков.

### Стек используемых технологий:
- _Python 3.8_
- _Django REST Framework_
- _Gunicorn_
- _Nginx_
- _PostgreSQL_
- _Docker_
- _React_


### Подготовка
- Установите докер и клонируйте репозиторий
```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
git clone git@github.com:ваш_логин_на_гитхаб/kittygram_final.git
```
- В корневой директории создайте файл ```.env```, в котором перечислите все переменные окружения, пример находится в файле ```.env.example```
***
### Билд и загрузка образов
- Замените ```username``` на ваш логин на Docker Hub и выполните следующие команды:
```shell
cd frontend
docker build -t username/kittygram_frontend .
cd ../backend
docker build -t username/kittygram_backend .
cd ../nginx
docker build -t username/kittygram_gateway .  
```
- Загрузите получившиеся образы на Docker Hub
```shell
docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway
```
***
### Развертывание на удаленном сервере
- Подключитесь к удаленному серверу и создайте директорию ```kittygram```
```shell
ssh -i path_to_ssh_key/ssh_key_filename username@ip_address
mkdir kittygram
```
- Установите Docker Compose
```shell
sudo apt update
sudo apt install curl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose 
```
- Скопируйте файлы docker-compose.production.yml и .env в корневую директорию проекта
- В этой же директории выполните команду для запуска Docker Compose в режиме демона
```shell
sudo docker-compose -f docker-compose.production.yml up -d
```
- Выполните миграции, соберите и скопируйте статику в ```/backend_static/static/```
```shell
sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml exec backend python manage.py migrate
sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```
***
#### Nginx
- Установите и настройте веб- и прокси-сервер Nginx
```shell
sudo apt install nginx -y
sudo systemctl start nginx
sudo nano /etc/nginx/sites-enabled/default 
```
- Запишите новые настройки для блока server проекта kittygram:
```text
server {
    server_name ваш_домен;
    server_tokens off;

    location / {
        proxy_pass http://127.0.0.1:9000;
    }
}
```
- Сохраните изменения, протестируйте и перезагрузите конфигурацию nginx
```shell
sudo nginx -t
sudo systemctl reload nginx
```
***
### Настройка CI/CD
- Для CI/CD в данном проекте используется Github Actions, все необходимое находится в ```.github/workflows/main.yml```, вам необходимо добавить секреты к вашему репозиторию:
```text
DOCKER_USERNAME // имя пользователя в DockerHub
DOCKER_PASSWORD // пароль в DockerHub
HOST // IP-адрес сервера
USER // имя пользователя
SSH_KEY // содержимое приватного SSH-ключа
SSH_PASSPHRASE // пароль для SSH-ключа
TELEGRAM_TO // ID вашего телеграм-аккаунта
TELEGRAM_TOKEN // токен вашего бота
```
Workflow вызывается при пуше в репозиторий

По завершению деплоя вы будете уведомлены в телеграме!
### Файрвол и SSL-сертификат (Let's Encrypt)
- Настройте файрвол на порты 80, 443 и 22, а затем включите и проверьте:
```shell
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH

sudo ufw enable
sudo ufw status
```
- Установите certbot
```shell
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
```
- Запустите certbot и укажите номер домена для активации HTTPS:
```shell
sudo certbot --nginx

Account registered.
Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains
in a VirtualHost/server block.
1: <доменное_имя_вашего_проекта_1>
2: <доменное_имя_вашего_проекта_2>
3: <доменное_имя_вашего_проекта_3>
...
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
```

#### Проект готов к использованию!
***
### Автор работы:
**[Антон Земцов](https://github.com/antonata-c)**