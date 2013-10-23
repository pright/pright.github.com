---
layout: post
title: "模拟msn登录机制"
description: ""
category: ""
tags: [MSN]
---
{% include JB/setup %}

使用msnp8协议模拟登录msn，写得很草，没做什么异常处理。
测试结果看来msnp8的passport 1.4认证仍旧可以正常登陆服务器，似乎并不需要使用之后更高版本的soap方式认证，可能在连接sb服务器时会有需要？

```
from socket import *
import urllib, httplib


#httplib.HTTPConnection.debuglevel = 1

LOGIN_HOST = 'messenger.hotmail.com'
LOGIN_PORT = 1863
LOGIN_ADDR = (LOGIN_HOST, LOGIN_PORT)


class msn:
    def __init__(self):
        self.usr = None
        self.pwd = None
        self.fd = None
        self.tid = 0
    
    def login(self, usr, pwd):
        self.usr = usr
        self.pwd = pwd

        # connect to dispatch server(messenger.hotmail.com on TCP port 1863) 
        self.fd = socket(AF_INET, SOCK_STREAM)
        self.fd.connect(LOGIN_ADDR)

        # use protocol version 8
        self.send('VER', 'MSNP8 CVR0')
        self.recv(1024)

        # send protocol version information
        self.send('CVR', '0x0409 win 4.10 i386 MSNMSGR 5.0.0544 MSMSGS' \
                + ' ' + self.usr)
        self.recv(1024)

        # send user name
        self.send('USR', 'TWN I' + ' ' + self.usr)
        data = self.recv(1024)

        if len(data) < 3:
            return # need to close socket, FIXME

        # NS 65.54.189.125:1863 0 64.4.9.254:1863
        params = data[2].split()
        if (params[0] != 'NS') and (len(params) < 4):
            return # need to close socket, FIXME

        ns = params[1].split(':')
        nsip = ns[0]
        nsport = int(ns[1])
        nsaddr = (nsip, nsport)

        # connect to notification server 
        self.fd.close()
        self.fd = socket(AF_INET, SOCK_STREAM)
        self.fd.connect(nsaddr)

        # use protocol version 8
        self.send('VER', 'MSNP8 CVR0')
        self.recv(1024)

        # send protocol version information again
        self.send('CVR', '0x0409 win 4.10 i386 MSNMSGR 5.0.0544 MSMSGS' \
            + ' ' + self.usr)
        self.recv(1024)

        # send user name again
        self.send('USR', 'TWN I' + ' ' + self.usr)
        data = self.recv(1024)

        # obtain ticket
        if len(data) < 3:
            return # need to close socket, FIXME

        params = data[2].split()

        if (params[0] != 'TWN') and (len(params) < 3):
            return # need to close socket, FIXME

        ticket = self.auth(params[2])
        
        # send ticket
        self.send('USR', 'TWN S t=' + ticket)
        self.recv(4096)
        
    def gettid(self):
        self.tid += 1
        return str(self.tid)

    def send(self, cmd, params):
        if not self.fd:
            return
        
        s = cmd + ' ' + self.gettid() + ' ' + params + '\r\n'
        self.fd.send(s)
        print '>>> ' + s

    def recv(self, size):
        if not self.fd:
            return

        data = self.fd.recv(size)
        buf = data.split(' ', 2)
        cmd = buf[0]
        if len(buf) >= 3:
            tid = buf[1]
            params = buf[2]
        elif len(buf) == 2:
            tid = buf[1]
            params = ''
        else:
            tid = ''
            params = ''
        
        print '<<< ' + data
        return (cmd, tid, params)

    def auth(self, param):
        # connect to nexus
        nexus = urllib.urlopen('https://nexus.passport.com/rdr/pprdr.asp')

        # print nexus.headers
        # PassportURLs: DARealm=Passport.Net,
        # DALogin=login.live.com/login2.srf,
        # DAReg=https://accountservices.passport.net/UIXPWiz.srf,
        # Properties=https://accountservices.msn.com/editprof.srf,
        # Privacy=https://accountservices.passport.net/PPPrivacyStatement.srf,
        # GeneralRedir=http://nexusrdr.passport.com/redir.asp,
        # Help=https://accountservices.passport.net,C

        login_server = nexus.headers['PassportURLs']. \
            split(',')[1].split('=')[1] # dirty
        login_url = 'https://' + login_server
        login_host = login_server.split('/')[0]
        nexus.close()

        # connect to login server
        ls = httplib.HTTPSConnection(login_host)
        headers = {'Authorization' : 'Passport1.4 OrgVerb=GET,' \
            + 'OrgURL=http%3A%2F%2Fmessenger%2Emsn%2Ecom,sign-in=' \
            + self.usr + ',pwd=' + self.pwd + ',' + param}
        ls.request('GET', login_url, '', headers)
        resp = ls.getresponse()
        ls.close()

        # Passport1.4 da-status=success,
        # from-PP='t=9fyWZvT3XE1LsWme6JmWyK2U2w50UOuf!rKKeZ06CIVw0CKfithhk
        # dyjnYzgYx*rSMHRFtXX0ht*qwS5hQTyOwrq09x!aUhD3Hg3asEnBGAHfAmCtQ08I
        # eGirnRX85V24b&p=9aFvsXXVTGqvskqKdm3LR*JtsQXmuQaazuZdxjkTIzOcU2FH
        # I35etUEFi9jmq3IgEsTjIqq9rqtuNTepjeVdRD5xQ2mpXzq6CcgSjBohh3Ss3mhf
        # 7XU8S1hPkceQ353!VORb!NsGqhpMm*0PNR*Y23EIeWNsleVeO*PeSTuzgz3LqSNE
        # QBVZBebQ$$',ru=http://messenger.msn.com

        auth = resp.getheader('authentication-info'). \
            split()[1].split(',')[1].split('=', 2)[2][:-1] # dirty

        # print auth
        return auth

if __name__ == "__main__":
    m = msn()
    m.login('pright_xj@hotmail.com', 'XXXXXXXXXXXXXXXX')
```

