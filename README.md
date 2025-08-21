# Домашнее задание к занятию 14 «Средство визуализации Grafana»


## Обязательные задания

### Задание 1

1. Используя директорию [help](./help) внутри этого домашнего задания, запустите связку prometheus-grafana.
2. Зайдите в веб-интерфейс grafana, используя авторизационные данные, указанные в манифесте docker-compose.
3. Подключите поднятый вами prometheus, как источник данных.
4. Решение домашнего задания — скриншот веб-интерфейса grafana со списком подключенных Datasource.

## Решение 1

1. запустим docker-compose.yml 
```
cd help
docker compose up -d
docker compose ps
```

![рисунок 1](https://github.com/ysatii/monitoring-hw3/blob/main/img/img1.jpg)

2. зайдем в веб интерфейс grafana
http://localhost:3000 
admin
admin

![рисунок 2](https://github.com/ysatii/monitoring-hw3/blob/main/img/img2.jpg)

3. Подключим поднятый вами prometheus, как источник данных.
Grafana → ⚙️ Configuration → Data sources → Add data source → Prometheus

URL: http://prometheus:9090 (имя сервиса из docker-compose в одной сети)

Save & Test — должна быть зелёная галочка

![рисунок 3](https://github.com/ysatii/monitoring-hw3/blob/main/img/img3.jpg)

4. скриншот веб-интерфейса grafana со списком подключенных Datasource
![рисунок 4](https://github.com/ysatii/monitoring-hw3/blob/main/img/img4.jpg)

## Задание 2

Изучите самостоятельно ресурсы:

1. [PromQL tutorial for beginners and humans](https://valyala.medium.com/promql-tutorial-for-beginners-9ab455142085).
2. [Understanding Machine CPU usage](https://www.robustperception.io/understanding-machine-cpu-usage).
3. [Introduction to PromQL, the Prometheus query language](https://grafana.com/blog/2020/02/04/introduction-to-promql-the-prometheus-query-language/).

Создайте Dashboard и в ней создайте Panels:

- утилизация CPU для nodeexporter (в процентах, 100-idle);
- CPULA 1/5/15;
- количество свободной оперативной памяти;
- количество места на файловой системе.

Для решения этого задания приведите promql-запросы для выдачи этих метрик, а также скриншот получившейся Dashboard.

## Решение 2
1. CPU Utilization, % 
![рисунок 5](https://github.com/ysatii/monitoring-hw3/blob/main/img/img5.jpg)

``` promsql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```  
Unit: percent  
Legend: {{instance}}  

2. Load Average 1/5/15 (одна панель, три серии)
![рисунок 6](https://github.com/ysatii/monitoring-hw3/blob/main/img/img6.jpg)

Query A = node_load1
Legend: load1 {{instance}}

Query B = node_load5
Legend: load5 {{instance}}


Query C = node_load15
Legend: load15 {{instance}}

3.  количество свободной оперативной памяти;
- Абсолютное значение (в байтах)

Query: node_memory_MemAvailable_bytes
Legend: {{instance}}
Unit: bytes (в Grafana → Panel → Field → Unit).

![рисунок 7](https://github.com/ysatii/monitoring-hw3/blob/main/img/img7.jpg)

-  Процент от общего объёма

Query: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
Legend: free RAM % {{instance}}
Unit: percent.

![рисунок 8](https://github.com/ysatii/monitoring-hw3/blob/main/img/img8.jpg)

4.  Свободное место на ФС  
— Свободно, GB  
Query A = 
```node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"} / (1024*1024*1024)
```
Legend: {{instance}} {{mountpoint}}  
Unit: gigabytes (GB)  
![рисунок 9](https://github.com/ysatii/monitoring-hw3/blob/main/img/img9.jpg)  

— Свободно, %  
Query A = 
```(node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"}  / node_filesystem_size_bytes{fstype!~"tmpfs|overlay| quashfs"}) * 100  
```
Legend: free % {{instance}} {{mountpoint}}  
Unit: percent (0-100)  
![рисунок 10](https://github.com/ysatii/monitoring-hw3/blob/main/img/img10.jpg)  


— Занято, %  
Query A = 
```
(1 - node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs"}) * 100  
```
Legend: used % {{instance}} {{mountpoint}}  
Unit: percent (0-100)  
![рисунок 11](https://github.com/ysatii/monitoring-hw3/blob/main/img/img11.jpg)  





## Задание 3

1. Создайте для каждой Dashboard подходящее правило alert — можно обратиться к первой лекции в блоке «Мониторинг».
2. В качестве решения задания приведите скриншот вашей итоговой Dashboard.

## Решение 3
- арлет для CPU Utilization, % 
  если средняя загрузка CPU >80% и держится 5 минут, алерт сработает → статус панели изменится (будет красный/жёлтый) и уйдёт уведомление в канал.
![рисунок 12](https://github.com/ysatii/monitoring-hw3/blob/main/img/img12.jpg)  
сообщение = CPU usage is above 80% on {{ $labels.instance }} for 5 minutes!

 проведем стресс тест 
 нагрузить все доступные ядра на 100% на 6 минут
```
sudo apt-get install stress-ng -y    # Debian/Ubuntu
stress-ng --cpu 0 --timeout 360
```
(--cpu 0 = все ядра)
![рисунок 15](https://github.com/ysatii/monitoring-hw3/blob/main/img/img15.jpg)  
это  подтверждает что рлет работает правильно



- RAM (свободно <10%)
  если свободной  памяти < 10% и держится 5 минут, алерт сработает → статус панели изменится (будет красный/жёлтый) и уйдёт уведомление в канал.
![рисунок 13](https://github.com/ysatii/monitoring-hw3/blob/main/img/img13.jpg)  

- Свободно, диска %
если диск свободен < 10% и держится 5 минут, алерт сработает → статус панели изменится (будет красный/жёлтый) и уйдёт уведомление в канал.
![рисунок 14](https://github.com/ysatii/monitoring-hw3/blob/main/img/img14.jpg)  


## Задание 4

1. Сохраните ваш Dashboard.Для этого перейдите в настройки Dashboard, выберите в боковом меню «JSON MODEL». Далее скопируйте отображаемое json-содержимое в отдельный файл и сохраните его.
1. В качестве решения задания приведите листинг этого файла.

---

## Задание повышенной сложности

**При решении задания 1** не используйте директорию [help](./help) для сборки проекта. Самостоятельно разверните grafana, где в роли источника данных будет выступать prometheus, а сборщиком данных будет node-exporter:

- grafana;
- prometheus-server;
- prometheus node-exporter.

За дополнительными материалами можете обратиться в официальную документацию grafana и prometheus.

В решении к домашнему заданию также приведите все конфигурации, скрипты, манифесты, которые вы 
использовали в процессе решения задания.

**При решении задания 3** вы должны самостоятельно завести удобный для вас канал нотификации, например, Telegram или email, и отправить туда тестовые события.

В решении приведите скриншоты тестовых событий из каналов нотификаций.


### Как оформить решение задания

Выполненное домашнее задание пришлите в виде ссылки на .md-файл в вашем репозитории.

---
