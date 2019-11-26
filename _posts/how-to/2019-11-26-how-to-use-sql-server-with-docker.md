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

docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>"
   -p 1433:1433
   --name <your_awesome_name>
   -d mcr.microsoft.com/mssql/server:2019-GA-ubuntu-16.04

```

После выполнения команды создастся образ сервера в докере, доступный по адресу `localhost:1433`. Посмотреть имеющиеся образы в докере можно, вызвав команду:

```powershell

docker ps -a

```

## 3. Подключиться к SQL Server через MS SQL Management Studio (SSMS)

Имхо удобнее работать с SQL сервером через GUI-клиент, чем через консоль. Для этого мы будем использовать MS SQL Management Studio (далее SSMS). Устанавливаем и запускаем, а затем подключаемся к серверу по следующим данным:

![login window](/img/2019-11-26-how-to-use-sql-server-with-docker/login.png)

Что вводим:

- Имя сервера - локальный ip-адрес и через запятую порт, который указали в команде для докера: `127.0.0.1,1433`
- Логин: `sa`
- Пароль: тот самый `<YourStrong@Passw0rd>`, что указали в команде для докера

## Полезные ссылки:

- [Quickstart: Run SQL Server container images with Docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-powershell)
- [Using SSMS to remote connect to docker container](https://stackoverflow.com/a/48105688)
