---
title: "VSTS tricks: 100500 build definitions"
date: 2016-12-09
layout: default
---

###  **Вопрос**

**  
**

Допустим Вы хотите изменить какой-то общий параметр сразу во всех Build Definitions. Например, путь для выгрузки артефактов.  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjxpa0VfZJorZ3jFdHciAru9ryiVR-t7I4X4Q73bAi5WgJpxpETAh0E0U5Cu-730QNyH2K22Xk_a_Oe4U7yHA66ewclsu5qHvNbv2CIo04fya8saO5gbPeixP4tRmWvblXSv4cLZCBM7oA7/s640/Build_old.jpg)](images/Build_old.jpg)

  
Как это сделать? Ни экспорта/импорта, ни PowerShell модулей в 2016 году для VSTS не существует. Но зато есть [REST API](https://www.visualstudio.com/en-us/docs/integrate/api/overview).  
  


###  **Ответ**

**  
**

И так, чтобы получить доступ к API необходимо сгенерировать  [Personal access token](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate).  
Переходим в VSTS в раздел Account -> Security -> Personal access tokens -> Add  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjEZ5TJ3ZzmolGsISADpBmFsbmW0l9kYI4kcdJmyDMCPim4QqKxMNdSWT4HTBxhQTfULarg2H1ktDjOgUqZF2WTN596dsDkhzwDGt3x_Ig2srvmMBVFz3PClZ63zHCydKr7iQ4oI6D3_tmo/s640/create+PAT.jpg)](images/create+PAT.jpg)

  
Задаём название, срок действия, нажимаем "Create Token".  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgdCavaZLlRPFvZw5CTOpa9gbrz5YI_hcKaEJ61sf_bbU31VlDZdmGxsv2k0sF2gvYQDfj35w8Q135dRTI9F07YxrjljXBk4kPgow8SSGWwaKirymXKNZCDLYW1_ccWdMXH48QsgvQEaj9z/s640/PAT.jpg)](images/PAT.jpg)

  
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
Тут логично ограничиться работой с JSON, но в моём случае старый адрес \\\KR-VSOBUILD01 присутствует в самых разных шагах билда. Поэтому мне проще сохранить его описание в файл, произвести замену при помощи -Replace и потом прочитать из файла.  
  
  
**В выводе скрипта - результат для каждой Build Definitions**  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgBExd45oLEEimOjoATP_oY5TXzzXqZkpHGjdUD2Q17MGYJQ7obPN9PChpFC66H9mlf1H0l53OqTxXMDGfSNL_A3sclIWyXZWQoBi_ruwt7g-CeW-TxJiq92QrWBpK22bEqw2cRey46JMMu/s640/Log.jpg)](images/Log.jpg)

  
**Скрипт целиком**  
  

