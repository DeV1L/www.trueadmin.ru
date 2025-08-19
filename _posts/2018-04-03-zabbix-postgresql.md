---
title: "Как сжать базу Zabbix (PostgreSQL)"
date: 2018-04-03
---

##  Введение

  
**Zabbix** прекрасная и, возможно, лучшая система мониторинга IT-инфраструктуры.  
Одно из его преимуществ - низкий порог вхождения. По сути - поставил сервер, агент и вперёд собирать метрики!  
  
К сожалению, у этой простоты есть и оборотная сторона. Далеко не всегда люди разбираются в системе, её параметрах и том, как правильно их настраивать.  
  
Одним из проявлений этого является проблема неконтролируемого распухания базы.  
  
Чаще всего это возникает при некорректно подобранных параметрах [элементов данных](https://www.zabbix.com/documentation/3.0/manual/config/items/item). Например: малый “Update interval” + большой “History storage period” + много хостов с такими элементами данных.  
  
Хуже всего, когда при этом ещё и не настроен (либо игнорируется) мониторинг свободного места на разделе с базой. Рано или поздно это приводит к такому результату.  
  


  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiF3kbOlybJGAn1u3tNw8PmCUPpA-d5rajMdx3vsXo12i0Ekw3SDM3zPRMdBhX6RDngdrvpD0CSmqU0FTGWL7xYpwlV9KkCOesBr9rIyaCIrJdwTnzWq7S6Owye1QpA1OyuoqURK_tuNDwf/s640/pgsql+disk+space+usage+1.jpg)](images/pgsql+disk+space+usage+1.jpg)

  
  
Для избежания подобной ситуации необходимо руководствоваться [рекомендациями](https://www.zabbix.com/documentation/3.0/manual/installation/requirements#database_size "https://www.zabbix.com/documentation/3.0/manual/installation/requirements#database_size") по планированию базы и не жадничать с глубиной хранения данных в истории (см. разницу между историей и трендами "[History and trends](https://www.zabbix.com/documentation/3.0/manual/config/items/history_and_trends)").  
  
Также необходимо учитывать, что добавление новых хостов в мониторинг **неизбежно** ведёт к увеличению размера базы. Таким образом требуется **иметь запас** свободного места на диске.  
  


##  Порядок уменьшения размера базы

  


Что же делать, если всё-таки база распухла больше, чем ................ ?  
Первое, что должно быть сделано - это выявление и исправление неоптимальных настроек.  
  


###  Оптимизация сбора и хранения данных

  


1) Найдите лишние хосты.  
Это могут быть хот  
Удалите или отключите их.  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEimsqaU7ywIV7wnCIFHocRZoPnuqvWpZ4L_pB0JqO3PRewP7sRXFAkx_2I6pFZW_vhTP6ws_bhLo9qPkJ83M1KPRkaWfIJrTIjFJY4UVq-CBeDVDjiM4MtPxDp4l4QwKwiXNN0DT9is5dE7/s400/disable+or+delete+host.jpg)](images/disable+or+delete+host.jpg)

  
  
2) Найдите лишние шаблоны.  
Возможно вы тестировали какие-то шаблоны с [Zabbix Share](https://share.zabbix.com/), возможно создавали шаблоны для какой-то конкретной уже не актуальной задачи. Возможно вы просто собираете много лишних данных :)  
 Удалите их. При это важно не забывать разницу между действиями "Delete" и "Delete and clear":  
  


  * Первое удалит лишь шаблон, оставив все айтемы и триггеры на хостах, к которым он был прилинкован;
  * Второе удалит и шаблон и данные



[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjmjojNBmR8wTbrwqasx8DZIBeMYqIUXewtjJ19eZT_RO_fd8gUpSf8Q4cXMfbSffNJyfhtM-PNiIATfMNpZyMG3JB3tzXRdQgGVD8JDsIPtMUn3J5pxHRsL_ACCQ-lNapIj1uTQrIElJ1o/s400/delete+template.jpg)](images/delete+template.jpg)

  


3) Проанализируйте все шаблоны и найдите криво настроенные айтемы. 

Сдесь нет однозначных критериев, руководствуйтесь логикой и [рекомендациями](https://www.zabbix.com/documentation/3.0/manual/installation/requirements#database_size "https://www.zabbix.com/documentation/3.0/manual/installation/requirements#database_size") по планированию базы.

  


Например, скриншот ниже.

  


  


[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi8Im1TuMKqpQBLuN1vpJP5E6JNdYXMZmoX3FcgzqXka_F1bjdOycAyMfY4QV3IIeh_FdWeed5TOgGXUWkR1wOvhg4eeZyC5EoiOP_kCQw1dytRBmjSLxiKAv7DIgTnQSL5VBjmwk7DyWuJ/s640/bad+template.jpg)](images/bad+template.jpg)

  
Конечно, возможно, вам действительно очень необходимо хранить данные по производительности памяти с двадцатисекундными интервалами в течении недели. Если так - будьте готовы заплатить за эторазмером базы. Если нет - просто уменьште эти параметры в несколько раз.  
  
