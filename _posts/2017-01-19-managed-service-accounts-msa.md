---
title: "Запуск службы от имени Managed Service Accounts (MSA)"
date: 2017-01-19
---
# Запуск службы от имени Managed Service Accounts (MSA)

Для запуска службы от имени [MSA](https://technet.microsoft.com/ru-ru/library/dd378925(v=ws.10).aspx) необходимо:  
  
**1)**  **Создать MSA (объект AD) в соответствующем домене**  

Пример: аккаунт `elk` в домене `rackspace` для сервера `elastic`.

```powershell
 #On the appropriate DC
Import-Module ActiveDirectory
$ServiceAccountname = "elk"
$Server = "elastic"
New-ADServiceAccount $ServiceAccountname –RestrictToSingleComputer
Add-ADComputerServiceAccount -Identity $Server -ServiceAccount $ServiceAccountname
```

Можно удостовериться в наличии объекта при помощи оснастке `Users and Computers`.

![msa_ad.jpg]({{ '/images/msa_ad.jpg' | absolute_url }})


**2) Зарегистрировать аккаунт на соответствующем сервере**
```powershell
# On the Server
# Install PoS modules if it's needed
# import-Module ServerManager 
# Add-WindowsFeature RSAT-AD-PowerShell,RSAT-AD-AdminCenter

Import-Module ActiveDirectory
$ServiceAccountname = "elk"
Install-ADServiceAccount -Identity $ServiceAccountname
```

**3) Настроить запуск служб от имени созданного аккаунта**

![select_user.jpg]({{ '/images/select_user.jpg' | absolute_url }})

![service.jpg]({{ '/images/service.jpg' | absolute_url }})

**4) Убедиться, что у данного аккаунта есть права на все необходимые файлы, каталоги и т.д.**