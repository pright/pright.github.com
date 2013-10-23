---
layout: post
title: "linux协议栈(3.11)处理组播数据包流程"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

```
ip_rcv -> ip_rcv_finish -> ip_route_input_noref -> ip_check_mc_rcu**
                                                -> ip_route_input_mc -> fib_validate_source -> __fib_validate_source
                        -> dst->input -> ip_local_deliver/ip_mr_input
```


```
1816 int ip_route_input_noref(struct sk_buff *skb, __be32 daddr, __be32 saddr,
1817                          u8 tos, struct net_device *dev)

1823         /* Multicast recognition logic is moved from route cache to here.
1824            The problem was that too many Ethernet cards have broken/missing
1825            hardware multicast filters :-( As result the host on multicasting
1826            network acquires a lot of useless route cache entries, sort of
1827            SDR messages from all the world. Now we try to get rid of them.
1828            Really, provided software IP multicast filter is organized
1829            reasonably (at least, hashed), it does not result in a slowdown
1830            comparing with route cache reject entries.
1831            Note, that multicast routers are not affected, because
1832            route cache entry is created eventually.
1833          */
1834         if (ipv4_is_multicast(daddr)) {
1835                 struct in_device *in_dev = __in_dev_get_rcu(dev);
1836 
1837                 if (in_dev) {
                             // 送给本机的返回1（加入了相应组播组）
1838                         int our = ip_check_mc_rcu(in_dev, daddr, saddr,
1839                                                   ip_hdr(skb)->protocol);
1840                         if (our
1841 #ifdef CONFIG_IP_MROUTE
1842                                 ||
1843                             (!ipv4_is_local_multicast(daddr) &&
1844                              IN_DEV_MFORWARD(in_dev)) // 判断是否送给本机或者配置了组播转发
1845 #endif
1846                            ) {
1847                                 int res = ip_route_input_mc(skb, daddr, saddr,
1848                                                             tos, dev, our);
1849                                 rcu_read_unlock();
1850                                 return res;
1851                         }
1852                 }
1853                 rcu_read_unlock();
1854                 return -EINVAL;
1855         }
```
 
```
2371 int ip_check_mc_rcu(struct in_device *in_dev, __be32 mc_addr, __be32 src_addr, u16 proto)

2378         mc_hash = rcu_dereference(in_dev->mc_hash);
2379         if (mc_hash) {
2380                 u32 hash = hash_32((__force u32)mc_addr, MC_HASH_SZ_LOG);
2381 
2382                 for (im = rcu_dereference(mc_hash[hash]);
2383                      im != NULL;
2384                      im = rcu_dereference(im->next_hash)) {
2385                         if (im->multiaddr == mc_addr) // 判断是否已经添加组播组
2386                                 break;
2387                 }
2388         } else {
2389                 for_each_pmc_rcu(in_dev, im) {
2390                         if (im->multiaddr == mc_addr) // 判断是否已经添加组播组
2391                                 break;
2392                 }
2393         }
2394         if (im && proto == IPPROTO_IGMP) {
2395                 rv = 1; // igmp包
2396         } else if (im) { // 组播数据包
2397                 if (src_addr) { // igmpv3增加的指定组播源控制功能（通过IP_ADD_SOURCE_MEMBERSHIP/IP_DROP_SOURCE_MEMBERSHIP配置）
2398                         for (psf=im->sources; psf; psf=psf->sf_next) {
2399                                 if (psf->sf_inaddr == src_addr)
2400                                         break;
2401                         }
2402                         if (psf)
2403                                 rv = psf->sf_count[MCAST_INCLUDE] ||
2404                                         psf->sf_count[MCAST_EXCLUDE] !=
2405                                         im->sfcount[MCAST_EXCLUDE];
2406                         else
2407                                 rv = im->sfcount[MCAST_EXCLUDE] != 0;
2408                 } else
2409                         rv = 1; /* unspecified source; tentatively allow */ // igmp v1/v2
2410         }
2411         return rv;
2412 }
```

