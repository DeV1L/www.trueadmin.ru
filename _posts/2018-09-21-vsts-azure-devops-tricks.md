---
title: "VSTS / Azure DevOps tricks"
date: 2018-09-21
---

###  VSTS (Azure DevOps) tricks. Как посмотреть мои комиты за месяц?

  


###  Вопрос

`Есть способ в VSTS сделать query и вытащить мои комиты за месяц?`

###  Ответ
  
В VSTS (Azure DevOps) [API](https://docs.microsoft.com/en-us/rest/api/vsts/git/commits/get%20commits?view=vsts-rest-4.1) есть поддержка работы с Git.  
В PowerShell это можно сделать так.  

```powershell
$PAT = "YOUR VSTS PERSONAL ACCESS TOKEN HERE"
$base64authinfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1}" -f "", $PAT)))
$Instance = "YOUR VSTS INSTANCE NAME"
$Project = "YOUR VSTS PROJECT NAME"
$Author = "YOUR EMAIL"
$fromDate = "8/21/2018 12:00:00 AM"
$toDate = "9/21/2018 12:00:00 AM"
$Version = "4.1"

$Repos = Invoke-RestMethod -Uri "https://dev.azure.com/$Instance/$Project//_apis/git/repositories?$Version" -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Get -ContentType “application/json”

foreach ($_ in $Repos.value.name)
{
$Commits = Invoke-RestMethod -Uri "https://dev.azure.com/$Instance/$Project/_apis/git/repositories/$_/commits?searchCriteria.author=$author&searchCriteria.toDate=$toDate&searchCriteria.fromDate=$fromDate&api-version=$Version" -Headers @{Authorization=("Basic {0}" -f $base64authinfo)} -Method Get -ContentType “application/json”
$Commits | ConvertTo-Json
}
```

Результат будет выглядеть так.  

[![]({{ '/images/Result.jpg' | relative_url }})]({{ '/images/Result.jpg' | relative_url }})  
