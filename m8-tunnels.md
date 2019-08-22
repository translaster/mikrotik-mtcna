# M8 Tunnels

## Tunnels

* Tunnels are a way of expanding your private network across a public network, such as the Internet.
* They are also referred to as VPNs \(virtual private networks\).
* The concept of security is associated with VPNs. They're used since it's not desirable to allow the users' traffic to go through unsecured and not privately owned \(by the client\) networks.

**PPP settings**

PPP profile

* PPP profiles represent configuration parameters to be used by PPP clients such as, but not limited to :

–

–

–

Local and remote IP addresses or pools

Compression

Encryption

**/ppp profile** \(example from a client\)

add change-tcp-mss=yes name=Profile-external use-compression=\

yes use-encryption=yes use-vj-compression=no

**/ppp profile** \(example from a server\)

add change-tcp-mss=yes local-address=192.168.222.1 name=Profile-external \

remote-address=192.168.222.2 use-compression=yes use-encryption=yes \

use-vj-compression=no

add change-tcp-mss=no dns-server=192.168.5.1 local-address=192.168.5.1 name=\

| Profile-internal remote-address=Pool-VPN use-compression=yes \ |  |  |
| :--- | :--- | :--- |
| use-encryption=yes use-vj-compression=no |  |  |
|  |  |  |

PPP secret

* PPP secrets are found on PPP servers and they specify the basic parameters required to authenticate a client, such as:

–

–

Name : The user's identification

Password : The user’s password

– Service : The protocol being serviced \(If left to "any", the PPP secret will authenticate the user through any service \(PPPoE, L2TP, PPTP, etc.\)\)

– Profile : The configuration subset to be used by this user. Profiles allow parameters to be used by many users without having to retype everything every time.

* Clients do not use PPP secrets as their authentication credentials. They are specified in the PPP client's interface under the "user" and "password" parameters.

**/ppp secret**

add name=Pod4-external password=pod4-123 profile=Profile-external routes=\

192.168.4.0/24

add name=alain password=alain!! profile=Profile-internal

PPP status

* It represents the connections' current status. Useful to debug and verify proper operations of your tunnels.

\[admin@Pod5\] &gt; **/ppp active print detail**

Flags: R - radius

* name="alain" service=pppoe caller-id="28:D2:44:2C:06:EE" address=192.168.5.100 uptime=3m56s encoding="MPPE128 statefull" session-id=0x81B00044 limit-bytes-in=0 limit-bytes-out=0
* name="Pod4-external" service=pppoe caller-id="D4:CA:6D:8E:1A:97" address=192.168.222.2 uptime=37s encoding="MPPE128 stateless" session-id=0x81B00045 limit-bytes-in=0 limit-bytes-out=0

| \[admin@Pod5\] &gt; **/ppp active print** |  |  |  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Flags: R - radius |  |  |  |  |  |  |  |
| \# | NAME | SERVICE CALLER-ID | ADDRESS | UPTIME | ENCODING |  |  |
| 0 | alain | pppoe | 28:D2:44:2C:06:EE | 192.168.5.100 | 4m12s | MPPE128 | statefull |
| 1 | Pod4-exte... | pppoe | D4:CA:6D:8E:1A:97 | 192.168.222.2 | 53s | MPPE128 | stateless |

**IP pool**

Creating a pool

* IP pools define a range of IP addresses for clients.
* Not only is it used for DHCP, as we saw earlier in this course, but it can be used for PPP and Hotspot clients.
* Useful when an interface can service many clients. Addresses are assigned from the pool automatically.

Managing ranges

* IP pool ranges are lists of non-overlapping IP addresses that can be assigned to clients through services \(DHCP, PPP, hotspots\).
* Let's demonstrate with an example. You have 50 computers on the corporate LAN and 50 coming in from you VPN.

**/ip pool**

add name=Pool-PC ranges=192.168.5.50-192.168.5.99

add name=Pool-VPN ranges=192.168.5.100-192.168.5.149

Managing ranges

* You need to add 50 more computers in the LAN's pool.

| **/ip pool print** |  |  |
| :--- | :--- | :--- |
| \# NAME | RANGES |  |
| 0 | Pool-PC | 192.168.5.50-192.168.5.99 |
| 1 | Pool-VPN | 192.168.5.100-192.168.5.149 |

**/ip pool**

set 0 ranges=192.168.5.50-192.168.5.99,192.168.5.150-192.168.5.199

