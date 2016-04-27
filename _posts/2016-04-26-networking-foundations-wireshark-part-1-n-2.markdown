---
layout: post
title: "Networking Playground: Exploring UDP via Wireshark"
date: 2016-04-26 18:30:00 -0700
catagories: Networking
tags:
  - UDP
  - networking
author: pysec
---
Hello ladies and gentlemen, I'm back with another informative(hopefully) article for you all.In this article I'm going to walk you through one of the main networking protocols when it comes to communication across the Internet between programs, aka UDP(User Datagram Protocol).
<!-- more -->

There are a lot of transport protocols. The transport layer includes all kinds of protocols, such as TCP, UDP, ICMP, ESP that is used for VPN connections, OSPF and EIGRP which are routing protocols and many other more. But when we are talking about programs talking across the network, they primarily use:

**UDP** - "I hope it gets there" - Unreliable Version

**TCP** - "I know it got there" - Reliable Version

Let's get into UDP, shall we? So the question is, why would you want to send a "I hope it gets there" request? The first thing to understand is that there is a cost to reliability. In order to say "I know it got there", there is a lot of setup that takes place. The first thing that happens is something known as a 3-way handshake.

<img src="http://img.wonderhowto.com/img/44/78/63592270802918/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Shortly, the 2 devices that are talking together have to establish a session between each other and make sure that they agree to talk which requires some time. Then, every single packet or stream of communication that gets sent between those 2 devices have to get an acknowledgement back saying "I got it!". Again, more delay, more overhead while some things may just not need that sort of process.

One example that doesn't need that sort of session establishment is VoIP, aka Voice over IP.

<img src="http://img.wonderhowto.com/img/65/99/63592271438746/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

In that case, we have some IP phones and there is a stream of data going between those and if something is dropped, it's gone, there is no use in retransmitting it at a later time because it's real-time traffic. But there is also some other data applications out there that use UDP as well. I will tell you one that you use every-single-day and it's named DNS(Donaim Name Service). DNS translates names to IP addresses, because remember in the OSI model at the network layer we can't squeeze in "google.com" because it deals with IP. So we have to have some kind of system that takes these names such as "youtube.com" or "google.com" and translate it to what IP address is really there. DNS uses UDP. You don't believe me? Let me prove it to you.

I'm going to fire up Wireshark. Wireshark is a phenomenal networking tool, aka the Network/Traffic Analyzer, it's free and one of my all time favorites. If you are on Windows, the installation process is pretty basic so if you Google your way around I'm 101% sure you will install in no time. This demo will be made on my Windows machine.

<img src="http://img.wonderhowto.com/img/34/74/63592272564261/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

My version of wireshark may differ from yours, thus the GUI look of it, if you don't have the latest update but the navigation process shouldn't be that hard to follow along.

Let me give you some basics of this program which will help you get started with it. Wireshark is a pretty terrifying looking tool once you open it for the first time. Above is how mine looks and I would advice you to update your version of Wireshark if you haven't already so you can follow along better. The key icon you want to go to is the "Capture Options".

<img src="http://img.wonderhowto.com/img/43/83/63592275300261/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

This is an extremely important utility. Once I click on this;

<img src="http://img.wonderhowto.com/img/29/67/63592275526857/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

These are all my network interfaces and if you look closely, the "Local Area Connection" seems to be the one with the network traffic, so I'm going to click on it and press Start.

<img src="http://img.wonderhowto.com/img/77/93/63592276082005/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

What you are seeing is a small part of the communication that I screenshoted, that is going accross the network. While the capture is on, try typing on your browser google.com or any site you want and check wireshark again. What you will notice is an increase of the packets being transmitted. Cool, right? The list of the packets is huge so if you would like to stop capturing the traffic after a while you can click the "Stop capturing packets" icon.

<img src="http://img.wonderhowto.com/img/30/10/63592276310058/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

In your case, the icon is going to look red because I have already stopped the traffic capture. As you can see from the "Protocol" tab, there are protocols running in the background such as UDP, TCP, TLS. In your traffic the protocols may differ, you may see plenty of TCP Protocols and not as many UDP as I have but that doesn't matter.

Let's go to what matter to us, which is DNS. As I said, DNS resolves names to IP addresses and I'm going to show you that it's using UDP as its protocol to do it.

You may be thinking "Look at all this traffic mess, how will I find DNS, I need some filtering". Well, PySec has the answer for you.

<img src="http://img.wonderhowto.com/img/49/09/63592277350020/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Bam! When you click on it you will see a list of filters which help you adjust the network traffic the way you want. Isn't that awesome? You can even make your own filter! How? I'm about to show you soon.

