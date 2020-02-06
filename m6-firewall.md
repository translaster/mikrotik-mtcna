# M6 Firewall

## Принципы брандмауэра (фаерволла)

* Брандмауэр - это служба, которая пропускает или блокирует пакеты данных, идущие к нему или через него на основе определенных пользователем правил.
* Брандмауэр действует как барьер между двумя сетями.
* Общий пример - ваша локальная сеть (доверенная) и Интернет (не доверенная).

### Как работает брандмауэр

* Брандмауэр работает по правилам. Они состоят из двух частей
  - Совпадение: _условия, которые нужны для совпадения_
  - Действие: _что я буду делать, когда у меня будет совпадение_
* Сопоставитель смотрит на такие параметры, как:
  - MAC-адрес источника
  - IP-адреса (сеть или список) и типы адресов (широковещательный, локальный, многоадресный, одноадресный)
  - Порт или диапазон портов
  - Протокол
  - Параметры протокола (поля типа и кода ICMP, флаги TCP, параметры IP)
  - Интерфейс: пакет приходит от или через
  * Байт DSCP
  * _**И многое другое…**_

### Потоки пакетов

* MikroTik создал диаграммы потока пакетов, чтобы помочь нам в создании более продвинутых конфигураций.
* Хорошо быть знакомым с ними, чтобы знать, что происходит с пакетами и в каком порядке
* Для этого курса мы будем держать его простым

* Общие диаграммы

![](/pics/m6_diagram_1.jpeg)

![](/pics/m6_diagram_2.jpeg)

![](/pics/m6_diagram_3.jpeg)

#### Потоки пакетов, пример*

* Сложно? Добро пожаловать в клуб!
* Этот следующий пример может помочь проиллюстрировать простой поток пакетов: Pinging a (несуществующий узел) на интерфейсе локальной сети маршрутизатора через его интерфейс WAN
  - IP узла, выполняющего пинг : 172.16.2.100
  - IP-адрес узла, который пингуется : 192.168.3.2
  - IP WAN маршрутизатора (ether1): 192.168.0.3

**Ping in**

```text
===PREROUTING===
Mangle-prerouting prerouting: in:ether1 out:(none), src-mac d4:ca:6d:33:b5:ef, proto ICMP (type 8, code 0),
172.16.2.100->192.168.3.2, len 60
dstnat dstnat: in:ether1 out:(none), src-mac d4:ca:6d:33:b5:ef, proto ICMP (type 8, code 0), 172.16.2.100-
>192.168.3.2, len 60
===FORWARD===
Mangle-forward forward: in:ether1 out:Bridge-PC, src-mac d4:ca:6d:33:b5:ef, proto ICMP (type 8, code 0),
172.16.2.100->192.168.3.2, len 60
Filter-forward forward: in:ether1 out:Bridge-PC, src-mac d4:ca:6d:33:b5:ef, proto ICMP (type 8, code 0),
172.16.2.100->192.168.3.2, len 60
===POSTROUTING===
Mangle-postrouting postrouting: in:(none) out:Bridge-PC, src-mac d4:ca:6d:33:b5:ef, proto ICMP (type 8, code 0),
172.16.2.100->192.168.3.2, len 60
srcnat srcnat: in:(none) out:Bridge-PC, src-mac d4:ca:6d:33:b5:ef, proto ICMP (type 8, code 0), 172.16.2.100-
>192.168.3.2, len 60
```

**Reply out**

```text
===OUTPUT===
Mangle-output output: in:(none) out:ether1, proto ICMP (type 3, code 1), 192.168.0.3->172.16.2.100, len 88
Filter-output output: in:(none) out:ether1, proto ICMP (type 3, code 1), 192.168.0.3->172.16.2.100, len 88
===POSTROUTING===
Mangle-postrouting postrouting: in:(none) out:ether1, proto ICMP (type 3, code 1), 192.168.0.3->172.16.2.100, len 88
```

#### Потоки пакетов, объяснение примера

```text
/ip firewall filter
add action=log chain=input log-prefix=Filter-input protocol=icmp
add action=log chain=output log-prefix=Filter-output protocol=icmp
add action=log chain=forward log-prefix=Filter-forward protocol=icmp
/ip firewall mangle
add action=log chain=prerouting log-prefix=Mangle-prerouting protocol=icmp
add action=log chain=output log-prefix=Mangle-output protocol=icmp
add action=log chain=input log-prefix=Mangle-input protocol=icmp
add action=log chain=forward log-prefix=Mangle-forward protocol=icmp
add action=log chain=postrouting log-prefix=Mangle-postrouting protocol=icmp
/ip firewall nat
add action=log chain=srcnat log-prefix=srcnat protocol=icmp
add action=log chain=dstnat log-prefix=dstnat protocol=icmp
```

