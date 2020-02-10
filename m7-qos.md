# M7 QoS

## Простая очередь

### Введение

* QoS (quality of service) - это искусство управления ресурсами полосы пропускания, а не просто "слепое" ограничение полосы пропускания определенными узлами.
* QoS может определять приоритетность трафика на основе метрик. Полезно для
  - Критически важных приложений
  - Чувствительному трафику, такой как голосовые и видеопотоки
* Простые очереди-это... простой ... способ ограничить пропускную способность до
  - Загрузки клиента (исходящей)
  - Закачки клиента (входящей)
  - Совокупно для клиента (загрузка и закачка)

### Цель

* Цель, к которой применяется простая очередь
* Цель должна быть определена. Это может быть
  - IP-адрес
  - Подсеть
  - Взаимодействие

* Порядок очередей очень важен. Каждый пакет должен пройти через каждую простую очередь, пока не произойдет совпадение

### Назначения

* IP-адрес, на который направлен целевой трафик, или
* Интерфейс, через который будет проходить целевой трафик;
* Не обязательно, как поле "цель"
* Может использоваться для ограничения очереди

### Max-limit и limit-at

* Параметр "max-limit" - это максимальная скорость передачи данных, которую может достичь цель
  - Рассматривается как MIR _(maximum information rate - максимальная скорость передачи)_
  - лучший вариант развития событий
* Параметр "limit-at" является гарантированной минимальной скоростью передачи данных для цели
  - Рассматривается как CIR _(committed information rate - совершенная скорость передачи информации)_
  - Худший вариант развития событий

### Bursting

* Bursting позволяет пользователям на короткое время получить большую пропускную способность, чем позволяет параметр "max-limit".
* Полезно для увеличения трафика, который не использует пропускную способность слишком часто. Например, HTTP. Получите быструю загрузку страницы, затем прочитайте ее за несколько секунд.

* Определения
  - **Burst-limit** : максимальная скорость передачи данных, в то время как burst разрешен.
  - **Burst-time** : время в секундах, в течение которого производится выборка. Это НЕ период, в течение которого трафик будет burst.
  - **Burst-threshold** : значение, которое определит, будет ли пользователю разрешен burst
  - **Average-rate** : среднее значение передачи данных, рассчитанное в 1/16 части "burst-time".
  - **Actual-rate** : Текущая (реальная) скорость передачи данных.

* Как это работает.
  - Bursting разрешен, пока **average-rate** остается ниже **burst-threshold**.
  - Bursting будет ограничен по скорости, указанной **burst-limit**.
  - **Average-rate** вычисляется путем усреднения 16 выборок (**actual-rate**) в течение **burst-time** секунд.
    - Если **burst-time** составляет 16 секунд, то образец берется каждую секунду.
    - Если **burst-time** составляет 8 секунд, то образец берется каждые ½ секунды. И так далее…
    - Когда начнется bursting, он будет разрешен на **longest-burst-time** секунд, что составляет
      - (burst-threshold x burst-time) / burst-limit.

![С burst-time 16 секунд](/pics/m7_bursting_16.png)

_С burst-time 16 секунд_

![С burst-time 8 секунд](/pics/m7_bursting_16.jpeg)

_С burst-time 8 секунд_

#### Синтаксис

* Простая очередь
  - `add max-limit=2M/2M name=queue1 target=192.168.3.0/24`
* The same queue with bursting
  - `add burst-limit=4M/4M burst-threshold=1500k/1500k burst-time=8s/8s limit-at=\`
  `1M/1M max-limit=2M/2M name=queue1 target=192.168.3.0/24`

**Подсказка**

* Возможно, вы заметили, что значки очереди меняют цвет в зависимости от использования. Цвет имеет значение.
  - Зеленый: 0-50% от используемой полосы пропускания
  - Желтый: 51-75% используется от доступной полосы пропускания используется
  - Красный: 76-100% от доступной полосы пропускания

## Одна простая очередь для всей сети (PCQ)

**Зачем иметь очередь на всех?**

* Per Connection Queue (PCQ) - это динамический способ формирования трафика для нескольких пользователей с использованием более простой конфигурации.
* Определите параметры, тогда каждый подпоток (например, определенные IP-адреса) будет иметь те же ограничения.

**Конфигурация Pcq-rate**

* Параметр **pcq-rate** ограничивает допустимую скорость передачи данных типа очереди.
* Классификатор - это то, что маршрутизатор проверяет, чтобы увидеть, как он будет применять это ограничение. Это может быть адрес источника или назначения, или порт источника или назначения. Таким образом, можно ограничить трафик пользователей или приложений (например, HTTP).

**Конфигурации Pcq-limit**

* Этот параметр измеряется в пакетах.

* Большое значение pcq-limit
  - Создаст больший буфер, таким образом уменьшая отброшенные пакеты
  - Увеличится задержка

* Малое значение pcq-limit
  - Увеличит количество отбрасываемых пакетов (так как буфер меньше) и заставит источник повторно отправить пакет, тем самым сократив задержку
  - Приведет к корректировке размера окна TCP, сообщая источнику уменьшить скорость передачи

* Какое значение я должен использовать? Здесь нет простого ответа.
  - Если часто начинается на основе "проб и ошибок" для каждого приложения
  - Если пользователи жалуются на задержку, уменьшите значение pcq-limit (длина очереди)
  - Если пакеты должны проходить через сложный брандмауэр, то вам, возможно, придется увеличить длину очереди, так как это может привести к задержкам
  - Быстрые интерфейсы (как Gig) требуют меньших очередей, поскольку они уменьшают задержки

### PCQ, пример

* Предположим, что у нас есть пользователи, совместно использующие ограниченное WAN-соединение. Мы дадим им следующие скорости передачи данных:
  - Скачивание: 2 Мбит/с
  - Отдача : 1 Мбит/с
  - WAN находится на ether1
  - Подсеть локальной сети 192.168.3.0/24

**/ip firewall mangle**
```
add action=mark-packet chain=forward new-packet-mark=client_upload \
  out-interface=ether1 src-address=192.168.3.0/24

