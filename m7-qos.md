# M7 QoS

## **Simple queue**

Introduction

* QoS \(quality of service\) is the art of managing bandwidth resources rather just "blindly" limiting bandwidth to certain nodes
* QoS can prioritize traffic based on metrics. Useful for

– Critical applications

– Sensitive traffic such as voice and video streams

Introduction

* Simple queues are a… simple… way to limit bandwidth to

– Client upload

– Client download

– Client aggregate \(download and upload\)

Target

* Target to which the simple queue is applied
* A target **MUST** be specified. It can be

– An IP address

– A subnet

– An interface

* Queue order IS important. Each packet must go through every simple queue until a match occurs

Destinations

* IP address where the target's traffic is aimed, or
* Interface through which target's traffic will flow through
* Not compulsory like the "target" field
* Can be used to limit the queue's restriction

Max-limit and limit-at

* The "max-limit" parameter is the maximum data rate that a target can reach

– Viewed as MIR \(_maximum information rate_\)

– Best case scenario

* The "limit-at" parameter is a guaranteed minimum data rate for the target

– Viewed as CIR \(_committed information rate_\)

– Worst case scenario

Bursting

* Bursting permits users to get, for a short time, more bandwidth than allowed by "max-limit" parameter.
* Useful to boost traffic that doesn't use bandwidth too often. For example, HTTP. Get a quick page download, than read it for a few seconds.

Bursting

* Definitions.

– **Burst-limit** : Maximum data rate while burst is allowed.

– **Burst-time** : Time, in seconds, over which the sampling is made. It is NOT the period during which traffic will burst.

– **Burst-threshold** : The value that will determine if a user will be permitted to burst

– **Average-rate** : An average of data transmission calculated in 1/16th parts of "burst-time".

– **Actual-rate** : Current \(real\) rate of data transfer.

Bursting

* How it works.

– Bursting is allowed while **average-rate** stays below **burst-threshold**.

– Bursting will be limited at the rate specified by **burst-limit**.

– **Average-rate** is calculated by averaging 16 samples \(**actual-rate**\) over **burst-time** seconds.

* * * If **burst-time** is 16 seconds, then a sample is taken every second.
    * If **burst-time** is 8 seconds, then a sample is taken every ½ second. And so on…

– When bursting starts, it will be allowed for **longest-burst-time** seconds, which is

* * * \(burst-threshold x burst-time\) / burst-limit.

Bursting

![](.gitbook/assets/0.png)

2013-01-01

With a burst-time of 16 seconds

Bursting

![](.gitbook/assets/1%20%281%29.jpeg)

With a burst-time of 8 seconds

Syntax

• A simple queue

– add max-limit=2M/2M name=queue1 target=192.168.3.0/24

• The same queue with bursting

– add burst-limit=4M/4M burst-threshold=1500k/1500k burst-time=8s/8s limit-at=\

1M/1M max-limit=2M/2M name=queue1 target=192.168.3.0/24

Tip

* You may have noticed that queue icons change color according to usage. Color has a meaning.

– Green : 0 – 50% of available bandwidth used

– Yellow : 51 – 75% of available bandwidth used

– Red : 76 – 100% of available bandwidth used

**One Simple queue for the whole Network \(PCQ\)**

Why have a queue for all?

* Per Connection Queue \(PCQ\) is a dynamic way of shaping traffic for multiple users using a simpler configuration.
* Define parameters, then each sub-stream \(specific IP addresses, for example\) will have the same limitations.

Pcq-rate configuration

* The parameter **pcq-rate** limits the queue type's allowed data rate.
* Classifier is what the router checks to see how it will apply this limitation. It can be on source or destination address, or source or destination port. You could thus limit user traffic or application traffic \(HTTP for example\).

Pcq-limit configuration

* This parameter is measured in packets.
* A large pcq-limit value

– Will create a larger buffer, thus reducing dropped packets

– Will increase latency

* A smaller pcq-limit value

– Will increase packets drops \(since buffer is smaller\) and will force the source to resend the packet, thus reducing latency

– Will bring about a TCP window size adjustment, telling the source to reduce the transmission rate

Pcq-limit configuration

* What value should I use? There's no easy answer.

– If often starts on a "Trial & Error" basis per application

– If users complain about latency, reduce the pcq-limit \(queue length\)value

– If packets have to go through a complex firewall, then you may have to increase the queue length as it may introduce delays

– Fast interfaces \(like Gig\) require smaller queues as they reduce delays

PCQ, an example

* Lets suppose that we have users sharing a limited WAN link. We'll give them the following data rates:

– Download : 2Mbps

– Upload : 1Mbps

* WAN is on ether1
* LAN subnet is 192.168.3.0/24

PCQ, an example

**/ip firewall mangle**

add action=mark-packet chain=forward new-packet-mark=client\_upload \ out-interface=ether1 src-address=192.168.3.0/24

add action=mark-packet chain=forward dst-address=192.168.3.0/24 \ in-interface=ether1 new-packet-mark=client\_download

**/queue type**

add kind=pcq name=PCQ\_download pcq-classifier=dst-address pcq-rate=2M add kind=pcq name=PCQ\_upload pcq-classifier=src-address pcq-rate=1M

/**queue tree**

add name=queue\_upload packet-mark=client\_upload parent=global queue=\ PCQ\_upload

add name=queue\_download packet-mark=client\_download parent=global queue=\ PCQ\_download

Our example explained

* **Mangle** : We are telling the router to mark packets with the

"_**client\_upload**_" or "_**client\_download**_" mark, depending on if

– Packets are coming from the LAN and are leaving from ether1 \(upload\) or,

– Packets are entering from ether1 and going to the LAN \(download\).

* **Queue types** : We're defining the data rates and classifiers to use to differentiate sub-streams \(source or destination\)
* **Queue tree** : The combinations that are checked to see if packets qualify for traffic shaping and what to apply.

– For example, in the case of uploaded traffic, we check input and output interfaces \(**global**\) for packets with the "**client\_upload**" mark and apply the "**PCQ\_upload**" queue type.

**Monitoring**

Interface traffic monitor

![](.gitbook/assets/2%20%283%29.jpeg)

The traffic monitor tool is used to run scripts when an interface traffic reaches a certain threshold.

**Example**

**/tool traffic-monitor**

add interface=ether1 name=TrafficMon1 on-event=script1 threshold=1500000 \ traffic=received

**/system script**

add name=script1 policy=ftp,read,test,winbox,api source="/tool e-mail send to=\"\ YOU@DOMAIN.CA\" subject=\(\[/system identity get name\] . \" Log \

\" . \[/system clock get date\]\) body=\"Hello World. You're going too fast!\""

Torch

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

