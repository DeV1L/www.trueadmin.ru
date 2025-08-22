---
title: "VSTS tricks: 100500 build definitions"
date: 2016-12-09
---

###  **Вопрос**

**  
**

Допустим Вы хотите изменить какой-то общий параметр сразу во всех Build Definitions. Например, путь для выгрузки артефактов.  
  

![Build_old.jpg]({{ '/images/Build_old.jpg' | absolute_url }})

  
Как это сделать? Ни экспорта/импорта, ни PowerShell модулей в 2016 году для VSTS не существует. Но зато есть [REST API](https://www.visualstudio.com/en-us/docs/integrate/api/overview).  
  

###  **Ответ**

**  
**

И так, чтобы получить доступ к API необходимо сгенерировать  [Personal access token](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate).  
Переходим в VSTS в раздел Account -> Security -> Personal access tokens -> Add  
  

![create PAT.jpg]({{ '/images/create+PAT.jpg' | absolute_url }})

  
Задаём название, срок действия, нажимаем "Create Token".  
  

![PAT.jpg]({{ '/images/PAT.jpg' | absolute_url }})

  
Теперь приступаем к скрипту на PowerShel.  
Сначала зададим основные параметры  
  
$PAT - Только что сгенерированный токен  
$Instance, $Project - URL Вашего VSTS и название проекта, подробнее [здесь](https://www.visualstudio.com/en-us/docs/integrate/api/xamlbuild/overview).  
$Version - Актуальная версия API  
  
Далее сформируем HTTP Get запрос для получения списка ID всех Build Definitions.  
  
В ответ мы получим JSON с которым довольно удобно работать в PowerShell.  
Так можно получить все ID.  
  
$BuildDefinitionsIDs = $BuildDefinitionsList.value.id  
  
Далее сделаем цикл, который будет проходить по всем  Build Definitions, выгружать описание в JSON и заменять необходимые параметры.  
Тут логично ограничиться работой с JSON, но в моём случае старый адрес \\\​KR-VSOBUILD01 присутствует в самых разных шагах билда. Поэтому мне проще сохранить его описание в файл, произвести замену при помощи -Replace и потом прочитать из файла.  
  
  
**В выводе скрипта - результат для каждой Build Definitions**  
  

![Log.jpg]({{ '/images/Log.jpg' | absolute_url }})

  
**Скрипт целиком**