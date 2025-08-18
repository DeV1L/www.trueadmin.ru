---
title: "Запуск службы от имени Managed Service Accounts (MSA)"
date: 2017-01-19
---

Для запуска службы от имени [MSA](https://technet.microsoft.com/ru-ru/library/dd378925\(v=ws.10\).aspx) необходимо:  
  
**1)**  **Создать MSA (объект AD) в соответствующем домене**  


Пример, аккаунт “**elk** ” в домене “**rackspace** ” для сервера “**elastic** ”.

  
  


Можно удостовериться в наличии объекта при помощи оснастке “Users and Computers”  
  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiQFalCqq3OgvNDUlNCE43k3nYWairkHfeOUoMvPaBHMm0CrXC7r_NZozhlLaYgBHIjIUheS9vlYoEz2JDNWsaK9eu_euJvMM_V16n5TubgHwOXuDjA0Hxae2FXBr-JNspkCoY6qImZlk6h/s640/msa_ad.jpg)](/images/msa_ad.jpg)

  
  


**2) Зарегистрировать аккаунт на соответствующем сервере**

  


**3) Настроить запуск служб от имени созданного аккаунта**

**  
**

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiHWeI5j7xxordKitJ4BwDZJePDicvwqNNsN02Oyt9UAR-QLmvvpA0Kk_ff0kV9CAN3bxnxNChd0AGz0ok6fa1ReYMvfOnfmgDQ79JXQue_sM6__kpZzrx02buBTzc3mekqXZKTqiX0RORK/s400/select_user.jpg)](/images/select_user.jpg)

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiY35dhSafW9s8pfchRNMoO_qkgE3yMj_HtJvkN15yc8eoZhu9BKzPl_34ML2voRbLpDra25hFkQpzlE2bJt13hjFYQN8n-BKPg6-0cNQMwO097PrAJcvLfEhR7peOat9j9JP1JxPQASc-X/s400/service.jpg)](/images/service.jpg)

**  
**

  


**4) Убедиться, что у данного аккаунта есть права на все необходимые файлы, каталоги и т.д.**