add action=mark-packet chain=forward dst-address=192.168.3.0/24 \
  in-interface=ether1 new-packet-mark=client_download
```
**/queue type**
```
add kind=pcq name=PCQ_download pcq-classifier=dst-address pcq-rate=2M \
add kind=pcq name=PCQ\_upload pcq-classifier=src-address pcq-rate=1M
```
**queue tree**
```
add name=queue_upload packet-mark=client_upload parent=global queue=\
  PCQ_upload

add name=queue_download packet-mark=client_download parent=global queue=\
  PCQ_download
```

Наш пример объяснил

* **Mangle**: мы говорим маршрутизатору помечать пакеты с помощью "_**client_upload**_" или "_**client_download**_", в зависимости от того, если
  - Пакеты поступают из локальной сети и уходят из ether1 (upload - отдача) или,
  - Пакеты поступают из ether1 и отправляются в локальную сеть (загрузка).

* **Queue types**: мы определяем скорости передачи данных и классификаторы, используемые для дифференциации подпотоков (источник или назначение)
* **Queue tree**: комбинации, которые проверяются, чтобы увидеть, подходят ли пакеты для формирования трафика и что применять.
  - Например, в случае загруженного трафика мы проверяем входные и выходные интерфейсы (**global**) для пакетов с "**client_upload**" и применяем тип очереди "**PCQ_upload**".

## Мониторинг

Мониторинг траффика интерфейса

![](/pics/m7_monitor.jpeg)

Средство мониторинга трафика используется для запуска сценариев, когда трафик интерфейса достигает определенного порогового значения.

**Пример**

**/tool traffic-monitor**
```
add interface=ether1 name=TrafficMon1 on-event=script1 threshold=1500000 \
  traffic=received
```
**/system script**
```
add name=script1 policy=ftp,read,test,winbox,api source="/tool e-mail send to=\"\
  YOU@DOMAIN.CA\" subject=([/system identity get name] . \"Log \
  \" . [/system clock get date\]\) body=\"Hello World. You're going too fast!\""
