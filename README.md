# ZeroTierCore
Language bindings and explorations using embedded ZeroTier network nodes.

## Why?
Integrating [ZeroTier](https://www.zerotier.com) peer-to-peer networking, security and network rules into the application enables the application to directly and securely connect to clients (devices, services or other apps).

## Less is more!
For me, [ZeroTier](https://www.zerotier.com) has changed how I look at (enterprise) networking. Why depend on a vertical (networking) stack of one vendor (AWS, VMware, Azure, Openstack, Docker etc.), or even try to make them work together, if you can make them irrelevant. Just bring the network management right to the Windows or Linux containers or mobile devices. In that respect I am a fan of [LXD](https://www.ubuntu.com/containers/lxd). Amazing what we can do scaling and building up-on a Linux container!

So yes, that is where we currently use ZeroTier, right in the containers and seamslessly connect them amd the clients all together, deploying ZeroTier networks where network rules engine is driven by [tags and capabilities](https://www.zerotier.com/manual.shtml#3) of the devices.

The next logical step is to integrate ZeroTier networking to the apps and services. The app or service will than be able to directly refer to the underlying ZeroTier rules, identity management, and connectivity. We no longer have to manage the specific app / service network requirements at the OS / container level, but can directly do this at the application level.

Every app and service has its private network stack that directly connects with its clients and users. Realtime chat, messaging, service bus protocols, are possbible without reliyng on third component such as RabbitMQ, or cloudbased messaging and backhaul. This also means one or more availabiltiy dependencies less.

## Learning more
This repository is first of all meant for myself, documenting as I explore the new possibilities and try to do some proof of concepts. Occasionally I may ask someone to take a look and help me out! Feel free to jump in.

I am also learning about the underlying protocols of ZeroTier. How to create rules, how they affect and regulate network traffic and how to troubleshoot these rules using tracing. In that respect, I have just made a little python program that combines both ZeroTier and [Scapy](https://github.com/secdev/scapy), a python library to construct and manipulate network packets. We can use this to easily create and observe network traffic and the network rules applied to them.

## The zerotiercore libray
ZeroTierOne can be built as a library, with just the required function as described in [ZeroTierOne.h](https://github.com/zerotier/ZeroTierOne/blob/master/include/ZeroTierOne.h) to embed in a program.

Since I only know some python (and also am looking at using [Nim](http://www.nim-lang.org) we need to create c bindings for these languages to libzerotiercore. This is shown in the subdirectory [ztcore/generate](./ztcore/generate).

## Scapy with ZeroTier embedded
Below is a quick demo of the [ztcode/ztpy/ztnode.py](./ztcode/ztpy/ztnode.py) code. After compiling ZeroTier and creating the binding we can do something like this.

```
./ztnode.py -h
ZTNode, embedding the ZeroTier Networking node in python

Usage:
  ztnode.py
  ztnode.py -n <number>
  ztnode.py -h show help

Options:
  -h --help    show this screen
  -n <number>  number of nodes to start [default: 3]
```

We will go for the default, and have three ZeroTier nodes created and started in our program. Although these nodes are in the same program, they are in different threads and communicate through the standard ZeroTier protocol over UDP. However, since the nodes are initialized from the main thread, we can work with the node objects (through a python class) and easily verify sent and received traffic.

```
# ./ztnode.py
Welcome to Scapy (2.3.3)
ZT
>>> Node n3 has been initialized
Node n1 has been initialized
Node n2 has been initialized
Node n3 is online
Node n1 is online
Node n2 is online

>>>
```

Great. The three nodes have been initialized (e.g. generated their ZeroTier identities) and where able to reach the public root servers. Let's check them out a litte.

```
>>> n1
<__main__.Node object at 0x7fcd35970450>
>>>
```

Yes, it looks a bit cryptic. I should add some descriptive methods!
Let's check out a bit more.

```
>>> n1.ztid
'c0331dd4aa'
>>> n2.ztid
'dd7a663e7f'
>>> n3.ztid
'671f23dc94'
>>>
```
These are their ZeroTier adresses, also known as device ID's.

Next we will check their ZeroTier network connection, as they should have autojoined the network configured in the python script.

```
>>> ffi.string(n1.list_networks().networks.name)
'0_ztcore'
>>>
>>>n1.list_networks().networks.nwid
10641916143449819159L
>>>
```

Ok, let's print the network id in hex.

```
>>> format(n1.list_networks().networks.nwid, 'x')
'93afae5963d24817'
>>>
```
Yes, that is the network I created at [my.zerotier.com](my.zerotier.com). Since it is configured as a public network, there is no need to authorize the new devices, and they get the network details such as the network name and setting automatically. However, network rules that are configured still apply. We'll check that out later.


