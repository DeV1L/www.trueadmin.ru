---
title: "VSTS tricks"
date: 2016-10-19
layout: default
---

###  **Вопрос**

Как в Visual Studio Team Services передать переменную (например результат выполнения скрипта) из одного шага (задачи) билда/релиза в другой?

  


###  **Ответ**

Чтобы переменная была доступна между шагами (задачами) её необходимо определить специальным образом.  


Выглядит это так: ##vso[task.setvariable]value

  


###  **Пример**

**Определяем переменную**

$Value = "ALL GLORY TO THE VSTS!"

Write-Host "##vso[task.setvariable variable=Testenv1;]$Value"

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgo-gmVFSnnXE_3a_Oez94t-uZfJGWCtSqBcpn0O7PiC0DyOBg98v32UFUaUyVS8SL6IxpNfXXECXP5c-g2AYdIhqvAqkyV8hWVm0bcWP7Uq1ndGuIfJh_So7GBqeMhontAoqfu8OgS_44n/s640/VSTS+tricks.01.jpg)](images/VSTS+tricks.01.jpg)

  
**Читаем переменную**  


$MyVar = (Get-ChildItem Env:Testenv1).Value  
Write-Verbose "My variable is $MyVar" -Verbose

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh9FBfXT33PcNVPTx_iLouABYKoi7psMibPOTkl2pj9QW0OVVUpFXoyPn17t3jJ1YkmfhG_Ody0LXO-Ro6BPLx4ejvvzxPkxzP64DpZebwc-FUXBt_OdrBkm7LJUSLsVrI6XX0oTkzHHhjE/s640/VSTS+tricks.02.jpg)](images/VSTS+tricks.02.jpg)

  


**Результат**

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhdPpyhL7KRf_KQfHT06b1PP9FMkU0FZvmICEOivaKoa-SeJk4RXyjmMyrVX1cpDDEyVXZ4O3EYXGkqsZDDktlq9aR2axJCdgjMD0avkEyl3UQA47NiaWUhJ396cMLPuXbtL4KCLrQ43gcJ/s640/VSTS+tricks.03.jpg)](images/VSTS+tricks.03.jpg)

  


Подробнее тут: <https://github.com/Microsoft/vsts-tasks/blob/master/docs/authoring/commands.md>

  


  

