# Laboratorium 14 PAwChO

Robert Horbaczewski

Modyfikacja rozwiązania zadania z lab. 13D z wykorzystaniem podziału pliku docker-compose.yaml na plik bazowy oraz override z wykorzystaniem mechanizmu merge.


## Podział plików:

docker-compose.yml:
```yaml
name: lab14

services:

  nginx:
    image: nginx:1.26
    container_name: nginx
    depends_on:
      - php
    networks:
      - frontend
      - backend

  php:
    image: php:8.3-fpm
    container_name: php
    expose:
      - "9000"
    networks:
      - backend

  mysql:
    image: mysql:8.4
    container_name: mysql
    secrets:
      - db_password
      - db_root_password
    volumes:
      - mysql_data:/var/lib/mysql
    expose:
      - "3306"
    networks:
      - backend

  phpmyadmin:
    image: phpmyadmin:5.2.2
    container_name: phpmyadmin
    depends_on:
      - mysql
    networks:
      - frontend
      - backend

secrets:
  db_password:
    file: ./secrets/db_password.txt
  db_root_password:
    file: ./secrets/db_root_password.txt

volumes:
  mysql_data:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```
W pliku bazowym pozostawiono składniki aplikacji, które powinny być takie same we wszystkich środowiskach:
- definicje usług,
- wykorzystywane obrazy,
- sieci,
- wolumeny,
- sekcja secrets,
- zależności między usługami.


docker-compose.override.yml:
```yaml
services:

  nginx:
    ports:
      - "4001:80"
    volumes:
      - ./php:/var/www/html
      - ./nginx/nginx.config:/etc/nginx/conf.d/default.conf:ro

  php:
    volumes:
      - ./php:/var/www/html

  mysql:
    environment:
      MYSQL_DATABASE: testdb
      MYSQL_USER: student
      MYSQL_PASSWORD_FILE: /run/secrets/db_password
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password

  phpmyadmin:
    ports:
      - "6001:80"
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
```
Do pliku override przeniesiono ustawienia środowiskowe:
- mapowanie portów,
- zmienne środowiskowe,
- montowanie katalogów lokalnych.

## Uruchomienie aplikacji:

```bash
(base) roberthorbaczewski@MacBook-Air-M2-Robert Lab14 % docker compose up -d
[+] up 4/4
 ✔ Container mysql      Created                                                                                                                  0.1s
 ✔ Container php        Created                                                                                                                  0.1s
 ✔ Container phpmyadmin Created                                                                                                                  0.2s
 ✔ Container nginx      Created                                                                                                                  0.1s
```
Zastosowanie nazewnictwa zgodnego ze standardową konwencją nazewnictwa dla mechanizmu merge (plik bazowy docker-compose.yml i plik nadpisywania docker-compose.override.yml) pozwoliło pominąć nazwy plików w poleceniu uruchamiającym aplikację.

## Potwierdzenie działania aplikacji:

```bash
(base) roberthorbaczewski@MacBook-Air-M2-Robert Lab14 % docker compose ls
NAME                STATUS              CONFIG FILES
lab14               running(4)          /Users/roberthorbaczewski/Desktop/Studia/Semestr VI/Programowanie Aplikacji w Chmurce/Lab14/docker-compose.yml,/Users/roberthorbaczewski/Desktop/Studia/Semestr VI/Programowanie Aplikacji w Chmurce/Lab14/docker-compose.override.yml
```

```bash
(base) roberthorbaczewski@MacBook-Air-M2-Robert Lab14 % docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED          STATUS                  PORTS                                         NAMES
ce01fe072624   phpmyadmin:5.2.2                   "/docker-entrypoint.…"   36 minutes ago   Up 36 minutes           0.0.0.0:6001->80/tcp, [::]:6001->80/tcp       phpmyadmin
fa759dc1da37   nginx:1.26                         "/docker-entrypoint.…"   36 minutes ago   Up 36 minutes           0.0.0.0:4001->80/tcp, [::]:4001->80/tcp       nginx
eb73194f7084   php:8.3-fpm                        "docker-php-entrypoi…"   36 minutes ago   Up 36 minutes           9000/tcp                                      php
84163401c384   mysql:8.4                          "docker-entrypoint.s…"   36 minutes ago   Up 36 minutes           3306/tcp, 33060/tcp                           mysql
```

### Strona PHP:
![Strona PHP](screenshots/zrzut1)
(Plik index.php niemodyfikowany względem poprzedniego laboratorium)

### phpMyAdmin:
![phpMyAdmin1](screenshots/zrzut2)

![phpMyAdmin2](screenshots/zrzut3)

![phpMyAdmin3](screenshots/zrzut4)