First, let's open our terminal and type `ipconfig /all`.

<img src="http://img.wonderhowto.com/img/12/75/63592278005161/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

At the bottom of the command's results I have my DNS servers and it shows that the primary DNS server my computer is using is **2a02:2148:82:8053::53**, which is an IPv6 address by the way. The last one is my router. Yes, routers can run as DNS or even as DHCP servers. How did those DNS IP addresses get there? Through DHCP. If you don't know what DHCP is, make sure to hop on my previous article for more detailed information on that concept.

I'm going to type `ip.addr==192.168.1.1`,which is the IP of my router, which is my alternative DNS server and press enter in the filter search bar on Wireshark and try to see the traffic.

<img src="http://img.wonderhowto.com/img/88/01/63592278992629/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

I'm not expecting to see anything since I've stopped the capture, so let's turn it back on. In order to do that, I will click on the icon right on the left side of the "Stop capturing packets" icon.

<img src="http://img.wonderhowto.com/img/58/44/63592278787270/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Once I click it, it's going to ask me if I want to delete the old capture. You can either save it and keep it for later use or click "Continue without saving". I'll click "Continue without saving" and I'm ready to check my traffic.

<img src="http://img.wonderhowto.com/img/46/03/63592279202645/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Hmm, nothing is happening and yes I've clicked the capture button and the proof is the red "stop capturing packets" option which is always red while capturing traffic. What's going on? Well, as I showed you before my primary DNS server wasn't my router. Let me show you a cool command you may never knew before. I'll type in the command prompt `nslookup`.

<img src="http://img.wonderhowto.com/img/31/88/63592279411864/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

The result is my default DNS and a new sort of prompt which allows me to ask questions of DNS. So I'll ask my DNS "who is google.com?" by typing `google.com` and my DNS server responds back with the public IP address of google.com.

<img src="http://img.wonderhowto.com/img/68/20/63592279635301/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Right now I was asking questions to the DNS with the IP **2a02:2148:82:8053::53**. I'm going to change that IP to our router's IP by typing `server 192.168.1.1`.

<img src="http://img.wonderhowto.com/img/72/17/63592279944926/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Now my router is the primary DNS server. If I type a webpage on my prompt and click enter;

<img src="http://img.wonderhowto.com/img/43/85/63592280177520/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Look at that! As I said before, I entered on the filter search bar the command `ip.addr==192.168.1.1` which will capture all the traffic coming through that IP address. The traffic is quite small because the website I requested is almost a blank page. You can have a look at it. I chose that website because that way the result of the capture would be small, thus easier for me to explain it to you.

Let's dig a little big deeper, shall we?

<img src="http://img.wonderhowto.com/img/31/61/63592280786317/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

I took a closer screenshot of the middle section of the previous picture so you can be able to see better. Does this look familiar to you? This is what I love about Wireshark. Wireshark breaks down communication in the layers of the OSI model. You can learn networking just by looking at a capture file(if you are patient and curiour enough to learn).

At the very very bottom, as physical as it can get we can see the Frame layer which is actually the physical layer of the OSI model.

<img src="http://img.wonderhowto.com/img/83/35/63592281114504/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

At the first line we can see how many bytes were actually sent on the wire. The next layer is the Ethernet layer, aka Datalink layer. What do we expect to see there? Let's find out.

<img src="http://img.wonderhowto.com/img/57/24/63592281383973/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

That's right, MAC addresses. More specifically, my source MAC address which is **d0:67:e5:27:5e:75**. You don't believe me? I'll prove that one as well.

<img src="http://img.wonderhowto.com/img/88/53/63592281745879/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

Looking back at the Datalink layer we can also see the destination MAC Address, how amazing is that? It get's even more amazing, trust me. The 3rd layer is the network layer and since Wireshark never fails to amaze, it shows us the source(my IP) and destination(DNS,router) IP address combined with the IP protocol, in that case, IPv4. It shows more detailed info than just the source and destination IP address but that's what we care about right now.

Finally, we come to the point that started the entire discussion, the UDP protocol.

<img src="http://img.wonderhowto.com/img/23/68/63592282283364/0/networking-foundations-exploring-udp-via-wireshark-part-1.w654.jpg"/>

DNS actually uses UDP as its protocol in the transport layer and if we look inside it shows that my computer is coming from the source port 63715, going to the destination port 53. What that says is that my computer contacted the DNS server by going to a destination port 53 and came from a source port 63715. Port 53 is a well known DNS port, as in all the DNSs in the world respond on port UDP 53. That's where they expect to receive request for and the computers will ask questions directed at UDP port 53 of their DNS server.