#### Отслеживание соединений и состояний

* Отслеживание соединений управляет информацией обо всех активных соединениях.
* Прежде чем создавать фильтры брандмауэра (или правила), полезно знать какой трафик проходит через ваш маршрутизатор. Отслеживание соединений покажет вам именно это.

```
1 ospf 172.16.0.6 224.0.0.5 5m49s
2 SA tcp 172.16.2.100:49164 172.16.9.254:445 established 23h42m51s
3 SA tcp 172.16.2.122:61739 206.53.159.211:443 established 23h44m8s
4 SA tcp 172.16.2.130:58171 17.149.36.108:443 established 23h43m41s
5 SA gre 172.16.0.254 172.16.0.1 4h44m11s
6 SA udp 172.16.0.254:4569 209.217.98.158:4569 13m9s
7 SA tcp 172.16.2.130:58174 173.252.103.16:443 established 23h42m40s
8 SA tcp 172.16.2.140:52032 69.171.235.48:443 established 23h43m27s
9 SA tcp 172.16.2.107:47318 173.252.79.23:443 established 23h43m26s
10 SA tcp 172.16.2.102:57632 173.252.102.241:443 established 23h44m15s
11 ospf 172.16.0.5 224.0.0.5 5m49s
12 SA tcp 172.16.2.102:56774 65.54.167.16:12350 established 23h35m28s
13 SA tcp 172.16.2.102:56960 173.194.76.125:5222 established 23h43m57s
14 SA tcp 172.16.0.254:37467 172.16.0.1:1723 established 4h44m11s
15 SA tcp 172.16.2.107:39374 79.125.114.47:5223 established 23h29m1s
```

* Если вы отключите отслеживание по какой-либо причине, следующие функции не будут работать:
  - NAT
  - Брандмауэр
    - connection-bytes connection-mark
    - connection-type connection-state
    - connection-limit connection-rate
    - layer7-protocol p2p
    - new-connection-mark tarpit
  - p2p matching in simple queues
* Прежде чем отключить отслеживание подключений, удостоверьтесь в том, какую цель вы хотите достичь!

Состояния соединения являются _(предполагая, что клиент-A инициирует соединение с клиентом-B)_:

| Состояние | Описание |
| :-------- | :------- |
| Established | Устанавливается сеанс TCP для удаленного хоста, обеспечивающий открытое соединение, в котором можно обмениваться данными |
| Time-wait | Время, потраченное на ожидание, чтобы убедиться, что удаленный хост получил подтверждение запроса на завершение соединения (после "close") |
| Close | Представляет собой ожидание запроса на завершение соединения от удаленного компьютера. |
| Syn-sent | Клиент-A ожидает соответствующего запроса на соединение после его отправки |
| Syn-received | Клиент-B ожидает подтверждения запроса на соединение после получения и отправки запроса на соединение. |

* Использование отслеживания соединений позволяет отслеживать соединения UDP, даже если UDP не имеет состояния. Таким образом, брандмауэр MikroTik может фильтровать по UDP "состояниям".

## Структура: цепочки и действия

* Цепочка - это группировка правил, основанная на одних и тех же критериях. Существует три цепочки по умолчанию, основанные на предопределенных критериях.
  - Input: _Трафик идущий к маршрутизатору_
  - Forward: _Трафик идущий через маршрутизатор_
  - Output: _Трафик, исходящий от маршрутизатора_

* Вы можете иметь пользовательские цепочки, основанные на пользовательских критериях. Например:
  - _Весь icmp-трафик_
  - _Трафик приходящий из Ether2 и идущий к интерфейсу моста "LAN"_

* Определенные пользователем цепочки создаются путем выбора желаемых "сопоставлений" и выбора действия "прыжок". Вы дадите определяемой пользователем цепочке имя в поле "jump target" (цель прыжка).
  - После этого вы можете начать создавать правила фильтрации, используя новую цепочку, введя ее в поле "Chain" нового фильтра брандмауэра.
* Действие определяет, что будет делать фильтр, когда пакеты будут сопоставлены с ним.
* Пакеты последовательно проверяются на соответствие существующим правилам в текущей цепочке брандмауэра до тех пор, пока не произойдет совпадение. Когда это происходит, это правило применяется.
* Знайте, что определенные действия могут потребовать или не потребовать дальнейшей обработки пакета.
* Другие действия могут потребовать дальнейшей обработки пакета в другой цепочке. Мы увидим это на последующих страницах.

