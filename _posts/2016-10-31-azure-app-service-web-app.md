---
title: "Azure App Service (Web App)"
date: 2016-10-31
---

Допустим, вы открыли доступ к своему on-premise SQL Server для приложения в Azure. Как проверить, что всё сделано правильно?  
  
На Azure Portal открываем <App Service Name> -> Advanced Tools -> Go или проще https://<sitename>.scm.azurewebsites.net/ и попадаем в [Kudu](https://github.com/projectkudu/kudu/wiki).  
  


![kudu.jpg]({{ '/images/kudu.jpg' | absolute_url }})

  


Далее выбираем Debug console -> CMD и в загрузившейся консоли пингуем наш SQL при помощи tcpping.

  


tcpping <sqladdress>:1443

  


![tcpping.jpg]({{ '/images/tcpping.jpg' | absolute_url }})