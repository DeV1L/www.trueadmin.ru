---
title: "WordPress + Application Insights"
date: 2016-12-11
---

Устанавливаем в WordPress плагин [Application Insights](https://wordpress.org/plugins/application-insights/) и активируем его.  
  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-3.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-3.jpg' | absolute_url }})

  


  


В Azire Portal cоздаём экземпляр Application Insights и в разделе "Properties" копируем "INSTRUMENTATION KEY".  
  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-8.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-8.jpg' | absolute_url }})

  


Далее идем в WordPress в раздел "Settings" -> "Application Insights" добавляем наш Instrumentation Key.

  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-9.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-9.jpg' | absolute_url }})

  


Сохраняем изменения и проверяем появление скрипта на страницах сайта.

  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-11.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-11.jpg' | absolute_url }})

  


  


Всё готово, через некоторое время данные начнут поступать в Application Insights и станут доступны на Azure Portal.

  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-12.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-12.jpg' | absolute_url }})

  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-14.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-14.jpg' | absolute_url }})

  


![%D0%97%D0%B0%D1%85%D0%B2%D0%B0%D1%82-15.jpg]({{ '/images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-15.jpg' | absolute_url }})