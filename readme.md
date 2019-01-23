# Этот мануал является дополнением к курсу канала [letsCode](https://www.youtube.com/watch?v=jH17YkBTpI4&list=PLU2ftbIeotGpAYRP9Iv2KLIwK36-o_qYk&index=1) </br>

##Тема: деплой приложения Sweater на сервер aws EC2 Amazon Linux 2 AMI из под Windows.


###Полезные ссылки и команды

[канал letsCode](https://www.youtube.com/watch?v=kT_xEflmaGE&index=14&list=PLU2ftbIeotGoGSEUf54LQH-DgiQPF2XRO)

[канал AWT-IT](https://www.youtube.com/watch?v=8jbx8O3wuLg&list=PLg5SS_4L6LYsxrZ_4xE_U95AtGsIB96k9)

[использование редактора vi](https://www.youtube.com/watch?v=6H0GDM8ExB8&list=PLU2ftbIeotGpewHtUVXBGfxywja3Ko-eA)

[видео манула](https://www.youtube.com/watch?v=boq5z0VRZOo&feature=youtu.be)

1. Создаём сервер на aws через EC2. Используем Amazon Linux2
            
    1.1. Регистрируем учётную запись (см видео выше)
     
    1.2. Создаём сервер EC Amazon Linux 2 AMI
    
    1.3. Создаём ключ и сохраняем в /home/user/.ssh на локальном компьютере
    
2. Копируем из настроек сервера PublicDNS и PublicIP в проект

3. В проекте в файле application.properties (и в классах, которые используют эту переменную)переименовываем "hostname" в свой вариант. Сервер Амазона перекрывает своей переменной "hostname" наши данные.
    
4. Добавляем ip сервера в googleRecaptcha

5. Прописываем ключ в скрипте

6. Можно использовать любую консоль. Но мы будем использовать bash прямо из Idea.
    Запускаем Terminal и набираем 
        
        bash --login
         
    Через ssh заходим на удалённый сервер.
            
        ssh -i ~/.ssh/sweater-london.pem ec2-user@ec2-18-130-75-9.eu-west-2.compute.amazonaws.com
    
    На вопрос отвечаем полным словом "yes".

7. Устанавливаем PostgreSQL
    качаем версию постгреса 9.6 (в репо сервера версия 9.2, а мы используем FlyWay, который работает с PostgreSQL от версии 9.4)
        
        sudo rpm -ivh https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-6-x86_64/pgdg-ami201503-96-9.6-2.noarch.rpm
        
    и теперь устанавливаем её
    
        sudo yum install postgresql96 postgresql96-server postgresql96-devel postgresql96-contrib postgresql96-libs postgresql96-docs
    
    проверяем:  
        
        psql --version
        
    получим ответ: 
    
        psql (PostgreSQL) 9.6.11

8. Настраиваем БД

    инициализация: 
        
        sudo service postgresql-9.6 initdb
        
    редактируем настройки: 
        
        sudo vim /var/lib/pgsql/9.6/data/pg_hba.conf
    
    мы должны получить такой результат:
    
        # "local" is for Unix domain socket connections only
        local   all             all                                 trust
        # IPv4 local connections:
        host    all             postgres        0.0.0.0/0            md5
        # IPv6 local connections:
        host    all             all             ::1/128              md5

    вносим правки в следующий фал:

        sudo vim /var/lib/pgsql/9.6/data/postgresql.conf
 
    Стартуем сервер: 
    
        sudo service postgresql-9.6 start

    Заходим на сервер: 
        
        sudo su - postgres
        psql
    
    появится приглашение:
    
        postgres=#

    назначаем главномю юзеру БД пароль(например на qwerty) - именно эти лог/пас указываем в java-коде для доступа к БД
    
        ALTER USER postgres WITH PASSWORD 'qwerty'; 
    
    создаём базу данных:

        CREATE DATABASE sweater WITH OWNER postgres;



9.  В личном кабинете Амазона в разделе Security Group настраиваем доступ для постгрес.

10. Устанавливаем java 8

        sudo yum install java-1.8.0-openjdk-devel

    и проверяем как поставилось:
        
        java -version
 

11. Устанавливаем NGINX 

        sudo amazon-linux-extras install nginx1.12

    редактируем конфигурацию:
        
        sudo vi  /etc/nginx/nginx.conf

    меняем имя пользователя. Было "nginx" стало "ec2-user".
    
    обновляем раздел location так:

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_pass http://localhost:8080;
        }

        location /img/ {
            alias /home/ec2-user/uploads/;
        }

    открываем доступ пользователю ec2-user к папке сервера nginx:
        
        sudo chown -Rf ec2-user:ec2-user /var/lib/nginx

    запускаем сервер:
    
        sudo service nginx start
        
    если нужен перезапуск сервера:
        
        sudo nginx -s reload
        
    если нужно посмотреть логи nginx:
        
        sudo vim /var/log/nginx/error.log

12. создаём папку для картинок uploads
        
        mkdir uploads

13. выходим из консоли сервера и возвращаемся в локальный баш. запускаем скрипт
        
        ./scripts/deploy.sh

14. Заходим на сервер через браузер по IP

        18.130.75.9

P.S. Все пароли, конкретные адреса и IP приведённые выше на каждом сервере будут отличаться.