| **/ip pool&gt; print** |  |  |
| :--- | :--- | :--- |
| \# NAME | RANGES |  |
| 0 | Pool-PC | 192.168.5.50-192.168.5.99 |
|  |  | 192.168.5.150-192.168.5.199 |
| 1 | Pool-VPN | 192.168.5.100-192.168.5.149 |

Assigning to a service

•

•

Pools can be assigned to services such as DHCP, PPP and hotspot.

We'll see the syntax in the slides to come.

**Secure local networks**

PPPoE

* Point-to-point over Ethernet is a layer 2 protocol.
* It is often used by ISP’s to control access to their networks.
* It can be used as a method of access on any layer 2 technology, such as 802.11 or Ethernet.

PPPoE service-name

* The service-name can be seen as the SSID of 802.11, meaning that it’s the network name that the client is looking for.
* Unlike the SSID, if the client doesn’t specify one, the access concentrator \(PPPoE server\) will send all service-names that it services. The client will respond to the first one it gets.

Creating a PPPoE server

* A PPPoE server is the device that is offering the tunneling service.
* It allows clients to get a secured layer 3 VPN service over a layer 2 infrastructure.
* You CANNOT reach a PPPoE server through routers. Since it's a layer 2 protocol, the server can only be reached through the same Ethernet broadcast domain on which the clients are.

Creating a PPPoE server

* Before creating the server itself, create the configuration parameters that you require \(for values other than default\), such as :

–

–

–

IP pools

1. profiles PPP secrets

* Create the server interface on the physical interface facing the clients.

Creating a PPPoE server, example

**/ip pool**

add name=Pool-PC ranges=192.168.5.50-192.168.5.99,192.168.5.150-192.168.5.199 add name=Pool-VPN ranges=192.168.5.100-192.168.5.149

**/ppp profile**

add change-tcp-mss=yes local-address=192.168.222.1 name=Profile-external \ remote-address=192.168.222.2 use-compression=yes use-encryption=yes \ use-vj-compression=no

add change-tcp-mss=no dns-server=192.168.5.1 local-address=192.168.5.1 name=\ Profile-internal remote-address=Pool-VPN use-compression=yes use-encryption=\ yes use-vj-compression=no

Creating a PPPoE server, example

**/ppp secret**

add name=Pod4-external password=pod4-123 profile=Profile-external routes=\ 192.168.4.0/24

add name=alain password=alain!! profile=Profile-internal

**/interface pppoe-server server**

add authentication=mschap2 default-profile=Profile-external disabled=no \ interface=ether1 mrru=1600 service-name=PPPoE-external

add authentication=mschap2 default-profile=Profile-internal disabled=no \ interface=ether5 mrru=1600 service-name=PPPoE-internal

Creating a PPPoE server

Tip :

You can leave an Ethernet port without a master port, a bridge or an IP address and the client that is connected to this port can still get Internet access if your PPPoE server \(and the PPPoE client\) is properly configured.

Point-to-point addresses

* The easiest way of setting up addresses is hardcoding them in the configuration.
* Address from /ppp secret has precedence over /ppp profile, and they take precedence over /ip pool.
* Both local and remote addresses can be unique or from a pool.
* Static IP addresses or DHCP should not be used on PPPoE client interfaces. Let the infrastructure control what is given out!

Creating PPPoE clients on RouterOS

* If you wish to use a different profile than the default ones, create it first. You won't have to come back to it later.

•

•

Create the client interface on the interface facing the ISP.

You're done!

Tip :

Your router would not have to be configured with a DHCP client on the WAN interface and it would still work if the PPPoE server is on the same layer 2 infrastructure as the WAN port.

PPPoE client on RouterOS, example

**/ppp profile**

add change-tcp-mss=yes name=Profile-external use-compression=yes \ use-encryption=yes use-vj-compression=no

**/interface pppoe-client**

add ac-name="" add-default-route=yes allow=mschap2 \ default-route-distance=1 dial-on-demand=no disabled=no \ interface=ether1 keepalive-timeout=60 max-mru=1480 max-mtu=1480 \ mrru=disabled name=Client-PPPoE password=pod4-123 profile=\ Profile-external service-name="" use-peer-dns=no user=\ Pod4-external

* Enable the client interface.

**Secure remote networks communication**

PPTP clients and servers

