# Task №6

* Установить Nginx и настроить его на работу с PHP-FPM.
* Установить Apache. Настроить обработку PHP. Добиться одновременной работы с Nginx.
* Настроить схему обратного прокси для Nginx (динамика - на Apache).
* Установить MySQL. Создать новую базу данных и таблицу в ней.
* **Установить пакет phpmyadmin и запустить его веб-интерфейс для управления MySQL.
* **Настроить схему балансировки трафика между несколькими серверами Apache<br>
на стороне Nginx с помощью модуля ngx_http_upstream_module.

# Solution №6

### Установить Nginx и настроить его на работу с PHP-FPM.

1) Установить Nginx:

    ```linux
    sudo apt install nginx
    ```

2) Установить PHP-FPM:

    ```linux
    sudo apt install php-fpm
    ```

3) Настроить Nginx на работу с PHP-FPM:

   * Переходим в каталог где лежат конфиг файлы:
   
   ```linux
   cd /etc/nginx/sites-available
   ```

   * Создадим свой конфиг файл, например такой:
   
   ```linux
   sudo touch project.local
   ```
   
   * Теперь откроем его в редакторе ```nano```:

   ```linux
   sudo nano project.local
   ```
   
   * Пропишем такой вот конфиг:

   ```nano
   server {
        listen 80; # порт, прослушивающий nginx
        server_name    project.local; # доменное имя, относящиеся к текущему виртуальному хосту
        root  /home/stavanger/code/project.local; # каталог в котором лежит проект, путь к точке входа


        index index.php;
        # add_header Access-Control-Allow-Origin *;


        # serve static files directly
        location ~* \.(jpg|jpeg|gif|css|png|js|ico|html)$ {
                access_log off;
                expires max;
                log_not_found off;
        }


        location / {
                # add_header Access-Control-Allow-Origin *;
                try_files $uri $uri/ /index.php?$query_string;
        }

        location ~* \.php$ {
        try_files $uri = 404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock; # подключаем сокет php-fpm
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
                deny all;
        }
   }
   ```
   
   * ```Ctrl+O``` для сохранения и ```Ctrl+X``` выходим из редактора.

   * Теперь активируем наш конфиг файл в каталоге ```/etc/nginx/sites-enabled/```:
   
   ````linux
   cd /etc/nginx/sites-enabled/
   ````

   * Далее нам нужен ``` /etc/hosts```:
   
   ````linux
   sudo nano /etc/hosts
   ````
   
   * Там нам нужно заменить/добавить строку:
   
   ```nano
   127.0.0.1    project.local
   ```
   
   * Запускаем php-fpm:
   
   ```linux
   sudo service php8.1-fpm start
   ```

### Установить Apache. Настроить обработку PHP. Добиться одновременной работы с Nginx.

1) Установить Apache2:

    ```linux
    sudo apt install apache2
    ```

2) Настроить обработку PHP:

   ```linux
   sudo apt install libapache2-mod-php8.1
   ```
   
3) Добиться одновременной работы с Nginx:

   * Для этого надо поменять порт Apache на котором он сидит, для этого открываем файл конфига:

   ```linux
   sudo nano /etc/apache2/ports.conf
   ```
   
   * И тут меняем строку ```Listen```, порт 8888 для примера, можно любой свободный:
   
   ```nano
   Listen 8888
   ```

### Настроить схему обратного прокси для Nginx (динамика - на Apache).

   * Открываем конфиг файл:

   ```linux
   sudo nano /etc/nginx/sites-enabled/default
   ```

   * Вставляем конфиг. Динамику перенаправляем на ```Apache```, статику оставляем на ```Nginx```:

   ```nano
   server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                proxy_pass http://localhost:8888;
                proxy_set_header Host Shost;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP Sremote_addr;
        }

        location ~* ^.+.(jpg|jpeg|gif|png|ico|css|zip|pdf|txt|tar|js)$ {
                root /var/www/html;
        }
   }
   ```

### Установить MySQL. Создать новую базу данных и таблицу в ней.

1) Установить MySQL:

   ```linux
   sudo apt install mysql-server-8.0
   ```

2) Создать новую базу данных и таблицу в ней:

   * Запуск MySQL:
   
   ```linux
   sudo mysql
   ```

   * Создание БД:
   
   ```MySQL
   CREATE DATABASE my_db;
   ```
   
   * Переключаемся на нашу БД:
   
   ```MySQL
   USE my_db;
   ```
   
   * Создаём новую таблицу:
   
   ```MySQL
   CREATE TABLE new_table (
   ```
   
   * Вводим поля таблицы и перечисляем их параметры:

   ```MySQL
   id INT PRIMARY KEY AUTO_INCREMENT,
   ```

   ```MySQL
   name TEXT);
   ```
   
   * Добавляем данные в таблицу:
   ```MySQL
   INSERT INTO new_table (name) VALUES ('John'), ('Lera'), ('George');
   ```
   
   * Просмотрим нашу таблицу:

   ```MySQL
   SELECT * FROM new_table;
   ```

### **Установить пакет phpmyadmin и запустить его веб-интерфейс для управления MySQL.

1) Установить пакет phpmyadmin:
   
   ```linux
   sudo apt install phpmyadmin
   ```
   
   * При выборе сервера ставим ```apache2```
   * Соглашаемся с настройкой
   * Устанавливаем пароль
   * Запускаем ```MySQL```

   ```linux
   sudo mysql
   ```
   
   * Создаём пользователя, задаём пароль и задаём ему права:

   ```MySQL
   CREATE USER 'sammy'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'ПАРОЛЬ';
   GRANT ALL PRIVILEGES ON *.* TO 'sammy'@'localhost' WITH GRANT OPTION;
   ```
   
2) Запускаем браузер и переходим по адресу ```http://localhost:8888/phpmyadmin/index.php```

### **Настроить схему балансировки трафика между несколькими серверами Apache<br> на стороне Nginx с помощью модуля ngx_http_upstream_module.