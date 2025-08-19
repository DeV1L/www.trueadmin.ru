---
title: "Azure App Service (Web App)"
date: 2016-10-31
---

Допустим, вы открыли доступ к своему on-premise SQL Server для приложения в Azure. Как проверить, что всё сделано правильно?  
  
На Azure Portal открываем <App Service Name> -> Advanced Tools -> Go или проще https://<sitename>.scm.azurewebsites.net/ и попадаем в [Kudu](https://github.com/projectkudu/kudu/wiki).  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgXMm6WuLpo4uIugmRJKk6gfwbe4k1izAENoqNEpUbReYcKOpBr_LsvIxRyX8mX5txUYSZdcdTrBGQ_AIgA8qfEbGsIAKdrMSozd1zgOyW1vMpAQpO3Dh5eCfu0g5moMxzsaEFPz2pcGUnU/s400/kudu.jpg)](images/kudu.jpg)

  


Далее выбираем Debug console -> CMD и в загрузившейся консоли пингуем наш SQL при помощи tcpping.

  


tcpping <sqladdress>:1443

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjHHHQEz6SSD-0p9Yx_L3Lu5tnlDSlSBxPQCuTPPBH4rVoUwxk_2CJARIzgAlSPqlDyUajKaDLOidRTb5uxeY9IUXfxKj1yzXxpTM1fGcMXEKGLK3EAXh_pZDyYm-GJZzrYtNCKtkNMO9X_/s400/tcpping.jpg)](images/tcpping.jpg)

  