* PPTP is a layer 3 tunneling protocol and uses IP routing information and addresses to bind clients to servers.
* Defining the PPTP server is almost the same thing as for PPPoE, except that no interface has to be specified.
* The client is defined almost the same way as a PPPoE client, except that an IP address has to be specified for the server.
* Tip : You must permit TCP, port 1723 in the router's firewall \(the PPTP server\) for your tunnel to come up.

**/interface pptp-server server**

set authentication=mschap2 default-profile=Profile-external enabled=yes

**/interface pptp-client**

add add-default-route=yes allow=mschap2 connect-to=192.168.0.5 \

default-route-distance=1 dial-on-demand=no disabled=no keepalive-timeout=60 \

max-mru=1450 max-mtu=1450 mrru=1600 name=Client-PPTP password=pod4-123 profile=\

Profile-external user=Pod4-external

SSTP clients and servers without certificates

•

•

•

Defining the SSTP server is almost the same thing as for PPTP, except that you specify a TCP port to connect to \(443 by default\).

The client is defined almost the same way as a PPTP client, except that you specify a TCP port to use to establish a connection \(443 by default\).

Tip : You must permit TCP, port 443 for your tunnel to come up. Also, leave the port at 443 to ensure SSL is used for your communications.

**/interface sstp-server server**

set authentication=mschap2 enabled=yes

**/interface sstp-client**

add add-default-route=no authentication=mschap2 certificate=none connect-to=\

192.168.0.5:443 dial-on-demand=no disabled=no http-proxy=0.0.0.0:443 \

keepalive-timeout=60 max-mru=1500 max-mtu=1500 mrru=1600 name=Client-SSTP \

password=pod4-123 profile=Profile-external user=Pod4-external \

verify-server-address-from-certificate=no verify-server-certificate=n

Setup routes between networks

* Once your tunnel is up, you need routes to move packets back and forth.
* The first way, for a single client tunnel, is the route that is automatically

created for that tunnel.

**/ip route print**

| Flags: X - disabled, A - active, D - dynamic, |  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- | :--- |
| C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme, |  |  |  |  |  |
| B - blackhole, U - unreachable, P - prohibit |  |  |  |  |  |
| \# |  | DST-ADDRESS | PREF-SRC | GATEWAY | DISTANCE |
| 0 | ADS | 0.0.0.0/0 |  | 192.168.0.254 | 0 |
| 1 | ADC | 192.168.0.0/24 | 192.168.0.5 | ether1 | 0 |
| 2 | ADC | 192.168.5.0/24 | 192.168.5.1 | Bridge-PC | 0 |
| **3** | **ADC** | **192.168.5.101/32** | **192.168.5.1** | **&lt;pptp-alain&gt;** | **0** |

Setup routes between networks

* The second way is to specify one or multiple routes within the PPP

secret for a client.

**/ppp secret export**

add name=Pod4-external password=pod4-123 profile=Profile-external routes=192.168.4.0/24 add name=alain password=alain!! profile=Profile-internal

**/ppp secret print**

Flags: X - disabled

| \# | NAME | SERVICE CALLER-ID | PASSWORD | PROFILE | REMOTE-ADDRESS |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 0 | Pod4-external | any | pod4-123 | Profile-external |  |
| 1 | alain | any | alain!! | Profile-internal |  |

**/ppp secret**

set 0 routes=192.168.4.0/24,10.10.2.0/24

**/ppp secret export**

add name=Pod4-external password=pod4-123 profile=Profile-external routes=192.168.4.0/24,10.10.2.0/24 add name=alain password=alain!! profile=Profile-internal

Setup routes between networks

* The third way is to add static routes to one or multiple networks across a tunnel.
* This method is useful if both routers must have their own default route, but it implies more maintenance and parameters.

**/ip route**

add comment="TO OFFICE LOOPBACKS" distance=1 dst-address=10.10.2.0/24 gateway=192.168.254.10

add comment="TO OFFICE NETWORKS" distance=1 dst-address=172.16.8.0/21 gateway=192.168.254.10

Closing note

![](.gitbook/assets/0%20%282%29.png)

