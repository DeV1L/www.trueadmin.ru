---
title: "WordPress + Application Insights"
date: 2016-12-11
---
Устанавливаем в WordPress плагин [Application Insights](https://wordpress.org/plugins/application-insights/) и активируем его.  
  

![zahvat-3]({{ '/images/zahvat-3.jpg' | absolute_url }})

В Azire Portal cоздаём экземпляр Application Insights и в разделе `Properties` копируем `INSTRUMENTATION KEY`.  
  
![zahvat-8]({{ '/images/zahvat-8.jpg' | absolute_url }})

  
Далее идем в WordPress в раздел `Settings` -> `Application Insights` добавляем наш Instrumentation Key.

![zahvat-9]({{ '/images/zahvat-9.jpg' | absolute_url }})

Сохраняем изменения и проверяем появление скрипта на страницах сайта.

![zahvat-11]({{ '/images/zahvat-11.jpg' | absolute_url }})

Всё готово, через некоторое время данные начнут поступать в Application Insights и станут доступны на Azure Portal.

  ![zahvat-12]({{ '/images/zahvat-12.jpg' | absolute_url }})

![zahvat-14]({{ '/images/zahvat-14.jpg' | absolute_url }})

![zahvat-15]({{ '/images/zahvat-15.jpg' | absolute_url }})