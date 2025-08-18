---
title: "Отправка Email из Azure Functions: secure way"
date: 2018-05-07
---

  
[Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/) \- очень удобный инструмент для автоматизации каких-либо задач. Как-только автоматизация настроена - сразу возникает вопрос: как получать уведомления о результатах? Самое логичное решение - использовать электронную почту для отправки оповещений.  
Как делать это безопасно и бесплатно расскажу ниже.  
  
**Подсказка** : данный мануал можно использовать не только для отправки почты, но и впринципе для безопасной работы с любыми секретами/сертификатами/ключами.  
  


###  Постановка задачи

  
Пускай у нас уже есть:  
  


  * Function App с именем "test-azurefunctions-notifications"
  * Учётная запись от почтового сервера (пускай будет Office 365)

  
Сделаем так, чтобы мы могли:  
  


  * Отправлять уведомления из Function App
  * Не хардкодить учётку в коде Function App

  
Все настройки можно выполнить при помощи Azure Portal.  
  


###  Настройка Function App

Открываем:**Function App - > Platform features -> NETWORKING -> Managed service identity**  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgn_GQqvB9tsJ5ZSaL-6UKZnCsXnTTTgJvRvpHS_AAz6FwaIyxjcQVqmxRg9Xv8MrR3_045T7-HuplbC6V1E4T7SCS3YQaJngS0G6aBDA5RedcF5M4c-QG76da-iIyskgPcjrlQ2A-_myPN/s640/Platform+features.jpg)](/images/Platform+features.jpg)

  
Включаем **"Register with Azure Active Directory"**  
  


[![](/images/Register+with+Azure+Active+Directory.jpg)](/images/Register+with+Azure+Active+Directory.jpg)

  
Это позволить приложению запрашивать access token для получения доступа к другим ресурсам Azure (в нашем случае Key Vault). Подробнее [тут](https://docs.microsoft.com/en-us/azure/app-service/app-service-managed-service-identity).  
  


###  Настройка Azure Key Vault

Создаём Key Vault по [инструкции](https://docs.microsoft.com/ru-ru/azure/key-vault/quick-create-portal). Добавляем логин и пароль от почтового аккаунта.  
  
**Key vault - > Secrets -> Generate/Import**  
  


[![](/images/sender+login.jpg)](/images/sender+login.jpg)

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEijs5HTl_hcWKC1fRgH7SYcm9xlp8NVXSsQuFVCE90VnaUDDhnXGJgG5RopYPCRXMuBdoIwIAbWq66idLZgvnI3443_WXQACKCMamnofQvbPXOFD3RUu0Nge_1Ta13IXYwnLmkWpHTaLa4Y/s640/sender+password.jpg)](/images/sender+password.jpg)

  
Разрешаем нашему Function App запрашивать эти логин и пароль. Для этого идём в **Key vault - > Access policies -> Add new  **и выбираем:  


  * **Secret permissions** : Get, List
  * **Select principal** : test-azurefunctions-notifications



[![](/images/Access+policy.jpg)](/images/Access+policy.jpg)

  


  


###  Отправка уведомлений

Проверим, что мы можем использовать сохранённые учётные данные.  
  
Приведу вариант реализации на PowerShell. Примеры для других языков можно посмотреть в [документации](https://docs.microsoft.com/en-us/azure/app-service/app-service-managed-service-identity#obtaining-tokens-for-azure-resources) Microsoft.  
  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg8zoLZ-jwjg1qpeGVCAvfirGGW_nhVrcPlwBXyNYwP89i3OvmZKshBhA43BAFrU7ywq3WHHCFBFa8UMz6TFQR2ykw-ey3hk4cYtNtNISb0QFlPAu3LAMtpiNZj7PaurbkxxAWn6QPZ1Sxe/s640/Function.jpg)](/images/Function.jpg)

  
  
Результат будет выглядеть примерно так:  
  


[![](/images/Email.jpg)](/images/Email.jpg)

  
  
Код на PowerShell.  