| **VPN Protocol** | **Encryption** | **Ports** | **Compatible with** | **Notes** |
| :--- | :--- | :--- | :--- | :--- |
|  |  |  |  |  |
| PPTP | MPPE with RC4 | 1723 TCP | Windows XP, Vista, 7 | PPTP is the most widely used VPN protocol today. |
|  | 128 bit key |  | Mac OS X | It is easy to setup and can be used to bypass all Internet restrictions. |
|  |  |  | iPhone OS | PPTP is considered less secure. |
|  |  |  | Android |  |
|  |  |  |  |  |
| SSTP |  |  | Windows 7 |  |
|  | SSL with AES |  |  | SSTP uses a generic port that is never blocked by firewalls. |
|  | 2048 bit key certificate | 443 TCP |  | You can use SSTP to bypass corporate or school firewalls. |
|  | 256 bit key for encryption |  |  | SSTP is considered a very secure protocol. |
|  |  |  |  |  |

Want to learn more?

[http://wiki.mikrotik.com/wiki/Manual:Interface/PPTP](http://wiki.mikrotik.com/wiki/Manual:Interface/PPTP) [http://wiki.mikrotik.com/wiki/Manual:Interface/SSTP](http://wiki.mikrotik.com/wiki/Manual:Interface/SSTP) [http://www.highspeedvpn.net/PPTP-L2TP-SSTP-OpenVPN.aspx](http://www.highspeedvpn.net/PPTP-L2TP-SSTP-OpenVPN.aspx) [http://www.squidoo.com/advantages-and-disadvantages-of-vpn-protocols](http://www.squidoo.com/advantages-and-disadvantages-of-vpn-protocols) [http://www.vpnonline.pl/en/protokoly-vpn-porownanie](http://www.vpnonline.pl/en/protokoly-vpn-porownanie) \(good table here!\)

Time for a practical exercise

**End of module 8**

Laboratory

* Goals of the lab

– Create PPP profiles and secrets

– Create and assign IP pools to services

– Create a PPPoE VPN between a computer and a router

– Create PPTP and SSTP tunnels between pods

– Insure proper routing

Laboratory : Setup

![](.gitbook/assets/1%20%282%29.png)

Laboratory : step 1

Students will pair up again for this laboratory.

* Students will create three PPP profiles

– Two to use with the neighbor pod.

•

•

One for the server service.

One for the client service.

– One to use for locally connected clients.

* Students will create two PPP secrets

– One to allow the neighbor pod to connect to the local pod.

– One to connect the locally connected clients.

* Paired students will agree on syntax and content for the parameters. For length's sake, please keep it simple!

Laboratory : step 2

* Create an IP pool to be used by clients wanting to connect by VPN.

– Your pool will be on a different network than your existing LAN.

– Assign the pool to the profile to be used by your future "corporate" VPN.

Laboratory : step 3

* Select a free port on your router and remove it from any bridge group or master port that it may be assigned to. It must not have an IP address or any DHCP configured on it.
* Configure a PPPoE server on your router to use that port. You should use the profile that you created for your VPN clients. Enable only MSChap2 for authentication. Look at the course material for compression and encryption settings.

Laboratory : step 4

•

•

Configure your computer to connect to your router with a PPPoE client connection.

Connect and browse away!

**Warnings!**

– Check the interface on which you configure your server \(and on which you plug your computer\).

– Check the profile setting in your PPPoE server and PPP secret.

Laboratory : step 5

* Connect your computer back on a normal Ethernet interface.
* The even numbered pods will create a PPTP server and a SSTP client.
* The odd numbered pods will create a PPTP client and a SSTP server.
* Use the profiles and secrets previously created.
* SSTP must not use certificates!
* Bring the VPN tunnels up and look at what's happening.

Laboratory : step 6

* Nothing? What did we forget?

– Hint : A new firewall filter maybe?

* Once the tunnels are up, look at the active connections' statuses.

Laboratory : step 7

* Remove static routes from your routing table. You should only have one to your peer pod.
* Ping your peer pod's LAN IP address. Does it work? But the tunnel is still up? How can that be? \(Leave the ping running\)
* Can you ping the remote address of your tunnel? All is not lost then.

Laboratory : step 8

* Open the PPP secret from your router and, in the "Routes" field, add the other pod's network and mask.
* Once this is done on both pods, restart your client tunnels.
* Notice the effect it has in your routing table. Your peer's subnet has appeared once the peer pod logged in. Once both tunnels are up, both will be able to ping.
* Notice also the addresses in IP address list.

Laboratory : step 9

* As usual, save the current configuration in binary and text format using the same name format that has been used in previous labs.

**Best of luck with the certification exam!!**