What about the 63715 port though? Well, Windows generated a dynamic port, this is not a well known port, this is considered my source port saying "Hey, my question is coming from source port 63715" and when the DNS gets the request for the webpage we wanted to go to, it's going to know where to send its response back. But Windows expected that, they expected to get a response back on that source port and that's actually one of the reasons of why DNS uses UDP. It's kind of an immediate response. Our computer says "Hey, I want to know who that webpage really is?" and the DNS says "Ok, here is your answer.". So all the communication that is happening between those 2 devices is "What's this?" "Here's your answer.", "What's this?" "Here's your answer."

It would be a waste of time to say "Let's build a session between us, are you ok talking?" and the DNS responds back "Yes, let's build a session" and our computer saying "ok". After that session was built our computer would make its request and it would wait for acknowledgement from the server once again. You see where I'm going with it? Why do you need all that overhead just to get the answer to who is google.com or whatever. DNS is geared in such a way that when you say "who is google.com?" and your computer doesn't get an answer back, it's configured to say "Well, I hope it got there, but I don't think it got there because I didn't get an answer back, let me ask again."

That's the idea of UDP ladies and gentlemen. If you look closer to the capture results, our computer makes a request for hmpg.net.forthnet.lan. What is that? And what is the request `A hmpg.net.forthnet.lan` or `AAAA hmpg.net.forthnet.lan`?

<img src="http://img.wonderhowto.com/img/67/71/63592325343318/0/networking-foundations-exploring-udp-via-wireshark-part-2.w654.jpg"/>

So our computer made its request to the DNS server and asked "Hey, I want to find out what is the IP address for `hmpg.net.forthnet.lan`?" Wait what? Where did that come from? I typed in the DNS prompt `hmpg.net`, right? Why on earth did it do that? Well, if I go back to my terminal and type `ipconfig /all` there is some info about the "DNS Suffix".

<img src="http://img.wonderhowto.com/img/37/20/63592326090635/0/networking-foundations-exploring-udp-via-wireshark-part-2.w654.jpg"/>

One of the things you can do with DNS is to assign to computers a default DNS Suffix. Suffix, where does that go, at the end, right? That would allow somebody to say, if they assign the forthnet.lan or whatever suffix, "I want to ping server". Then it's going to automatically ping `server.forthnet.lan`. Maybe that's my DNS domain that I have for my house or something like that. So immediately when I try to ping hmpg.net or any website domain our computer says "Well, I'm going to try and look up hmpg.net.forthnet.lan" and the DNS replies "I don't know about hmpg.net.forthnet.lan, there's no such name." as you can see on the picture.

Let's see the communication as a whole. So our computer says "Who is hmpg.net.forthnet.lan?", DNS replies "No such thing.". Did you notice the `A` in the `A hmpg.net.forthnet.lan`? That's our computer asking for an A record. In DNS language that's the **address**. Our computer comes back and says "Oh well, I would like an AAAA record for hmpg.forthnet.lan. Do you know what that is now?" and DNS replies "No such name.". So what's the different between A and AAAA you may be asking. The A record is looking for the IPv4 address of hmpg.net.forthnet.lan, while the AAAA record is actually the IPv6 address. Since our computer didn't get an answer for the IPv4 address, it thought "That didn't go well, maybe the webpage is on TCP/IPv6.".

Our computer keeps asking, but this time it's asking "Ok, do you have an A(IPv4) record(address) for hmpg.net?" and DNS replies "Actually, I do!".

Let's go back to the bad boy Wireshark. I'll go back to the capture file, select the 35th packet as shown in the picture above and expand the Domain Name System(response);

<img src="http://img.wonderhowto.com/img/25/42/63592327687928/0/networking-foundations-exploring-udp-via-wireshark-part-2.w654.jpg"/>

You can see the "Queries" section showing our query/request for an A record for hmpg.net and the answer from the DNS server being the IP address of hmpg.net.

Do you see how Wireshark can be really handy? With Wireshark you can have a full in-depth look about what's going on behind the scenes and explore all kinds of communications across the network. What I want you to do now is play around with Wireshark. Install it, type the commands ipconfig /all and see your MAC address, your DNS server, your LAN IP. Type nslookup in the command promt, change the primary DNS server and filter your network traffic with it.

That's been it for now. With this part we finish our discussion about UDP. I hope the Wireshark demo made you excited and helped you understand more than just the UDP concept. I hope the article has been informative for you and I would like to thank you for taking the time to read it. Have an amazing day and stay awesome.

Later...
