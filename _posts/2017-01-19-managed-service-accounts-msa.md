---
title: "Managed Service Accounts (MSA)"
date: 2017-01-19
---

Для запуска службы от имени [MSA](https://technet.microsoft.com/ru-ru/library/dd378925(v=ws.10).aspx) необходимо:  
  
**1)**  **Создать MSA (объект AD) в соответствующем домене**  


Пример, аккаунт “**elk** ” в домене “**rackspace** ” для сервера “**elastic** ”.

  
  


Можно удостовериться в наличии объекта при помощи оснастке “Users and Computers”  
  
  


![msa_ad.jpg]({{ '/images/msa_ad.jpg' | absolute_url }})

  
  

**2) Зарегистрировать аккаунт на соответствующем сервере**

  

**3) Настроить запуск служб от имени созданного аккаунта**

**  
**

![select_user.jpg]({{ '/images/select_user.jpg' | absolute_url }})

![service.jpg]({{ '/images/service.jpg' | absolute_url }})

**  
**

  


**4) Убедиться, что у данного аккаунта есть права на все необходимые файлы, каталоги и т.д.**