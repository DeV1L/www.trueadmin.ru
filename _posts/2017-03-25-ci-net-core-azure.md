---
title: "CI: .NET Core + Azure"
date: 2017-03-25
layout: default
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

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiRtKq89YEobxFL-7mJ8hH6zUzChlzZdnYEPxQGf3Ejdg-npb1RJwrByr9mm02uAPbeDDiZAUb6fdID2aCxCj8y4OORdh-6Z9BN12UcR4IZle2MUhYJa2EpldPmTF7VLt-XYU2h0koDpczi/s640/new+project.jpg)](/images/new+project.jpg)

  


jj

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgEVyUOZ7qcz8UX6vt9QGupTc0sirO7W73rMQe98KUG7xfN7jr89yZptn9jn62L9U4exJF6dqz-qPDB8vJ42PNj1_Oobcap2TBLQhwJsHMmmfTBRSOXnzwZDx-QcEO_iC-0hKAxP5kutftn/s400/new+projec+create.jpg)](/images/new+projec+create.jpg)

  


В на открывшейся странице проекта выбираем "or import a repository" -> "Import"

Вставляем https://github.com/simplcommerce/SimplCommerce.git и запускаем импорт.

После окончания процесса переходим в раздел "Code" и _убеждаемся в корректности импорта репозитория_

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiNHNGH2Dn2b5U87-ouzAhyphenhyphenPbzgO30ZkdU5BgTi8wXEevGEGNmOZR-Pn3shP-ArEQIlRE47ISbXfFT-QWuGU5wqhTXuPKtPAEXeU4x-hDqLzqDovy7TaOUiFtm9lrkxGC6OUBAnoq_ZyWAk/s640/Code.jpg)](/images/Code.jpg)

  


##  Настройка агента

###  Создание виртуальной машины в Azure

Для простоты создадим новую виртуальную машину при помощи стандартного шаблона Microsoft. Для этого переходим по ссылке <https://github.com/Azure/azure-quickstart-templates/tree/master/docker-simple-on-ubuntu> и нажимаем "Deploy to Azure". 

По нажатию этой кнопки на Azure Portal откроется  страница настройки шаблона.

На ней необходимо заполнить основные параметры и нажать "Purchase".

После завершения деплоя всех компонент можно перейти в созданную ресурсную группу и преступить к настойке.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgmEAcYRfHOfsul3H_WqZKwsGGggMK2QbyOiMiI4j5M21vUoYIEGWs22oz_A5NsYmdEBB8PHdZFYXWTn9_bFIH4CKQyYowVwI-HgveZ6tYSLCtRQfiaDp-DMvPtbNaq3nENkFOvsHnh_aSR/s640/Deploy+Status.jpg)](/images/Deploy+Status.jpg)

  


!VM Size

  


dockerfortest.westeurope.cloudapp.azure.com  
  
  
  
  
  
  
После окончания деплоя, переходим к созданной ВМ.

  


###  Настройка Docker

Настраиваем SSL по инструкции <https://docs.docker.com/engine/security/https/>

  


  


##  Билд

По умолчанию в VSTS отсутствуют задачи для Docker, поэтому нам необходимо установить бесплатное официальное расширение. Для этого переходим в Marketplace по ссылке <https://go.microsoft.com/fwlink/?LinkId=797831> и в строке поиска вводим "Docker".

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj-cPzeAfBPO9LULU7AcP_BDn-FdnPF_Jbb6lihIRDcvGUF9mY1m0zFL2akAFfBR1Oo6N4aY2RIndoUILirACk7E8ZBRgbuHlh_HzoiLgJ1I03465WUZMR-q2ZQyJwiLl8irZ7-CYJBX6-j/s640/docker+extension.jpg)](/images/docker+extension.jpg)

  


Устанавливаем "Docker Integration" от Microsoft.

  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhK4jQYy-_TByo6qD7QbSu1uvIyaaVZNXtOdmga5cD31gOKtZjOUBZOPe4f5mjsZQ9Et4DTW8Cf1igpCMsHot8ePqCR2dnaRJDNkStL5zcDVgBsWnkbV0adgpJIqhNFjs_8C4T7CsEOfT-J/s640/docker+extension+install.jpg)](/images/docker+extension+install.jpg)

  


  


  


  


  


  


  

