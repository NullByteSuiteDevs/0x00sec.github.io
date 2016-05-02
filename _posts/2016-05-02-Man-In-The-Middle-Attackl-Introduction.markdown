--- 
layout: post 
title: Man in the middle Attack (Introduction to Hacking)
date: 2016-05-02 09:39
categories: Networking
author: L3akM3-0day
--- 

# Introduction
As a hacker or anything else related to security or hacking, you should know the most famous hacking attack a.k.a *The Man in the middle Attack*. 
This attack is the easiest attack that should be known by all of security engineer and hackers.

I've seen a lot of people (mainly script kiddies) in forum and other internet place asking for:
- **How do I hack my friend ?** 
- **How can I get into a computer ?**

without knowing the basic understanding of computer networking. So today I will teach you some basic networking knowledge.

If you already have an understanding of how arp protocol works you can got to the next section, if not please take your time to read the section **The ARP Protocol**.

## The ARP Protocol

Let's begin ! 

Here a little drawing for you to understand the arp protocol.

We have the Hacker, Alice and bob.

![Man In The Middle](http://i.imgur.com/TpDL57z.png)

In a computer network, all connected devices has a _Mac Address_ also know as _Physical Address_. This Address is use in the arp protocol. 
Here's the story
> The Hacker John want to sniff the conversation of Alice and bob to steal their credential, John will perform a Man In The Middle attack to sniff the converstation between Bob and Alice.


So here is how arp protocol work

- **Alice** want to send a message to **Bob**
- **Alice** will send a arp-request : **arp-request who-is 192.168.1.3**
- **Bob** will then answer with an arp-reply : **arp-reply 192.168.1.3 is-at CC:CC:CC:CC:CC:CC**

It's the same process when Bob when to send a message to Alice.
They is a little problem in the arp protocol, and it's the Gratuious ARP. John, the Hacker will send a fake Gratuious ARP to Bob and Alice.

![Man In The Middle](http://i.imgur.com/r1sBhxY.png)

So here the process of the Attack :

- **John** send arp-reply : **arp-reply 192.168.1.3 is-at BB:BB:BB:BB:BB:BB** to **Alice**
- **John** send arp-reply : **arp-reply 192.168.1.1 is-at BB:BB:BB:BB:BB:BB** to **Bob**
- **Alice** arp cache is poisoned 
- **Bob** arp cache is poisoned 
- **John** can now sniff the converstation 





## The Attack
Now that you read **The ARP Protocol** section we can delve into the attack.

In this tutorial I will use Kali Linux, in order to perform this attack we will use the tool arpspoof for arp-cache poisonning and wireshark to sniff the network traffic.
Here the setup of my Lab 
- A debian Machine as a webserver
- A windows Client
- Kali Linux as the Hacker

Here's how the Page looks without *Man in the middle attack*

![Man In The Middle](http://i.imgur.com/npk2H7x.png)


Turn on your Kali linux Machine and run :

Run `echo 1 > /proc/sys/net/ipv4/ip_forward` to enable forwarding 

> without this command you'll do a denial of service to the windows client. The client won't be able to access to the webserver

and then run 

`arpspoof -t *ip address of the gateway* *ip address of the victim device*`

`arpspoof -t *ip address of the victim device* *ip address of the gateway* `

to perform the attack.

> Here the ip address of the gateway will be the webserver ip, and the ip of the victim ip device will be the windows client

So the attack will look like this in Kali Linux :

![MITM](http://i.imgur.com/itJ30gw.jpg)

Run the last command : 

`iptables -t nat -A PREROUTING -p tcp -m tcp --dport 80 -j DNAT --to-destination *your ip address*:80`

(e.g : iptables -t nat -A PREROUTING -p tcp -m tcp --dport 80 -j DNAT --to-destination 192.168.56.3:80)

> On The kali linux machine I run a fake web server, the victim on the windows 7 client will now see my fake page. If we refresh the page on the windows Client, our page will be shown to the user.

![MITM](http://i.imgur.com/hth12FJ.png)

## How to protect yourself against _Man in the middle Attack_
First of all, don't use free wifi access point to check you bank account. Free Hotspot can be a good spot to hack people and steal credential
- Gmail
- Facebook
- Bank Account
- Force user to download an update

Sometime hackers just use a smartphone to create a fake hotspot ( e.g in a supermarket ) and get some user to connect to their free AP. 
tehy will use their own services ( DNS , DHCP ) to perform malicious attack on you.

If you want to use free wifi, always check the url and look for the *https* and try to avoid log into your bank or email account.

On some networking device *(e.g Cisco switches)*, you can enable Dynamic Arp Inspection to protect your user against this attack.

Hope you understand this tutorial and see you soon ! 

**@L3akM3-0day**

