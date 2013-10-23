---
layout: post
title: "常用加密算法库支持的rsa填充模式"
description: ""
category: ""
tags: [RSA]
---
{% include JB/setup %}

### 1. openSSL 
RSA_PKCS1_PADDING（RSAES-PKCS1-v1_5/RSASSA-PKCS1-v1_5填充） 
RSA_SSLV23_PADDING（SSLv23填充）
RSA_NO_PADDING（不填充）
RSA_PKCS1_OAEP_PADDING   （RSAES-OAEP填充，强制使用SHA1，加密使用）
RSA_X931_PADDING（X9.31填充，签名使用）
RSA_PKCS1_PSS_PADDING（RSASSA-PSS填充，签名使用）

### 2. PolarSSL
RSA_PKCS_V15（RSAES-PKCS1-v1_5/RSASSA-PKCS1-v1_5填充）
RSA_PKCS_V21（RSAES-OAEP填充，可选不同hash，加密使用；RSASSA-PSS填充，签名使用）

### 3. CyaSSL
支持RSAES-PKCS#1-v1.5
不支持RSAES-OAEP(到2.7.0尚未实现)

### 4. Cryptlib
支持RSAES-PKCS#1-v1.5
不支持RSAES-OAEP(不打算实现）

### 5. LibTomCrypt
LTC_LTC_PKCS_1_V1_5（RSAES-PKCS1-v1_5/RSASSA-PKCS1_v1_5填充）
LTC_LTC_PKCS_1_OAEP  （RSAES-OAEP填充，可选不同hash，加密使用）
LTC_LTC_PKCS_1_PSS（RSASSA-PSS填充，签名使用）
