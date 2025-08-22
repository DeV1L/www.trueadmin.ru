---
title: "CI: .NET Core + Azure"
date: 2017-03-25
---

Введение  
  
Для примера возьмём https://github.com/simplcommerce/SimplCommerce (SQL Server)  
Для простоты будем собирать и деплоить Docker образ на одной и той же виртуальной машине в Azure.  
  
Requirenments  
  


  1. Учётная запись Microsoft
  2. _Аккаунт_  Microsoft Azure
  3. Аккаунт Visual Studio Team Services

  


##    
Оглавление

  
  


  1. Team Project
  2. Настройка агента


  * Создание виртуальной машины в Azure
  * Доступ, порты
  * Настройка Docker


  1. Настройка Azure Container Service ?
  2. Билд
  3. Релиз

  
  
  


##  Создадим Team Project

![new project.jpg]({{ '/images/new+project.jpg' | absolute_url }})

  


jj

  


![new projec create.jpg]({{ '/images/new+projec+create.jpg' | absolute_url }})

  


В на открывшейся странице проекта выбираем "or import a repository" -> "Import"

Вставляем https://github.com/simplcommerce/SimplCommerce.git и запускаем импорт.

После окончания процесса переходим в раздел "Code" и _убеждаемся в корректности импорта репозитория_

![Code.jpg]({{ '/images/Code.jpg' | absolute_url }})

  


##  Настройка агента

###  Создание виртуальной машины в Azure

Для простоты создадим новую виртуальную машину при помощи стандартного шаблона Microsoft. Для этого переходим по ссылке <https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu> и нажимаем "Deploy to Azure". 

По нажатию этой кнопки на Azure Portal откроется  страница настройки шаблона.

На ней необходимо заполнить основные параметры и нажать "Purchase".

После завершения деплоя всех компонент можно перейти в созданную ресурсную группу и преступить к настойке.

![Deploy Status.jpg]({{ '/images/Deploy+Status.jpg' | absolute_url }})

  


!VM Size

  


dockerfortest.westeurope.cloudapp.azure.com  
  
  
  
  
  
  
После окончания деплоя, переходим к созданной ВМ.

  


###  Настройка Docker

Настраиваем SSL по инструкции <https://docs.docker.com/engine/security/https/>

  


  


##  Билд

По умолчанию в VSTS отсутствуют задачи для Docker, поэтому нам необходимо установить бесплатное официальное расширение. Для этого переходим в Marketplace по ссылке <https://go.microsoft.com/fwlink/?LinkId=797831> и в строке поиска вводим "Docker".

![docker extension.jpg]({{ '/images/docker+extension.jpg' | absolute_url }})

  


Устанавливаем "Docker Integration" от Microsoft.

  


![docker extension install.jpg]({{ '/images/docker+extension+install.jpg' | absolute_url }})