### Фильтры брандмауэра в действии

Основная философия безопасности

* Вы можете подходить к безопасности по-разному.
  - Мы доверяем внутреннему миру, правила влияют на то, что приходит извне.
  - Мы блокируем все и разрешаем то, о чем договариваемся.
  - Мы разрешаем все и блокируем то, что, как мы знаем, подозрительно.

Основные советы и рекомендации

* Перед настройкой или изменением правил активируйте "safe mode".
* После настройки или изменения правил протестируйте их с помощью такого инструмента, как ShieldsUP.(https://www.grc.com/x/ne.dll?bh0bkyd2)
  - Это даст вам отчет о слабых сторонах.
* Прежде чем вы начнете, установите политику.
* Запишите простым текстом на своем языке основные правила, которые вам нужны.
  - Как только вы поймете их и согласитесь с ними, введите их в маршрутизатор.
* Постепенно добавляйте другие правила, как только вы будете удовлетворены основными из них.
  - Если вы новичок в безопасности, это не поможет вам стрелять во всех направлениях. Делайте основы, но делайте их хорошо.
  - Просто не ждите слишком долго, чтобы добавить следующие правила. Одно дело - хорошо работать, но совсем другое - оставлять дыры открытыми, потому что вы хотите проверить первые правила.
* Это хорошая идея, чтобы закончить ваши цепочки правилом "поймать все" и посмотреть, что вы, возможно, пропустили.
* Вам понадобятся два правила "поймать все", одно для "регистрации" и одно для "отбрасывания" несовпадающего трафика. Оба должны быть основаны на одних и тех же сопоставлениях, чтобы быть полезными для вас.
* Как только вы увидите, что достигает правил "catch-all", вы можете добавить новые правила, основанные на желаемом поведении брандмауэра.

### Фильтр соответствий

* Перед выполнением "действия" над пакетом его необходимо идентифицировать.
* Совпадений много!

![](/pics/m6_firewall_rule.png)

### Действия фильтра

* Как только пакет сопоставлен с правилом, к нему будет применено действие.
* Фильтры брандмауэра MikroTik имеют 10 действий.

|            |                      |
| :--------- | :------------------- |
| **Accept** | Принимает пакет. Пакет не передается следующему правилу брандмауэра. |
| **Add-dst-to-address-list** | Добавляет адрес получателя в список адресов, указанный параметром address-list. Пакет передается следующему правилу брандмауэра. |
| **Add-src-to-address-list** | Добавить адрес источника в список адресов, указанный параметром address-list. Пакет передается следующему правилу брандмауэра. |
| **Drop** | Молча отбрасывает пакет. Пакет не передается следующему правилу брандмауэра. |
| **Jump** | Перейти к пользовательской цепочке, заданной значением параметра jump-target. Пакет передается следующему правилу брандмауэра (в пользовательской цепочке). |
| **Log** | Добавьте сообщение в системный журнал, содержащее следующие данные: _**in-interface**_, _**out-interface**_, _**src-mac**_, _**protocol**_, _**src-ip:port->dst-ip:port**_ and _**length of the packet**_ (длина пакета). Пакет передается в следующее правило брандмауэра. |
| **Passthrough** | Проигнорируйте это правило и перейдите к следующему (полезно для статистики). |
| **Reject** | Отбросить пакет и отправить сообщение ICMP reject. Пакет не передается следующему правилу брандмауэра. |
| **Return** | Передать управление обратно в цепочку, откуда произошел прыжок. Пакет передается следующему правилу брандмауэра (в исходной цепочке, если не было предыдущего совпадения для остановки анализа пакетов). |
| **Tarpit** | Захват и удержание TCP-соединений (ответы с помощью SYN/ACK на входящий пакет TCP SYN). Пакет не передается следующему правилу брандмауэра. |

**Защита вашего маршрутизатора (вход)**

* Входная цепочка смотрит на трафик, направленный на маршрутизатор.
* Правила, которые вы добавляете во входную цепочку, должны предотвращать доступ хакеров к маршрутизатору, не мешая ему выполнять свою работу.

**Защита вашего маршрутизатора (пример)**

* Ниже приведены предложения!
  - Предположим, что _ether01_ подключен к WAN (ненадежной сети), и мы используем политику "доверяй внутреннему".
    - Принимайте ICMP эхо-ответы (_Возможно, вы захотите проверить связь с сервером в Интернете. Это было бы полезно для вас, чтобы получать ответы!_)
    - Отбрасывайте эхо-запросы icmp (_Вы не хотите чтобы другие пинговали вас. Оставайтесь под радаром!_)
    - Принимайте весь "established" и "related" входящий трафик (_Вам понадобятся ответы на все запросы маршрутизатора, такие как запросы NTP и DNS_)
    - Отбрасывайте весь "invalid" входящий трафик (_Когда маршрутизатор получает то, что он не запрашивал_)
    - Протоколируйте остальной входящий трафик (_Я пропустил что-нибудь важное?_)
    - Отбрасывайте остальной входящий трафик (_Я хочу быть в безопасности!_)

**Защита ваших клиентов (forward)**

* Как указывалось ранее, цепочка forward смотрит на трафик, проходящий через маршрутизатор.
* Правила, которые вы добавляете в цепочку forward, должны препятствовать проникновению хакеров в вашу "безопасную" сеть, не мешая вам выполнять свою работу.

**Защита ваших клиентов (пример)**

* Ниже приведены предложения!
  - Опять же, предположим, что _ether01_ подключен к WAN (ненадежной сети), и мы используем политику "доверяй внутреннему".
    - Принимаем все "established" и "related" проходящий трафик (_вам понадобятся ответы на все, что вы запросили, например HTTP и запросы электронной почты_)
    - Отбрасываем весь "invalid" проходящий трафик (_Все, что вы получаете, вы не запрашивали_)
    - Протоколируем остаток прямого трафика (_Я пропустил что-нибудь важное?_)
    - Отбросим остальную часть проходящего трафика (_Я хочу быть в безопасности!_)

**Как это выглядит в конце концов**

![](/pics/m6_firewall.jpeg)

**Firewall filter syntax**

* View existing filter rules
  - /ip firewall filter print _(produces a clearer, readable output)_
  - /ip firewall filter export _(shows complete syntax)_

* Create various rules _(from /ip firewall filter)_
  - add chain=input comment="Established-Related (in)" connection-state=established in-interface=ether01
  - add chain=forward comment="Established-Related (fwd)" connection-state=established in-interface=ether01
  - add action=log chain=input comment="===CATCH-ALL==" in-interface=ether01 log-prefix="CATCH-ALL(in)"
  - add action=drop chain=input in-interface=ether01
  - add action=add-dst-to-address-list address-list=temp-list address-list-timeout=3d1h1m1s chain=input protocol=tcp src-address=172.16.2.0/24

## Basic address-list

* Address lists are groups of IP addresses
* They can be used to simplify filter rules
  - For example, you could create 100 rules to block 100 addresses, or!!
  - You could create one group with those 100 addresses and create only one filter rule.
* The groups \(address lists\) can represent
  - IT Admins with special rights
  - Hackers
  - Anything else you can think of…

* They can be used in firewall filters, mangle and NAT facilities.
* Creation of address lists can be automated by using **add-src-to-address-list** or **add-dst-to-address-list** actions in the firewall filter, mangle or NAT facilities.
  - This is a great way of automatically blocking IP addresses without having to enter them one by one
  - Example : add action=add-src-to-address-list address-list=BLACKLIST chain=input comment=psd in-interface=ether1-Internet psd=21,3s,3,1

Address list syntax

* View existing address lists
  - /ip firewall address-list print
* Create a permanent address list
  - /ip firewall address-list add address=1.2.3.4 list=hackers
* Create an address list through a firewall filter rule
  - /ip firewall filter add action=add-dst-to-address-list address-list=temp-list address-list-timeout=3d1h1m1s chain=input protocol=tcp src-address=172.16.2.0/24
  - /ip firewall nat add action=add-src-to-address-list address-list=NAT-AL chain=srcnat
  - /ip firewall mangle add action=add-dst-to-address-list address-list=DST-AL address-list-timeout=10m chain=prerouting protocol=tcp

**Source NAT**

NAT

* Network Address Translation \(NAT\) allows hosts to use one set of IP addresses on the LAN side and an other set of IP addresses when accessing external networks.
* Source NAT translates private IP addresses \(on the LAN\) to public IP addresses when accessing the Internet. The reverse is done for return traffic. It's sometimes referred to as "hiding" your address space \(your network\) behind the ISP supplied address.

Masquerade and src-nat action

* The first chain for NATing is "srcnat". It's used by traffic leaving the router.
* Much like firewall filters, NAT rules have many properties and actions \(_13 actions!_\).
* The first, and most basic of NAT rules, only uses the "masquerade" action.
* Masquerade replaces the source IP address in packets by one determined by the routing facility.

– Typically, the source IP address of packets going to the Internet will be replaced by the address of the outside \(WAN\) interface. This is required for return traffic to "_find it's way home_".

Masquerade and src-nat action

* The "src-nat" action changes the source IP address and port of packets to those specified by the network administrator

– **Usage example** : Two companies \(Alpha and Beta\) have merged and they both use the same address space \(ex. 172.16.0.0/16\). They will set up a segment using a totally different address space as a buffer and both networks will require src-nat and dst-nat rules.

**Destination NAT**

Dst-nat and redirection action

* "Dst-nat" is an action used with the "dstnat" chain to redirect incoming traffic to a different IP address or port

– **Usage example** : In our previous Alpha and Beta example, we see that dst -nat rules will be required to reconvert the "buffer IP address" to Beta's server's address.

Dst-nat and redirection action

* "Redirect" changes the destination port to the specified "to-ports" port of the router.

– **Usage example** : All http \(TCP, port 80\) traffic is to be sent to the web proxy service at TCP port 8080.

NAT Syntax

* Source NAT \(from /ip firewall nat\)

– Add the masquerade rule

* * * add action=masquerade chain=srcnat

– Change the source IP address

* * * add chain=srcnat src-address=192.168.0.109 action=src-nat to-addresses=10.5.8.200
* Destination NAT

– Redirect all web traffic \(TCP, port 80\) to the router's web proxy on port 8080

* * * add action=redirect chain=dstnat dst-port=80 protocol=tcp to-ports=8080

Time for a practical exercise

**End of module 6**

## Лабораторка

* Goals of the lab
  - Setup basic firewall rules
  - Configure a basic address-list
  - Apply basic source NAT rules and test them out
  - Apply basic destination NAT rules and test them out

Лабораторка: Установка

![](/pics/m6_lab_setup.jpeg)

Лабораторка: шаг 1

* Before going ahead with firewall rules, we'll test a NAT rule : Masquerading
  - Look into your settings to see if you have a "masquerading" NAT rule. Create one if you don't BUT leave it disabled. If you have one make sure that it's disabled
  - Launch Winbox and connect to a neighbour pod.
  - In the IP FIREWALL CONNECTION section, look at active connections. What do you see? Why?
  - Set the configuration option that will let you track connections. Check the results.
  - Enable the masquerade NAT rule and check connection tracking again.

Лабораторка: шаг 2

* Let's make things more interesting by adding filter rules. Apply the following rules to incoming traffic on your WAN interface.
  - Accept icmp echo replies
  - Drop icmp echo requests
  - Accept all "established" and "related" input and forward traffic
  - Drop all "invalid" input and forward traffic
  - Log the rest of input and forward traffic
  - Drop the rest of input and forward traffic
  - Add meaningful comments to all rules.
  - Do the same for the "log" rules' prefixes.

Лабораторка: шаг 3

* Now that you have rules, check your logs. Look at the messages and their format
* Seeing what you see now, do you think troubleshooting connection problems would be easier? Why?

Лабораторка: шаг 4

* Create Address Lists representing all pods
* Use the following format:
  - Name : Pod1
  - Address : &lt;_network/mask_&gt; of the LAN
  - Name : Pod1
  - Address : &lt;_IP_&gt; of the WAN interface

* Do so for all pods, even your own

Лабораторка: шаг 5

* Pods should be matched in pairs for the following tests
* Close your WinBox window and reopen it, connecting to your peer pod. What's happening?
* With one filter rule ONLY, allow all IP addresses from you peer pod to connect to your router with WinBox (TCP, 8291)
  - Make sure that it's in the right spot so that it works
  - And DON'T forget comments!

Лабораторка: шаг 6

* To test port redirection, we'll need to make a small change to the IP SERVICES of your pod.
  - In the IP Services section, change the WinBox port to 8111.

Лабораторка: шаг 7

* Close and reopen the WinBox interface without adding any special parameters. What result do you get?
* Log into the WinBox using port 8111.
* Create a dst-nat rule with a redirect action to port 8111 on all TCP port 8291 traffic.
* Close and reopen WinBox without the port after the IP address. Does it work now?
* Log into you peer pod's router. What's happening?

Лабораторка: шаг 8

* Return the WinBox port to it's normal value of 8291.
* Disable (don't delete) the dstnat rule of "redirect".
* Close WinBox and validate that you can log into your router and your peer's router normally.

Лабораторка: шаг 9

* Create a dst-nat rule with a redirect action to port 8291 on all TCP port 1313 traffic coming into the WAN port.
* Open WinBox and log into your router using port 1313.
* Open WinBox and log into your peer's router using port 1313.
* Explain the different results.

Лабораторка: шаг 10

* Do an export AND a binary backup under the file name module6-pod_x_.
