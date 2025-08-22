---
title: "VSTS tricks"
date: 2016-10-19
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

  


![VSTS tricks.01.jpg]({{ '/images/VSTS+tricks.01.jpg' | absolute_url }})

  
**Читаем переменную**  


$MyVar = (Get-ChildItem Env:Testenv1).Value  
Write-Verbose "My variable is $MyVar" -Verbose

  


![VSTS tricks.02.jpg]({{ '/images/VSTS+tricks.02.jpg' | absolute_url }})

  


**Результат**

  


![VSTS tricks.03.jpg]({{ '/images/VSTS+tricks.03.jpg' | absolute_url }})

  


Подробнее тут: <https://github.com/Microsoft/vsts-tasks/blob/master/docs/authoring/commands.md>