```

### Torch

* Torch is a real-time traffic monitoring tool that can be used to monitor the traffic going through an interface.
* Although CLI is VERY flexible, the Torch interface in Winbox is very intuitive.

Torch, CLI

| \[admin@Pod3\] /tool&gt; torch interface=ether2 | port=winbox |  |  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| SRC-PORT | DST-PORT | TX | RX TX-PACKETS RX-PACKETS |  |  |  |  |
| 53217 | 8291 \(winbox\) | 12.0kbps | 4.7kbps | 7 | 6 |  |  |
|  |  | 12.0kbps | 4.7kbps | 7 | 6 |  |  |
| \[admin@Pod3\] /tool&gt; torch interface=ether2 | port=any |  |  |  |  |  |  |
| SRC-PORT | DST-PORT |  | TX | RX TX- |  |  |  |
| PACKETS RX-PACK |  |  |  |  |  |  |  |
| 53217 | 8291 \(winbox\) |  | 15.2kbps | 5.1kbps |  |  |  |
| 7 |  |  |  |  |  |  |  |
| 62414 | 53 | \(dns\) |  | 728bps | 600bps |  |  |
| 1 |  |  |  |  |  |  |  |
| 53538 | 80 | \(http\) |  | 92.8kbps | 5.3kbps |  |  |
| 12 |  |  |  |  |  |  |  |
| 62437 | 53 | \(dns\) |  | 744bps | 616bps |  |  |
| 1 |  |  |  |  |  |  |  |
| 53540 | 80 | \(http\) |  | 182.2kbps | 8.4kbps |  |  |
| 18 |  |  |  |  |  |  |  |
| 53541 | 80 | \(http\) |  | 191.1kbps | 8.6kbps |  |  |
| 19 |  |  |  |  |  |  |  |
| 59150 | 53 | \(dns\) |  | 760bps | 632bps |  |  |
| 1 |  |  |  |  |  |  |  |
| 53542 | 80 | \(http\) |  | 112.9kbps | 7.0kbps |  |  |
| 12 |  |  |  |  |  |  |  |
| 53543 | 443 \(https\) |  | 34.8kbps | 6.3kbps |  |  |  |

Torch, Winbox

![](.gitbook/assets/3%20%283%29.jpeg)

Graphs

* Graphing is a tool used to monitor various RouterOS parameters over time and put the collected data in graphs.
* The following parameters can be captured.

– Voltage and temperature

– CPU, memory and disk usage

– Interface traffic

– Queue traffic

* Graphs can be accessed by typing **http://&lt;router-IP-address&gt;/graphs**

Graphs

First steps

\[admin@Pod3\] /tool graphing&gt; set store-every=5min page-refresh=300 \[admin@Pod3\] /tool graphing&gt; print

store-every: 5min

page-refresh: 300

\[admin@Pod3\] /tool graphing&gt;

Then we add values to be graphed.

\[admin@Pod3\] /tool graphing&gt; interface add allow-address=0.0.0.0/0 interface=all \[admin@Pod3\] /tool graphing&gt; queue add allow-address=0.0.0.0/0 simple-queue=test-queue1 \[admin@Pod3\] /tool graphing&gt; resource add

Graphs

![](.gitbook/assets/4.png)

SNMP

* SNMP, which stands for Simple Network Management Protocol, is an Internet-standard protocol used for managing devices on IP networks.
* Many tools, both open source and commercial, are available to manage your networks and automate many tasks.
* Like all things, configuration must be thought out since one could use SNMP to hack your network.

SNMP

First steps.

\[admin@Pod3\] /snmp&gt; set enabled=yes

\[admin@Pod3\] /snmp&gt; set contact=YOU

\[admin@Pod3\] /snmp&gt; set location=OFFICE

\[admin@Pod3\] /snmp&gt; print

enabled: yes

contact: YOU

location: OFFICE

engine-id:

trap-target:

trap-community: \(unknown\)

trap-version: 1

trap-generators:

\[admin@Pod3\] /snmp&gt;

SNMP

* Special attention should be given to communities.
* They dictate privileges.

\[admin@Pod3\] /snmp community&gt; print detail

Flags: \* - default

0 \* name="public" addresses=0.0.0.0/0 security=none read-access=yes write-access=no authentication-protocol=MD5 encryption-protocol=DES authentication-password="" encryption-password=""

\[admin@Pod3\] /snmp community&gt;

SNMP

![](.gitbook/assets/5%20%284%29.jpeg)

Time for a practical exercise

**End of module 7**

Laboratory

* Goals of the lab

– Setting up and testing a simple queue.

– Setting up and testing a PCQ based queuing configuration.

– Being able to tell the pros and cons of both.

– Test out monitoring tools and see how they can help in everyday situations.

Laboratory : Setup

![](.gitbook/assets/6%20%282%29.jpeg)

Laboratory : step 1

* Before going any further, install a MIB browser of your computers.
* Also, pods should pair up for this lab as many steps will require that more than one computer be connected to the routers.

Laboratory : step 2

* Test throughput using a speed testing web site. Note the results.
* Configure a simple queue \(call it "lab7"\) that will limit your entire LAN to 4Mbps download and 2Mbps upload.
* Test throughput again.
* Ask a fellow student to plug into your router and repeat the speed test. What do you get? Does your fellow student get the same results when you connect to his router?

Laboratory : step 3

* Add bursting in the "lab7" queue. Parameters are :

– Burst limit 4M \(upload\), 6M \(download\)

– Burst-threshold 3M \(upload\), 5M \(download\)

– Burst-time 16 seconds for both

* Repeat the same tests as before and view results.
* Once done, disable the simple queue.

Laboratory : step 4

* Create a PCQ based system so that all computers on the same LAN have a limit of 4Mbps for downloads and 2Mbps for uploads.
* Make sure that the names that you use are meaningful!
* Test throughput using a speed testing web site. Note the results.
* Ask a fellow student to plug into your router and repeat the speed test. What do you get? Does your fellow student get the same results when you connect to his router?

Laboratory : step 5

* Configure traffic monitoring in such a way that it will send you an e-mail if inbound traffic exceeds 3Mbps on your wireless interface.

Laboratory : step 6

* Use the torch tool in such a way that you can see the source address of nodes doing any IP traffic on any port through the wireless interface.

– Experiment with the CLI and Winbox approaches.

Laboratory : step 7

* Enable graphs on :

– Wireless interface

– Hardware resources

* View them on your browser

Laboratory : step 8

* Enable SNMP, and supply these parameters :

– Your name as contact info.

– Your pod number as location \(Podx\).

– Keep the rest at default value.

* Using a MIB Browser, walk through your router's MIBs. Can you see your name and location?

Laboratory : step 9

* As usual, save the current configuration in binary and text format using the same name format that has been used in previous labs.
