---
description: М2 Маршрутизация
---

# M2 Routing

## **Обзор маршрутизации**

Концепции маршрутизации

* Маршрутизация-это процесс уровня 3 в модели OSI ISO.
* Маршрутизация определяет, куда перенаправляется \(отправляется\) трафик.
* Она необходима чтобы разрешить различные подсети для обмена данными.
  * Даже если они должны быть на одном “проводе”

### Концепции маршрутизации, пример 1

* Компьютеры не будут взаимодействовать

![](.gitbook/assets/0%20%281%29.jpeg)

### Концепции маршршутизации, пример 2

* Компьютеры теперь могут взаимодействовать

![](.gitbook/assets/1.jpeg)

Флаги маршрута

* Маршруты имеют статусы. В этом курсе мы ознакомимся со следующим:
  * X: Disabled \(отключен\)
  * A : Active \(активен\)
  * D : Dynamic \(динамический\)
  * C : Connected \(подключен\)
  * S : Static \(статичен\)

![](.gitbook/assets/2%20%282%29.jpeg)

Флаги маршрута

* **Disabled** : маршрут отключен. Не имеет никакого влияния на процесс маршрутизации.
* **Active** : маршрут активен и используется в процессе маршрутизации.
* **Dynamic** : маршрут был создан процессом маршрутизации, а не через интерфейс управления.
* **Connected** : маршруты создаются для каждой IP-подсети, которая имеет активный интерфейс на маршрутизаторе.
* **Static** : маршрут, созданный для принудительной пересылки пакетов через определенный .

## **Static Routing**

Static routes

* Routes to subnets that exist on a router are automatically created and known by that router. But what happens if you need to reach a subnet that exists on another router? You create a static route!
* A static route is a manual way of forwarding traffic to unknown subnets.

Static routes

![](.gitbook/assets/3.jpeg)

Static routes

* Understanding the fields

– **Flags** : The state of each route, as explained in previous slides

– **Dst. Address** : The destination addresses this route is used for.

– **Gateway** : Typically, the IP address of the next hop that will receive the packets destined for “_**Dst. Address**_”.

**Distance** : Value used for route selection.

– In configurations where various distances are posible, the route with the smallest

Why use static routing

* Makes configuration simpler on very small network which will most likely not grow.
* Limits the use of router resources \(memory, CPU\)

![](.gitbook/assets/4.jpeg)

Limits of static routing

* Doesn’t scale well.
* Manual configuration is required every time a new subnet needs to be reached.

| Limits of static routing, example |
| :--- |
|  |
| Your network grows and you need to add links to remote routers \(and subnets\). |
| • Assume that all routers have 2 LAN subnets and 1 or more WAN ![](.gitbook/assets/5.jpeg)subnets. |
|  |

Limits of static routing, example

How many static routes to add on router-1?

![](.gitbook/assets/6%20%281%29.jpeg)

* Routers 3 to 5 : 9
* Router 2 : 2
* Router 6 and 7 : 4

| **Total of 15 static** |  |  |
| :--- | :--- | :--- |
| **routes to add** |  |  |
| **manually!!** |  |  |
|  |  |  |

Creating routes

| • To add a static |  |  |
| :--- | :--- | :--- |
| route : |  |  |
| – | IP -&gt; Routes |  |
| – | + \(Add\) |  |
| – | Specify |  |
|  | destination |  |
|  | subnet and mask |  |
| – | Specify |  |
|  | “Gateway” \(next |  |
|  | hop\) |  |
|  |  |  |

Setting the default route

* The route 0.0.0.0/0

– Known as the **Default route**.

– It is the destination where all traffic to unknown subnets will be forwarded.

– It is also a static route.

Managing dynamic routes

* As mentioned before, dynamic routes are added by the routing process, not by the administrator.
* This is done automatically.
* You can’t manage dynamic routes. If the interface to which the dynamic route is linked goes down, so does the route!

Managing dynamic routes, example

![](.gitbook/assets/7%20%281%29.jpeg)

Implementing static routing on simple networks

Consider the following example.

![](.gitbook/assets/8%20%282%29.jpeg)

Implementing static routing on simple networks

* Exercise:

Assuming ip addresses have been properly entered, what commands would you use to enable complete communications for both subnets \(LAN1 and LAN2\)?

\(Answer on next slide. Don’t peak \)

Implementing static routing on simple networks

* router-1

/ip route

add gateway=172.22.0.18

add dst-address=10.1.2.0/24 gateway=10.0.0.2

* router-2

/ip route

add gateway=10.0.0.1

Time for a practical exercise

## Laboratory

* Goals of the lab

– Gain connectivity to other POD LANs

– Validate use of default route

– View and explain route flags

Laboratory : Setup

![](.gitbook/assets/9%20%281%29.jpeg)

Laboratory : step 1

* Delete the default route that was created in module 1
* Ping other PODs’ computers. Note results
* Create static routes to other PODs’ LAN subnets
* Ping other PODs’ computers. Note results

Laboratory : step 2

* Open a Web browser and try accessing Mikrotik’s Web page. Note results
* Create the default route using the trainer’s router as the gateway
* Open a Web browser and try accessing Mikrotik’s Web page. Note results

**End of Laboratory 2**

