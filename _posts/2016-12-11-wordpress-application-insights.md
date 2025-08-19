---
title: "WordPress + Application Insights"
date: 2016-12-11
---

Устанавливаем в WordPress плагин [Application Insights](https://wordpress.org/plugins/application-insights/) и активируем его.  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgrmftIvuFm55aWg0ru4P9VX6IEC3ykJshEhmcEfY2VaTTY0YZwy8fCZdPPOc4Fhk-Fkr-9_B58QxzqUvbdBTK3N1a__g2aGGec97gILfpBmk71hSdFrElDEaC9XHhh6ag9PTlfdrFXZKk7/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-3.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-3.jpg)

  


  


В Azire Portal cоздаём экземпляр Application Insights и в разделе "Properties" копируем "INSTRUMENTATION KEY".  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiGOaozk75mclBqIc89zGnL1fq7hbkBE9aNmCkoGpKiWA6SlD7RkUon1lZ3JoevhSIvNH3oEcz2IhyphenhyphenPm5uNzfBo-XHjPh5ZJJfrVzkia8eI-lPs9AtXqjrOTWiBIcIZyCRodCohMngOoKWG/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-8.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-8.jpg)

  


Далее идем в WordPress в раздел "Settings" -> "Application Insights" добавляем наш Instrumentation Key.

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiTgkLugkY8gQ5pdXZ9I2Zqq4R0dwadxm66IXOcfALN5k9RBwuW44UG8BO1iOwxnlQ9AXqj-AZQqzLkqIl1tDKikiOebf5jFiAfV_Ze5a4dLX6BJ3qEFPKETk5axhPs6ALHn8ALCL-RvuBZ/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-9.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-9.jpg)

  


Сохраняем изменения и проверяем появление скрипта на страницах сайта.

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhwS64YoQEs_zrO6NNPeMVRBsgfrAuh5B5mi1zwtZqe66mfmtQitj0rN9YtwonZzj8wLZsOmZxjH_x_hJKeT0ztv-4rb7JIbgSJZ1Fg1oirQoplr8aR5bahyphenhyphens49pIP_QFS4FCQCoOfWQt_W/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-11.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-11.jpg)

  


  


Всё готово, через некоторое время данные начнут поступать в Application Insights и станут доступны на Azure Portal.

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiAHSfgsksJYyRiUu48ii1cmXbp67MGjg3_VmLt9UAAqRvla02PjgAmDN6VanB1Xhk8MGyUGcRsDe02VpylvPiG3bT_ifL8zQVDA9AudaiboqRawnJBTQNgWgA8x4mp7Si6wuHIVpE4EV6Z/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-12.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-12.jpg)

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhK9x1DcyWEbZ5lZk0O9ATXPCksxdQhLZg2tR6GLsY-pZNmneBZtHqLhChfZbrg3ORzH9gEVCmYPGrNk6JnHmm_3oym_HLo2qv1Xdn1yz-M9sEM0HUO4hCukFdsQV51yJs9R2N31dPZlcoJ/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-14.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-14.jpg)

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEigCKvfmZemqG70bRTKVCIs0QoqyFvFHyOwNN-kBkRWmgid0ZNCoEAfB9l1iowo4VkJjoGZc4gU6kHj1NXveyblB4vsiNnqGNpWKGn-2MbDfiakMtNkxS-BWNRFqbwAklcXpMAOHTfjq6_M/s400/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-15.jpg)](images/%25D0%2597%25D0%25B0%25D1%2585%25D0%25B2%25D0%25B0%25D1%2582-15.jpg)

  




