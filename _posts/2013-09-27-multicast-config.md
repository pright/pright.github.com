---
layout: post
title: "组播设置"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

接收组播包，需要设置默认网关（不必真实存在）或者组播路由
route add default gw IP
route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0

原因：
在未设置本地接口情况下使用setsockopt设置IP_ADD_MEMBERSHIP，会根据组播地址去查路由表选择本地接口。如果网络没有配置网关或者组播路由，就无法找到对应的目标地址因而返回No such device的错误。

```
1487 static struct in_device *ip_mc_find_dev(struct net *net, struct ip_mreqn *imr)
1488 {
1489         struct net_device *dev = NULL;
1490         struct in_device *idev = NULL;
1491 
1492         if (imr->imr_ifindex) {
1493                 idev = inetdev_by_index(net, imr->imr_ifindex);
1494                 return idev;
1495         }
1496         if (imr->imr_address.s_addr) {
1497                 dev = __ip_dev_find(net, imr->imr_address.s_addr, false);
1498                 if (!dev)
1499                         return NULL;
1500         }
1501 
1502         if (!dev) {
1503                 struct rtable *rt = ip_route_output(net,
1504                                                     imr->imr_multiaddr.s_addr,
1505                                                     0, 0, 0);
1506                 if (!IS_ERR(rt)) {
1507                         dev = rt->dst.dev;
1508                         ip_rt_put(rt);
1509                 }
1510         }
1511         if (dev) {
1512                 imr->imr_ifindex = dev->ifindex;
1513                 idev = __in_dev_get_rtnl(dev);
1514         }
1515         return idev;
1516 }
```

因此在设置IP_ADD_MEMBERSHIP时建议正确设置本地接口，可避免未正确配置路由导致无法加入组播组的问题。