运行结果：

```
>>> VER 1 MSNP8 CVR0
<<< VER 1 MSNP8
>>> CVR 2 0x0409 win 4.10 i386 MSNMSGR 5.0.0544 MSMSGS pright_xj@hotmail.com
<<< CVR 2 1.0.0000 1.0.0000 1.0.0000 http://msgr.dlservice.microsoft.com  http://download.live.com/?sku=messenger 
>>> USR 3 TWN I pright_xj@hotmail.com
<<< XFR 3 NS 207.46.124.151:1863 0 64.4.50.62:1863
>>> VER 4 MSNP8 CVR0
<<< VER 4 MSNP8
>>> CVR 5 0x0409 win 4.10 i386 MSNMSGR 5.0.0544 MSMSGS pright_xj@hotmail.com
<<< CVR 5 1.0.0000 1.0.0000 1.0.0000 http://msgr.dlservice.microsoft.com  http://download.live.com/?sku=messenger 
>>> USR 6 TWN I pright_xj@hotmail.com
<<< USR 6 TWN S ct=1268843598,rver=5.5.4182.0,wp=FS_40SEC_0_COMPACT,lc=1033,id=507,ru=http:%2F%2Fmessenger.msn.com,tw=0,kpp=1,kv=4,ver=2.1.6000.1,rn=1lgjBfIL,tpf=b0735e3a873dfb5e75054465196398e0
>>> USR 7 TWN S t=9YYa!qOc0E63y0JTPqGnVKd*BM4N1HpXaIZWLLWXnvC37XgY9PSTwSE!AG7M5tAE8ofkyjoJzz03xGMi5GAGWqUTbvR3AUwKsJ43KHS1VJP!JygQOAg!2PLzBPFIDEt*oT&p=9zKXmGJ*hG6bBgQ3n6pupPf2aNDdazYCn650Z2!A3e6vmPtnTXFF7Rf4EmA2yrOJtTetrhzR9UVMAfK!poR62Ia8iZgKAF!IXj8gbOA9rwvnGAKk*Bx9xB2uln1Vua*0dRGHQI01KbMinIqNvBMDaZ4lqx5mNO4!aHse6jzgJ7rHpMz2XieyqTSg$$
<<< USR 7 OK pright_xj@hotmail.com Jun 1 0 
```
