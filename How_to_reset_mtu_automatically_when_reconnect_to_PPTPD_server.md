# How to reset mtu automatically when reconnect to PPTPD server?

Recently I encountered an interesting thing, I purchased a VPS(os:Centos) located in HongKong, then deployed a PPTPD server.

And when I connect to this service by PPTP protocol from Windows 7 PC, the host generate an ethernet device shortly, with the prefixed name of "ppp", mtu 1396 as default.

Type command "*ifconfig -a*" to check:
```
ppp0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1396
        inet 192.168.0.1  netmask 255.255.255.255  destination 192.168.0.235
        ppp  txqueuelen 3  (Point-to-Point Protocol)
        RX packets 31572  bytes 3340415 (3.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 103428  bytes 70070905 (66.8 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
But regretful there's a bad experience I can't visit some domestic(acutally not only) websites because of different mtu values: The Windows is 1496, but 1396 on Centos(more details [refer to:wikipedia](https://en.wikipedia.org/wiki/Maximum_transmission_unit)).

## So how can we resolve?

I google and seek a few remedies, the fastest way is to reset the mtu value of specific device:
```
[mmocean@VM_0_2_centos ~]$ sudo ifconfig ppp0 mtu 1496
```

Woooa, it goes into effect! Enjoy it!

But wait, why can't I visit again when I reconnect to the service? Because the setting above is temporal!

I have tried some permanent approaches like *vim /etc/ppp/options.pptpd*, but it seems none to them works well.

Ok and the identity may be various, and I only change the mtu value of pptpd, so I compose a shell script to achieve this goal:
```
#!/bin/bash

while(true)
do
	#ether_list=$(ifconfig|grep encap|awk -F "Link" '{print $1}')
	ether_list=$(ifconfig|grep ppp|grep mtu|awk -F ":" '{print $1}')
	#echo $ether_list
	for ether in $ether_list
	do
	    #echo $ether
		ifconfig $ether mtu 1496 >/dev/null 2>&1
	done
	sleep 10
done
```
Despite this script can launch by crontab per 10s(resemble *sleep 10* above), but it need configuer multiple items(minium precision:1 min),

## start like this 

```
[root@VM_0_2_centos ~]# nohup /root/mtu_set.sh >/dev/null 2>&1 &
```
It's really ok now!

