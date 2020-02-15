---
layout: post
title: Как использовать SQL Server и Docker
category: how-to
tags: [tutorial, database, docker]
---

Чтобы использовать MS SQL Server на компьютере без установки самого сервера, мы можем использовать образ для докера. Полная инструкция дана наиболее полно в самой [документации](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-powershell) от MS, однако здесь я бы привел краткое резюме и что именно я делал.

## 1. Установить Docker

Очевидно

## 2. Установить образ SQL Server в Docker

Для этого нужно вызвать следующую команду:

```powershell

docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=ssms-password" `
   -p 1433:1433 `
   -d mcr.microsoft.com/mssql/server:2019-GA-ubuntu-16.04

```

После выполнения команды создастся образ сервера в докере, доступный по адресу `localhost:1433`. Посмотреть имеющиеся образы в докере можно, вызвав команду:

```powershell

docker ps -a

```

## 3. Подключиться к SQL Server через MS SQL Management Studio (SSMS)

Имхо удобнее работать с SQL сервером через GUI-клиент, чем через консоль. Для этого мы будем использовать MS SQL Management Studio (далее SSMS). Устанавливаем и запускаем, а затем подключаемся к серверу по следующим данным:

![login window](/assets/img/2019-11-26-how-to-use-sql-server-with-docker/login.png)

Что вводим:

- Имя сервера - локальный ip-адрес и через запятую порт, который указали в команде для докера: `127.0.0.1,1433`
- Логин: `sa`
- Пароль: тот самый `<YourStrong@Passw0rd>`, что указали в команде для докера

## Upd: Файл docker-compose для докера

Для упрощения работы с сервером базы данных можно создать скрипт, который будет запускать сервер только на время, пока открыт скрипт. Соответственно, когда скрипт закрывается, то и контейнер останавливается и перестает есть ресурсы. Для этого нужно создать два файла в некоторой директории:

1. `docker-compose.yml`:

```yml

version: "3.4"

services:
  database:
    container_name: "sql-server-2019"
    image: "mcr.microsoft.com/mssql/server:2019-latest"
    environment:
      SA_PASSWORD: "STRONG!Passw0rd"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"

```

2. Файл-лаунчер сервера `run.ps1`:

```powershell

docker-compose -f "docker-compose.yml" stop
docker-compose -f "docker-compose.yml" rm --force
docker-compose -f "docker-compose.yml" up --build database

```

3. Теперь можно использовать следующую строку подключения в своем приложении:

```json

"ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=<Database Name>;Trusted_Connection=False;MultipleActiveResultSets=true;User ID=SA;Password=STRONG!Passw0rd"
  }

```


## Полезные ссылки:

- [Quickstart: Run SQL Server container images with Docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-powershell)
- [Using SSMS to remote connect to docker container](https://stackoverflow.com/a/48105688)