```
1421 static int ip_route_input_mc(struct sk_buff *skb, __be32 daddr, __be32 saddr,
1422                                 u8 tos, struct net_device *dev, int our)

             // 源地址为组播地址或者本地广播地址或协议非IP协议则返回
1434         if (ipv4_is_multicast(saddr) || ipv4_is_lbcast(saddr) ||
1435             skb->protocol != htons(ETH_P_IP))
1436                 goto e_inval;
1437 
1438         if (likely(!IN_DEV_ROUTE_LOCALNET(in_dev)))
1439                 if (ipv4_is_loopback(saddr)) // 源地址为环回地址则返回
1440                         goto e_inval;
1441 
1442         if (ipv4_is_zeronet(saddr)) {
1443                 if (!ipv4_is_local_multicast(daddr)) // 源地址全0且目标地址非本地组播地址（224.X.X.X）则返回
1444                         goto e_inval;
1445         } else {
1446                 err = fib_validate_source(skb, saddr, 0, tos, 0, dev,
1447                                           in_dev, &itag);
1448                 if (err < 0)
1449                         goto e_err;
1450         }
1451         rth = rt_dst_alloc(dev_net(dev)->loopback_dev,
1452                            IN_DEV_CONF_GET(in_dev, NOPOLICY), false, false);
1453         if (!rth)
1454                 goto e_nobufs;
1455 
1456 #ifdef CONFIG_IP_ROUTE_CLASSID
1457         rth->dst.tclassid = itag;
1458 #endif
1459         rth->dst.output = ip_rt_bug;
1460 
1461         rth->rt_genid   = rt_genid(dev_net(dev));
1462         rth->rt_flags   = RTCF_MULTICAST;
1463         rth->rt_type    = RTN_MULTICAST;
1464         rth->rt_is_input= 1;
1465         rth->rt_iif     = 0;
1466         rth->rt_pmtu    = 0;
1467         rth->rt_gateway = 0;
1468         rth->rt_uses_gateway = 0;
1469         INIT_LIST_HEAD(&rth->rt_uncached);
1470         if (our) {
1471                 rth->dst.input= ip_local_deliver; // 
1472                 rth->rt_flags |= RTCF_LOCAL;
1473         }
1474 
1475 #ifdef CONFIG_IP_MROUTE
1476         if (!ipv4_is_local_multicast(daddr) && IN_DEV_MFORWARD(in_dev))
1477                 rth->dst.input = ip_mr_input;
1478 #endif
```
 
```
234 /* Given (packet source, input interface) and optional (dst, oif, tos):
235  * - (main) check, that source is valid i.e. not broadcast or our local
236  *   address.
237  * - figure out what "logical" interface this packet arrived
238  *   and calculate "specific destination" address.
239  * - check, that packet arrived from expected physical interface.
240  * called with rcu_read_lock()
241  */
242 static int __fib_validate_source(struct sk_buff *skb, __be32 src, __be32 dst,
243                                  u8 tos, int oif, struct net_device *dev,
244                                  int rpf, struct in_device *idev, u32 *itag)
252         fl4.flowi4_oif = 0;
253         fl4.flowi4_iif = oif;
254         fl4.daddr = src; // src反填入daddr
255         fl4.saddr = dst; // dst这里传入为0
256         fl4.flowi4_tos = tos;
257         fl4.flowi4_scope = RT_SCOPE_UNIVERSE;
258 
259         no_addr = idev->ifa_list == NULL;
260 
261         accept_local = IN_DEV_ACCEPT_LOCAL(idev);
262         fl4.flowi4_mark = IN_DEV_SRC_VMARK(idev) ? skb->mark : 0;
263 
264         net = dev_net(dev);
265         if (fib_lookup(net, &fl4, &res)) // 查路由表是否有dst（0）->src的相应表项
266                 goto last_resort;
267         if (res.type != RTN_UNICAST) {
268                 if (res.type != RTN_LOCAL || !accept_local)
269                         goto e_inval;
270         }
271         fib_combine_itag(itag, &res);
272         dev_match = false;
273 
274 #ifdef CONFIG_IP_ROUTE_MULTIPATH
275         for (ret = 0; ret < res.fi->fib_nhs; ret++) {
276                 struct fib_nh *nh = &res.fi->fib_nh[ret];
277 
278                 if (nh->nh_dev == dev) { // 检查输入接口是否一致
279                         dev_match = true;
280                         break;
281                 }
282         }
283 #else
284         if (FIB_RES_DEV(res) == dev) // 检查输入接口是否一致
285                 dev_match = true;
286 #endif
287         if (dev_match) {
                    // 是否可直达，RT_SCOPE_HOST（发自链路直连的主机）和RS_SCOPE_NOWHERE（发自本机）可直达，否则不可直达（需经过网关）
288                 ret = FIB_RES_NH(res).nh_scope >= RT_SCOPE_HOST;
289                 return ret;
290         }
291         if (no_addr)
292                 goto last_resort;
293         if (rpf == 1)
294                 goto e_rpf;
295         fl4.flowi4_oif = dev->ifindex;
296 
297         ret = 0;
298         if (fib_lookup(net, &fl4, &res) == 0) {
299                 if (res.type == RTN_UNICAST)
300                         ret = FIB_RES_NH(res).nh_scope >= RT_SCOPE_HOST;
301         }
302         return ret;
303 
304 last_resort:
305         if (rpf)
306                 goto e_rpf;
307         *itag = 0;
308         return 0;
309 
310 e_inval:
311         return -EINVAL;
312 e_rpf:
313         return -EXDEV;
314 }
```

主机能接收组播包需满足的条件：
1. 主机加入相应组播组
2. 组播数据包源地址不能为组播地址、本地广播地址或环回地址
3. 组播数据包源地址不能为全0同时目标地址为本地组播地址
4. 主机配置了能和组播源连接的路由