4) Если совсем лень - можно не заморачиваться с отдельными шаблонами, а задать History и Trends интервалы на уровне всей системы.  
Для этого в [разделе](https://www.zabbix.com/documentation/3.4/manual/web_interface/frontend_sections/administration/general) Administration -> General -> Housekeeping необходимо задать соответствующие периоды и проставить чекбокс "Override ... period"  
  


[![](images/override+houskeeping.jpg)](images/override+houskeeping.jpg)

  


###  Уменьшение размера базы

После того, как найдены и устранены причины раздувания базы можно приступить к её "сдуванию" чтобы высвободить дисковое пространство. Для этого нужно сделать следующее.

  


  


**1) Подключиться к Postgress, выбрать базу zabbix**  

    
    
    -bash-4.2$psql
    
    postgres=# \c zabbix
    psql (9.2.18, server 9.6.4)
    
    
    

**  
****2) Определить размер базы**   

    
    
    zabbix=# SELECT pg_size_pretty(pg_database_size('zabbix'));
     pg_size_pretty
    ----------------
     95 GB
    (1 row)
    
    
    
    
    
    

**3) Определить размеры таблиц (первыее 20 по размеру)**   

    
    
    zabbix=# SELECT nspname || '.' || relname AS "relation",
        pg_size_pretty(pg_total_relation_size(C.oid)) AS "total_size"
      FROM pg_class C
      LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
      WHERE nspname NOT IN ('pg_catalog', 'information_schema')
        AND C.relkind <> 'i'
        AND nspname !~ '^pg_toast'
      ORDER BY pg_total_relation_size(C.oid) DESC
      LIMIT 20;
    
             relation          | total_size
    ---------------------------+------------
     public.history            | 59 GB
     public.history_uint       | 21 GB
     public.history_text       | 6532 MB
     public.trends             | 5257 MB
     public.trends_uint        | 2545 MB
     public.history_str        | 347 MB
     public.events             | 264 MB
     public.alerts             | 248 MB
     public.event_recovery     | 63 MB
     public.items              | 29 MB
     public.item_discovery     | 18 MB
     public.history_log        | 11 MB
     public.event_tag          | 8776 kB
     public.auditlog           | 7568 kB
     public.sessions           | 6864 kB
     public.problem            | 5088 kB
     public.items_applications | 4936 kB
     public.graphs             | 4088 kB
     public.triggers           | 3872 kB
     public.graphs_items       | 3528 kB
    (20 rows)
    
    
    

**  
****4) Удалить данные из таблиц, старше желаемого времени**   
  

    
    
    \set history_interval 7
    \set trends_interval 90
    
    zabbix=# DELETE FROM history where age(to_timestamp(history.clock))> (:history_interval * interval '1 day');
    zabbix=# DELETE FROM history_uint where age(to_timestamp(history_uint.clock))> (:history_interval * interval '1 day') ;
    zabbix=# DELETE FROM history_str where age(to_timestamp(history_str.clock))> (:history_interval * interval '1 day') ;
    zabbix=# DELETE FROM history_text where age(to_timestamp(history_text.clock))> (:history_interval * interval '1 day') ;
    zabbix=# DELETE FROM history_log where age(to_timestamp(history_log.clock))> (:history_interval * interval '1 day') ;
    zabbix=# DELETE FROM trends where age(to_timestamp(trends.clock))> (:trends_interval * interval '1 day');
    zabbix=# DELETE FROM trends_uint where age(to_timestamp(trends_uint.clock))> (:trends_interval * interval '1 day') ;

**  
**Больше полезных SQL запросов для Zabbix можно взять[ тут](https://github.com/DeV1L/zabbix-sql).  
**  
****  
****5) Выполнить сжатие базы**   

    
    
    zabbix=# VACUUM FULL;
    
    
    
    

**  
****6) Проверить размер базы после сжатия**   
  

    
    
    zabbix=# SELECT pg_size_pretty(pg_database_size('zabbix'));
     pg_size_pretty
    ----------------
     15 GB
    (1 row)
    
    
    

Ещё раз посмотрим топ-5 таблиц по размеру

  

    
    
             relation          | total_size
    ---------------------------+------------
     public.history            | 8521 MB
     public.history_uint       | 3508 MB
     public.history_text       | 1554 MB
     public.trends             | 1105 MB
     public.trends_uint        | 645 MB
    
    
    
    
    
    

Итого размер занимаемого дискового пространства уменьшлся более чем в **6 раз**.

[![](images/pgsql+disk+space+usage+2.jpg)](images/pgsql+disk+space+usage+2.jpg)
    
    
    
