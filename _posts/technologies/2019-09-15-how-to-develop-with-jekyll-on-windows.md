---
layout: post
title: Как разрабатывать сайт на jekyll на Windows 10
category: technologies
tags: [jekyll, technologies, windows]
---

# Установка

Прежде всего нужно следовать инструкции по установке на [официальном сайте](https://jekyllrb.com/docs/installation/windows/#installation-via-bash-on-windows-10). Инструкция потребует от юзера установить Linux-подсистему, и уже в ней поставить ruby и jekyll.

Гемы (gem) я ставил через sudo, потому что без супер-юзера установка ругалась на отсутствие прав на запись в определенные директории. Надеюсь, что разрабы jekyll не подложили мне "свинью", и супер-права им я давал не во вред себе.

# Запуск

Запуск сайта нужно осуществлять не с помощью стандартной для MacOs команды `jekyll serve`, а с помощью `bundle exec jekyll serve`. Да и вообще [советуют](https://github.com/jekyll/jekyll/issues/6227#issuecomment-315937985) именно через `bundle exec <command>` запускать команды.