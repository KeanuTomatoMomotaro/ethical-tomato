# Capturing Remote Network Traffic With WireShark

This will be the first in a series of documents that keeps track of the things that were learned while practicing ethical hacking activities. Mind you, I have little networking knowledge and is more comfortable working with software-engineering tasks compared to infrastructure related tasks. However, I hope to be able to branch-out by writing this series.

The first thing that we'll learn to do is to capture network traffic with WireShark. [WireShark](https://www.wireshark.org/) is a tool that can be used to capture network packets in order to analyse them later. A lot of my colleagues who have dabbled in the infra / networking domain are pretty well acquainted with WireShark, and might also think that I'm a total beginner by writing a "beginner-friendly" tutorial note like this (but I'm willing to put up with their teases in the pursuit of knowledge). Lets dive-in and start swimming with the fishies!

## Environment Topology Setup

In order to keep things simple, I will not be investing in a lot of hardware and rely on a single computer with [virtualbox](https://www.virtualbox.org/) running an Ubuntu and Kali Linux virtual machine. Ubuntu is chosen for its simplicity and popular uses, while Kali Linux will be used as the internet measurement / hacking machine (+1 points for online rep edginess). In addition, WireShark and a variety of other tools come pre-installed with Kali Linux, refer [here](https://www.kali.org/docs/tools/) for a comprehensive list of tools that comes with Kali Linux.

Each virtual machines will need to be able to communicate with each other, as well as access the internet (should the need arise in later tutorials). Hence, they are configured to use two network adapters that consists of a network access translator (NAT) network and virtualbox virtual network. The virtual network configured in virtualbox serves as a simulated network that allows us to capture network traffic from the Ubuntu machine with the Kali Linux box. The NAT network adaptor is used by each machine for external network communications (browsers for emergencies).

I did a quick google search and found some tips [here](https://www.techrepublic.com/article/how-to-create-virtualbox-networks-with-the-host-network-manager/) for the aforementioned network configuration so you don't have to.


## Check for connectivity

Before we can start capturing network traffic, we must insure that each machine is able to observe each other in the virtual network. 

Issue an `ifconfig` command to find the ip address of the Ubuntu machine. In this example, the Ubuntu machine's virtual network ip address is `192.168.56.101`. The `10.0.2.15` ip address is the NAT adaptors ip address, we wont be using this one in this tutorial. (Kindly provide pointers on a more "elegant" network setting for my virtualbox setup if you know any good ones!).
```
knurherb@knurherb-VirtualBox:~$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::836a:77d1:85e7:8bef  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:d7:48:fc  txqueuelen 1000  (Ethernet)
        RX packets 625  bytes 207555 (207.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 664  bytes 77118 (77.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.101  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::2dab:8034:583a:98e3  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:c4:a9:ee  txqueuelen 1000  (Ethernet)
        RX packets 238  bytes 28870 (28.8 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 105  bytes 13183 (13.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 304  bytes 30337 (30.3 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 304  bytes 30337 (30.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
In the Kali Linux machine, you would have to use the  `ip` command according to the [Kali Linux Forums](https://forums.kali.org/showthread.php?36392-bash-ifconfig-command-not-found!!!). The Kali machine's ip address is `192.168.56.102`

```
knurherb@knurherb-kali:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:dc:3d:c5 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:63:ad:4b brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.102/24 brd 192.168.56.255 scope global dynamic noprefixroute eth1
       valid_lft 514sec preferred_lft 514sec
    inet6 fe80::a00:27ff:fe63:ad4b/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

```
## Capture Network Traffic with WireShark
#### Start WireShark capture
Now that we have connectivity, lets start capturing the Ubuntu machine's network traffic with WireShark on the Kali box. Start WireShark on the Kali Linux machine and enter your password if you are prompted. WireShark requires certain permissions to capture traffic with the network interfaces.

Select the network adaptor that uses the the virtual local ip address. If you look at the results of the `ip` command, that would mean `eth1`. In the capture filter settings, specify `hostname` along with the ip address of the Ubuntu machine as follows:
```
hostname 192.168.56.101
```
Finally, click "start" to begin capturing network traffic. You should not be able to see anything on the capture window results, because the Ubuntu machine has not attempt any connections with its virtual local ip address. 

#### Simulate network activity on target machine
Next, once the WireShark capture has been setup, go to the Ubuntu machine and issue ping commands to an external network address, and the virtual local address of the Kali machine.

Execute a ping command to `google.com`. 
```
ping google.com
```
<details>
  <summary>Do you think the network activity will be captured by the Kali Linux machine?</summary>

  If you switch back to your Kali Linux machine, the WireShark capture window would still be empty. The ip address used to access external connections is the NAT ip address, not the local virtual ip address. In the WireShark capture settings, recall that the specified host to capture is the local virtual ip address of the Ubuntu machine.
</details>

Afterwards, execute a ping command to `192.168.56.102`;
```
ping 192.168.56.102
```
<details>
  <summary>Do you think the network activity will be captured by the Kali Linux machine this time?</summary>

  If you switch back to your Kali Linux machine, the WireShark capture would start showing network packets relevant to the ping command executed.
```
No.	Time	Source	Destination	Protocol	Length	Info
1	0.000000000	192.168.56.101	192.168.56.102	ICMP	98	Echo (ping) request  id=0x0008, seq=1/256, ttl=64 (reply in 2)
2	0.000020379	192.168.56.102	192.168.56.101	ICMP	98	Echo (ping) reply    id=0x0008, seq=1/256, ttl=64 (request in 1)
3	1.026422645	192.168.56.101	192.168.56.102	ICMP	98	Echo (ping) request  id=0x0008, seq=2/512, ttl=64 (reply in 4)
4	1.026458106	192.168.56.102	192.168.56.101	ICMP	98	Echo (ping) reply    id=0x0008, seq=2/512, ttl=64 (request in 3)
5	2.050812453	192.168.56.101	192.168.56.102	ICMP	98	Echo (ping) request  id=0x0008, seq=3/768, ttl=64 (reply in 6)
6	2.050849063	192.168.56.102	192.168.56.101	ICMP	98	Echo (ping) reply    id=0x0008, seq=3/768, ttl=64 (request in 5)
7	5.027229926	PcsCompu_c4:a9:ee	PcsCompu_63:ad:4b	ARP	60	Who has 192.168.56.102? Tell 192.168.56.101
8	5.027243664	PcsCompu_63:ad:4b	PcsCompu_c4:a9:ee	ARP	42	192.168.56.102 is at 08:00:27:63:ad:4b
9	5.231419574	PcsCompu_63:ad:4b	PcsCompu_c4:a9:ee	ARP	42	Who has 192.168.56.101? Tell 192.168.56.102
10	5.231944386	PcsCompu_c4:a9:ee	PcsCompu_63:ad:4b	ARP	60	192.168.56.101 is at 08:00:27:c4:a9:ee
```
Here, we can observe the source and destination ip address, TCP protocol used, and other information of the packets.
</details>

## Wrap Up

We've seen the steps that I used to practice network packet capture with WireShark and virtualbox. It may seem simplistic, but its a good starting-step for later complex network analysis. For example, we can specify which tcp protocol to capture, or we could even capture the traffic via remote machine. The WireShark [capture documentation](https://wiki.wireshark.org/CaptureSetup) mentions a variety of options that we can choose to practice more stuff.

In addition, the steps I wrote here is my attempt at following along [Youtube tutorial by Hackersploit](https://www.youtube.com/watch?v=XrngOSIsxko&t=479s). It is a pretty interesting and helpful channel (and apparently quite famous for ethical hacking tutorials).

If you kept reading until the end, let me know if I can write better documentations. Leave a comment or star on this readme / repository so I know how many people have come across my notes, and would like to tag along with me on my journey to become a better "ethical hacker"
