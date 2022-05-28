---
title: kali重新获取ip
tags: kali
abbrlink: 41514
date: 2022-05-12 19:58:31
---

~~~~
释放原有ip：#dhclient -r
获取新的ip：#dhclient eth0
查看ip： # ifconfig
查看ip详细信息：#cat /var/lib/dhcp/dhclient.leases
~~~~

