---
title: "VSTS tricks. Как изменить 100500 Build Definitions за один ра"
date: 2016-12-09
---
# VSTS tricks. Как изменить 100500 Build Definitions за один ра
###  **Вопрос**

Допустим Вы хотите изменить какой-то общий параметр сразу во всех Build Definitions. Например, путь для выгрузки артефактов.  
  
![Build_old.jpg]({{ '/images/Build_old.jpg' | absolute_url }})

Как это сделать? Ни экспорта/импорта, ни PowerShell модулей в 2016 году для VSTS не существует. Но зато есть [REST API](https://www.visualstudio.com/en-us/docs/integrate/api/overview).  
  
###  **Ответ**

И так, чтобы получить доступ к API необходимо сгенерировать  [Personal access token](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate).  
Переходим в VSTS в раздел `Account` -> `Security` -> `Personal access tokens` -> `Add` 
  
![create PAT.jpg]({{ '/images/create+PAT.jpg' | absolute_url }})

Задаём название, срок действия, нажимаем `Create Token`.  
  
![PAT.jpg]({{ '/images/PAT.jpg' | absolute_url }})

  
Теперь приступаем к скрипту на PowerShel.  
Сначала зададим основные параметры.

```powershell
$User = "Script"
$PAT = "PUT YOUR PERSONAL ACCESS TOKEN HERE"
$base64authinfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $User, $PAT)))
$Instance = "PUT VSTS ADDRESS HERE"
$Project = "PUT YOUR PROJECT NAME HERE"
$Version = "2.0"
$OutputDir = $env:TEMP
```
  
- `$PAT` - Только что сгенерированный токен  
- `$Instance`, `$Project` - URL Вашего VSTS и название проекта, подробнее [здесь](https://www.visualstudio.com/en-us/docs/integrate/api/xamlbuild/overview).  
- `$Version` - Актуальная версия API  
  
Далее сформируем HTTP запрос для получения списка ID всех Build Definitions.  

```powershell
$BuildDefinitionsListUrl = "$instance/DefaultCollection/$Project/_apis/build/definitions?api-version=$Version"
$BuildDefinitionsList = Invoke-RestMethod -Uri $BuildDefinitionsListUrl -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Get -ContentType “application/json”
```

В ответ мы получим JSON с которым довольно удобно работать в PowerShell.  
Так можно получить все ID.  
```powershell
$BuildDefinitionsIDs = $BuildDefinitionsList.value.id 
``` 
  
Далее сделаем цикл, который будет проходить по всем  Build Definitions, выгружать описание в JSON и заменять необходимые параметры.  
Тут логично ограничиться работой с JSON, но в моём случае старый адрес `\\​KR-VSOBUILD01` присутствует в самых разных шагах билда. Поэтому мне проще сохранить его описание в файл, произвести замену при помощи `-Replace` и потом прочитать из файла.  
```powershell
foreach ( $_ in $BuildDefinitionsIDs) { 
$url = "$instance/DefaultCollection/$Project/_apis/build/definitions/"+$_+"?api-version=$Version"
$BuildDefinition = Invoke-RestMethod -Uri $url -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Get -ContentType “application/json” -OutFile $OutputDir\$_ -Verbose
$BuildDefinitionReplaced = ((Get-Content $File) | Foreach-Object {$_ -replace 'KR-VSOBUILD01','KR-TFS'})
$BuildDefinitionReplaced = Get-Content $OutputDir\$_
Invoke-RestMethod -Uri $url -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Put -ContentType “application/json” -Body $BuildDefinitionReplaced -Verbose
}
```
  
**В выводе скрипта - результат для каждой Build Definitions**  

![Log.jpg]({{ '/images/Log.jpg' | absolute_url }})

**Скрипт целиком**
```powershell
$User = "Script"
$PAT = "PUT YOUR PERSONAL ACCESS TOKEN HERE"
$base64authinfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f $User, $PAT)))
$Instance = "PUT VSTS ADDRESS HERE"
$Project = "PUT YOUR PROJECT NAME HERE"
$Version = "2.0"
$OutputDir = $env:TEMP

$BuildDefinitionsListUrl = "$instance/DefaultCollection/$Project/_apis/build/definitions?api-version=$Version"
$BuildDefinitionsList = Invoke-RestMethod -Uri $BuildDefinitionsListUrl -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Get -ContentType “application/json”
$BuildDefinitionsIDs = $BuildDefinitionsList.value.id

foreach ( $_ in $BuildDefinitionsIDs) { 
$url = "$instance/DefaultCollection/$Project/_apis/build/definitions/"+$_+"?api-version=$Version"
$File = "$OutputDir\$_"
$BuildDefinition = Invoke-RestMethod -Uri $url -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Get -ContentType “application/json” -OutFile $OutputDir\$_ -Verbose
$BuildDefinitionReplaced = ((Get-Content $File) | Foreach-Object {$_ -replace 'KR-VSOBUILD01\\\\AutoBuildsDrop','KR-TFS\\AutoBuildsDrop'}) 
$BuildDefinitionReplaced = Get-Content $OutputDir\$_
Invoke-RestMethod -Uri $url -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Put -ContentType “application/json” -Body $BuildDefinitionReplaced -Verbose
}
```