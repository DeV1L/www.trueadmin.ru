---
title: "Email from Azure Functions — secure way"
date: 2018-05-07
---

# Email from Azure Functions — secure way
  
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

![Platform features.jpg]({{ '/images/Platform+features.jpg' | absolute_url }})

  
Включаем **"Register with Azure Active Directory"**  
  
![Register with Azure Active Directory.jpg]({{ '/images/Register+with+Azure+Active+Directory.jpg' | absolute_url }})

Это позволить приложению запрашивать access token для получения доступа к другим ресурсам Azure (в нашем случае Key Vault). Подробнее [тут](https://docs.microsoft.com/en-us/azure/app-service/app-service-managed-service-identity).  
  
###  Настройка Azure Key Vault
Создаём Key Vault по [инструкции](https://docs.microsoft.com/ru-ru/azure/key-vault/quick-create-portal). Добавляем логин и пароль от почтового аккаунта.  
  
**Key vault - > Secrets -> Generate/Import**  

![sender login.jpg]({{ '/images/sender+login.jpg' | absolute_url }})


![sender password.jpg]({{ '/images/sender+password.jpg' | absolute_url }})

  
Разрешаем нашему Function App запрашивать эти логин и пароль. Для этого идём в **Key vault - > Access policies -> Add new  **и выбираем:  

  * **Secret permissions** : Get, List
  * **Select principal** : test-azurefunctions-notifications

![Access policy.jpg]({{ '/images/Access+policy.jpg' | absolute_url }})

###  Отправка уведомлений
Проверим, что мы можем использовать сохранённые учётные данные.  
  
Приведу вариант реализации на PowerShell. Примеры для других языков можно посмотреть в [документации](https://docs.microsoft.com/en-us/azure/app-service/app-service-managed-service-identity#obtaining-tokens-for-azure-resources) Microsoft.  
  
![Function.jpg]({{ '/images/Function.jpg' | absolute_url }})

  
Результат будет выглядеть примерно так:  
![Email.jpg]({{ '/images/Email.jpg' | absolute_url }})

  
Код на PowerShell.  
```powershell
#Email settings
$SendTo = "YOUR_EMAIL"
$Subj = "Hello from Azure Function"
$SMTPserver = "smtp.office365.com"
$SMTPPort = "587"
$VaultURI = "https://YOUR.vault.azure.net/"
$SenderLoginName = "YOUR-sender-ofice365-login" #How it's called in the Key Vault
$SenderPasswordName = "YOUR-sender-ofice365-password" #How it's called in the Key Vault


#Function: get Key Vault access token
function Get-AccessTokenKeyVault 
{
	$ResourceURI = "https://vault.azure.net"
	$ApiVersion = "2017-09-01"
	$TokenAuthURI = $env:MSI_ENDPOINT + "?resource=$ResourceURI&api-version=$ApiVersion"
	$TokenResponse = Invoke-RestMethod -Method Get -Headers @{"Secret"="$env:MSI_SECRET"} -Uri $TokenAuthURI
	$TokenResponse.access_token
}

#Function: get Key Vault secret
function Get-KeyVaultSecret
{
Param(
    [string] $AccessTokenKeyVault,
    [string] $SecretName
)
    $ResourceURI = $VaultURI
    $Uri = $ResourceURI + "secrets/$SecretName/?api-version=2016-10-01"
    $keysResponse = Invoke-RestMethod -Method Get -Headers @{Authorization="Bearer $AccessTokenKeyVault"} -Uri $Uri
    $keysResponse
}

function Send-Result 
{
Param(
	$Body
)
	$Credentials = New-Object System.Management.Automation.PSCredential -ArgumentList $From, $($Password | ConvertTo-SecureString -AsPlainText -Force) 
	Send-MailMessage –From $From –To $SendTo –Subject $Subj –Body $Body -SmtpServer $SMTPserver -Credential $Credentials -UseSsl -Port $SMTPPort
}

#Obtain secrets
$AccessTokenKeyVault = Get-AccessTokenKeyVault
$From = (Get-KeyVaultSecret -AccessToken $AccessTokenKeyVault -SecretName $SenderLoginName).value
$Password = (Get-KeyVaultSecret -AccessToken $AccessTokenKeyVault -SecretName $SenderPasswordName).value

#Send email
Send-Result -Body "This is test from Function App at $(Get-Date)"
```