---
weight: 10
title: "یافتن دستگاه سانسور کننده ی DNS"
description: "یافتن Middlebox و منشاء اختلالات و فیلترینگ. ساخت گراف مسیر packet ها. معرفی TraceVis"
tags: ["سانسور", "DNS", "دی ان اس", "Do53", "DNS over UDP", "بررسی DNS", "فیلتر", "Middlebox", "فیلترنت", "اینترنت", "TraceVis", "بررسی سانسور اینترنت", "سانسور اینترنت"]
images:
- "/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-copy-dig-packet-twitter-8888.png"
---

# یافتن دستگاه دستکاری کننده ی DNS

به دلیل عدم شفافیت ISP ها و حکومت در اعمال سانسور و نبود حد و مرز در آن، نیاز است تا دریابیم که چه کسی مسئول یک اختلال و سانسور است. 
همچنین این بخش مقدمه ای برای بررسی بیشتر و اثبات رفتار سیستم سانسور در مسدود کردن برخی ارتباطات است که در بخش «سانسور random» آن را بررسی خواهیم کرد.

اگر از DNS ارایه شده توسط ISP استفاده می کنید احتمالا به صورت پیشفرض این خود ISP است که دارد جواب DNS را دستکاری می کند. 
اما در مورد DNS های دیگر و بخصوص موارد خارج از ایران، بررسی هایی می توان انجام داد.

شیوه ی کار به این صورت است که یک سری درخواست DNS ارسال کنیم که هر بار یک مرتبه مقدار TTL  در لایه ی IP ی آن افزایش می یابد. 
با توجه به هدف فعلی بهترین ابزار برای این کار [scapy](https://scapy.net/) است. 
ابزاری که این امکان را فراهم می کند تا یک packet جدید بسازید و یا یک packet قبلی را با کمی تغییرات دوباره ارسال کنید. 

در ادامه دو روش را بررسی می کنیم:

- ساختن یک packet ساده
- کپی یک packet واقعی و تغییر جزئی آن.

در واقع نباید بین این دو روش تفاوتی باشد، اما شواهد در بعضی شبکه ها، خلاف آن را ثابت می کند. 
همچنین در مطالب بعدی وجود cache برای درخواست های DNS نیز بررسی می شوند.

طی بررسی این تحقیق، یک ابزار هم ساخته شده به اسم [TraceVis](https://github.com/wikicensorship/tracevis/) که علاوه بر امکان انجام هر دو روش گفته شده به صورت راحت تر، یک گراف هم از کل مسیر می سازد. 
به همراه چند قابلیت دیگر.

<div dir="ltr">

![tracevis dns over udp mci](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-dns-mci.png)
</div>

اما برگردیم به اصل مطلب و بررسی جزئیات.

## ساختن یک packet ساده ی DNS over UDP :

ما میتوانیم این یک فایل با پسوند .py بسازیم و این کد را در آن قرار دهیم و یا درون برنامه ی Scapy این کد را اجرا کنیم.

<div dir="ltr">

```py {linenos=table}
#!/usr/bin/env python3
from scapy.all import *
for myttl in range(30):
       dns_request = IP(dst="8.8.8.8", id=RandShort(), ttl=myttl)/UDP(dport=53)/DNS(rd=1, id=RandShort(), qd=DNSQR(qname="www.google.com"))
       print(">>>request:" + "   ip.dst: " + dns_request[IP].dst + "   ip.ttl: " + str(myttl))
       req_answer = sr1(dns_request, verbose=0, timeout=3)
       if req_answer is not None:
               print("   <<< answer:" + "   ip.src: " + req_answer[IP].src + "   ip.ttl: " + str(req_answer[IP].ttl))
               print("   " + req_answer.summary())
       print(" · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · ")
```
</div>

1. (اگر در خود scapy از این کد استفاده می کنید، این خط را وارد نکنید.) 

2. (اگر در خود scapy از این کد استفاده می کنید، این خط را نیز وارد نکنید.) 

3. بازه ی مقدار TTL را مشخص می کردیم. از 0 تا 30.

4. یک packet ساده ی DNS over UDP ساختیم (بدون لایه ی Ether) . همچنین برای جلوگیری از ignore شدن packet، از ID های random استفاده کردیم.

5. در جواب دریافتی، مقدار IP ی مقصد و همچنین TTL ای که تنظیم کردیم را چاپ می کنیم.

6. packet ساخته شده را بدون نمایش log و حداکثر زمان انتظار سه ثانیه ارسال کردیم.

7. بررسی می کنیم که جواب دریافتی پوچ نباشد. 

8. مقدار IP ی مبدا ای که در حال جواب دادن به ما هست را به همراه TTL ای که در زمان دریافت packet مشاهده کردیم را چاپ می کنیم.

9. یک خلاصه از جواب دریافتی هر چه که هست (DNS یا ICMP) را چاپ می کنیم.

10. فقط خطی جدا کننده را چاپ می کنیم.

<div dir="ltr">

![scapy dns over udp 8.8.8.8 googledotcom](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-dns-over-udp-8888-googledotcom.png)
</div>

از سرور مورد تست (AS48147-امین داده) تا Hop یازدهم، به جای DNS جواب از نوع ICMP دریافت کردیم که به معنای این است که TTL صفر شده است و آن hop نیز مقصد مورد نظر ما نیست. 
بعد از آن، تمام جواب هایی که دریافت کردیم مقدار 56 در TTL داشتند و همراه با پاسخ DNS.

<div dir="ltr">

```c
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 0
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 1
   <<< answer:   ip.src: 172.22.8.3   ip.ttl: 255
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 2
   <<< answer:   ip.src: 172.22.6.2   ip.ttl: 254
      IP / ICMP 172.22.6.2 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 3
   <<< answer:   ip.src: 172.22.9.2   ip.ttl: 62
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 4
   <<< answer:   ip.src: 10.202.5.49   ip.ttl: 252
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'"  / Padding
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 5
   <<< answer:   ip.src: 10.202.4.83   ip.ttl: 251
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'"  / Padding
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 6
   <<< answer:   ip.src: 85.185.45.133   ip.ttl: 250
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'"  / Padding
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 7
   <<< answer:   ip.src: 10.202.4.206   ip.ttl: 251
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'"  / Padding
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 8
   <<< answer:   ip.src: 213.202.4.172   ip.ttl: 249
      IP / ICMP 213.202.4.172 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 9
   <<< answer:   ip.src: 213.202.5.239   ip.ttl: 238
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'"  / Padding
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 10
   <<< answer:   ip.src: 216.239.48.133   ip.ttl: 50
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 11
   <<< answer:   ip.src: 216.239.51.115   ip.ttl: 52
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 12
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 13
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 14
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 15
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 16
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 17
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 18
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 19
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 20
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 21
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 22
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 23
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 24
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 25
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 26
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 27
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 28
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 29
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · ·
```
</div>

همچنین به نظر می رسد که در این تست تا Hop سوم، که با آدرس 172.22 شروع می شود، همچنان در شبکه ی امین داده قرار دارد. 
بعد از آن تا Hop هفتم، در شبکه ی شرکت ارتباطات زیرساخت (AS12880). 
علاوه بر آن، مسیر برگشت در Hop های 7 و 8 یکسان است. 
که می تواند یا ناشی از تغییرات routing در شبکه باشد و یا نشان از جایی که firewall برای ایزوله کردن شبکه قرار دارد.

<div dir="ltr">

![AS12880 ITC Iran gateway](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/AS12880-ITC-Iran-gateway.png)
</div>

در ادامه، hob های 8 و 9 به در عمان‌تل (AS8529) قرار دارد. 
اما نکته ای وجود دارد و آن این است که در جواب ICMP درون این شبکه، بین hop های 8 و 9، حدود 11 واحد مقدار TTL تفاوت دارد. 
بدین صورت که اگر 255 را منهای 238 کنیم می شود 17. 
یعنی ما با 9 تا Hop به آن میرسیم اما پاسخ با 17 تا Hop به ما می رسد. 
این می تواند به این دلیل باشد که hop نهم، مسیر متفاوتی برای پاسخ دادن انتخاب می کند که با routing اصلی متفاوت است.

<div dir="ltr">

![AS8529 OmanTel transit to Iran](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/AS8529-OmanTel-transit-to-Iran.png)
</div>

در ادامه Hop های 10 و 11 در در زیرساخت گوگل قرار دارند. 
اما هنوز درخواست ما به مقصد نرسیده است. 
اگر 64 را منهای 50 کنیم، می شود 14. 
یعنی ما با 10 تا Hop به زیرساخت گوگل می رسیم و 14 واحد مسیر برگشت پیام ICMP است.

در انتها بعد از hop  دوازدهم و دریافت جواب از مقصد مورد نظر، TTL های دریافتی همیشه 56 هستند. 
که یعنی جواب، 8 تا Hop مسیر را طی می کند تا به ما برسد! (این را نیز در مطالب بعدی بیشتر بررسی خواهیم کرد.)

ما آدرس درخواستی را از `www.google.com` به `www.twitter.com` تغییر می دهیم تا ببینیم در کجای مسیر، DNS Hijacking انجام می شود: 

<div dir="ltr">

![scapy dns over udp 8888 twitter.com](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-dns-over-udp-8888-twitter.png)
</div>

بعد از Hop چهارم به بعد، یعنی در شبکه ی شرکت ارتباطات زیرساخت، از ارسال Packet ها DNS ما که حاوی آدرس Twitter است جلوگیری می شود. 
لاگ کامل:

<div dir="ltr">

```c
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 0
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 1
   <<< answer:   ip.src: 172.22.8.3   ip.ttl: 255
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.twitter.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 2
   <<< answer:   ip.src: 172.22.6.2   ip.ttl: 254
      IP / ICMP 172.22.6.2 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 3
   <<< answer:   ip.src: 172.22.9.2   ip.ttl: 62
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.twitter.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 4
   <<< answer:   ip.src: 10.202.5.49   ip.ttl: 252
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.twitter.com.'"  / Padding
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 5
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 6
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 7
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 8
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 9
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 10
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 11
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 12
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 13
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 14
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 15
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 16
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 17
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 18
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 19
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 20
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 21
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 22
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 23
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 24
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 25
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 26
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 27
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 28
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 29
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
```
</div>

(شبکه های مختلف، رفتار متفاوتی دارند و ممکن است این درخواست در شبکه ی شما بدون مشکل عبور کند.)

اما سوالی به وجود می آید که آیا این یک سانسور عادی است و یا یک لیست سفید از Packet های DNS است؟ برای تست آن باید یک packet واقعی را کپی و دوباره ارسال کنیم.


## کپی یک packet واقعی ی DNS over UDP و تغییر جزئی آن:

یکی از قابلیت های مهم در برنامه ی Scapy امکان کپی کردن یک packet واقعی به صورت Hex و تغییر و ارسال مجدد آن است.

راه های مختلفی برای دریافت Hex یک Packet دارد که در در بخش «آنالیز packet ها» بیشتر توضیح داده خواهد شد.

در اینجا برای سادگی کار، از tcpdump استفاده کردیم. 

ابتدا این دستور را اجرا می کنیم:

<div dir="ltr">

```sh
$ sudo tcpdump -s 0 -x 'port 53'
```
</div>

به معنای اینکه مقدار hex از لایه ی IP به بعد و فقط ترافیک پورت 53.

بعد در یک پنجره ی دیگر، دستور مورد نظرمان را اجرا می کنیم:

<div dir="ltr">

```sh
$ dig www.google.com @8.8.8.8
```
<br/>

![dig google.com 8.8.8.8](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/dig-google-8888.png)
</div>

به صورتی که در تصویر مشخص شده بخشی از Packet مورد نظرمان که به صورت Hex است را کپی می کنیم:

<div dir="ltr">

![tcpdump x port 53 hex](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tcpdump-x-port-53-hex.png)
</div>

برنامه ی scapy را اجرا می کنیم:

<div dir="ltr">

![scapy start](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-start.png)
</div>

ابتدا دستور زیر را وارد می کنیم و enter می زنیم:

<div dir="ltr">

```py
>>> mypacket = IP(import_hexcap())
```

<br/>

![scapy import hex](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-import-hex.png)
</div>

بعد محتوای کپی شده را paste می کنیم و enter میزنیم تا دوباره علامت <‌<‌< ظاهر شود:

<div dir="ltr">

![scapy show packet details](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-show-packet-details.png)
</div>

اگر نام متغیر ساخته شده را به تنهایی وارد کنید، جزئیات کامل این packet نمایش داده خواهد شد. 
اگر بجز لایه ی IP بقیه فقط Hex نمایش داده شد، کپی را به درستی انجام **ندادید**. 

برای اینکه بتوانیم این packet را دوباره ارسال کنیم و انتظار جواب مثبت داشته باشیم، نیاز داریم که تغییراتی در آن انجام دهیم. 
کدی که در بالا استفاده کرده بودیم را کمی تغییر می دهیم و enter میزنیم تا دستور اجرا شود:

<div dir="ltr">

```py {linenos=table}
>>> for myttl in range(30):
       mypacket[IP].ttl = myttl
       mypacket[IP].id = RandShort()
       mypacket[DNS].id = RandShort()
       mypacket[UDP].sport = RandShort()
       print(">>>request:" + "   ip.dst: " + mypacket[IP].dst + "   ip.ttl: " + str(myttl))
       del(mypacket[IP].chksum)
       del(mypacket[UDP].chksum)
       del(mypacket[IP].len)
       req_answer = sr1(mypacket, verbose=0, timeout=3)
       if req_answer is not None:
               print("   <<< answer:" + "   ip.src: " + req_answer[IP].src + "   ip.ttl: " + str(req_answer[IP].ttl))
               print("   " + req_answer.summary())
       print(" · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · ")
```
</div>

در این کد، TTL پیشفرض که 64 بود را به مقدار دلخواه خود تغییر دادیم. 
برای ID ی لایه ی IP ، برای ID ی DNS و برای پورت مبدا در UDP یک مقدار عددی random تعیین می کنیم. 
Checksum های مربوط به IP و UDP را حذف می کنیم تا هر بار دوباره توسط برنامه ی Scapy محاسبه و تعیین شود.

<div dir="ltr">

![scapy copy dig packet google.com 8.8.8.8](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-copy-dig-packet-google-8888.png)
</div>

تا Hop یازدهم تقریبا رفتار شبیه قبل است. 
(آدرس مورد آزمایش `www.google.com` است.)

اما بعد از آن، ما رفتاری کاملا متفاوت می بینیم و هر بار با یک TTL متفاوت با فاصله ی 8 Hop تا 14 Hop. 
به معنای 6 واحد تفاوت در بین پاسخ ها.

لاگ کامل:

<div dir="ltr">

```c
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 0
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 1
   <<< answer:   ip.src: 172.22.8.3   ip.ttl: 255
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 2
   <<< answer:   ip.src: 172.22.6.2   ip.ttl: 254
      IP / ICMP 172.22.6.2 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 3
   <<< answer:   ip.src: 172.22.9.2   ip.ttl: 62
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 4
   <<< answer:   ip.src: 10.202.5.49   ip.ttl: 252
      IP / ICMP 10.202.5.49 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 5
   <<< answer:   ip.src: 10.202.4.83   ip.ttl: 251
      IP / ICMP 10.202.4.83 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 6
   <<< answer:   ip.src: 85.185.45.133   ip.ttl: 250
      IP / ICMP 85.185.45.133 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 7
   <<< answer:   ip.src: 10.202.4.206   ip.ttl: 251
      IP / ICMP 10.202.4.206 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 8
   <<< answer:   ip.src: 213.202.4.172   ip.ttl: 249
      IP / ICMP 213.202.4.172 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 9
   <<< answer:   ip.src: 213.202.5.239   ip.ttl: 238
      IP / ICMP 213.202.5.239 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 10
   <<< answer:   ip.src: 216.239.48.87   ip.ttl: 49
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.google.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 11
   <<< answer:   ip.src: 108.170.233.243   ip.ttl: 55
      IP / ICMP 108.170.233.243 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 12
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 115
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 13
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 116
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 14
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 119
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 15
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 119
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 16
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 119
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 17
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 116
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 18
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 55
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 19
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 119
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 20
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 114
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 21
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 120
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 22
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 116
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 23
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 120
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 24
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 119
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 25
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 116
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 26
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 27
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 120
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 28
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 29
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 56
      IP / UDP / DNS Ans "216.58.209.132" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
```

</div>

حالا همان مسیر قبلیِ کپی کردن Packet را در مورد آدرس `www.twitter.com` انجام می دهیم که در شیوه ی قبلی بعد از Hop چهارم وارد سیاهچاله می شد.

<div dir="ltr">

![scapy copy dig packet twitter 8888](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-copy-dig-packet-twitter-8888.png)
</div>

در این شرایط می بینیم که دوباره از Hop چهارم وارد سیاهچاله شده است اما از Hop نهم جوابی ارسال می شود که هم آدرس مربوط به صفحه ی سانسور جمهوری اسلامی برگردانده می شود و هم  TTL آن بسیار پایین است. 
این  مقدار TTL دریافتی با افزایش TTL ارسالی، افزایش می یابد. 
لاگ کامل:

<div dir="ltr">

```c
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 0
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 1
   <<< answer:   ip.src: 172.22.8.3   ip.ttl: 255
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.twitter.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 2
   <<< answer:   ip.src: 172.22.6.2   ip.ttl: 254
      IP / ICMP 172.22.6.2 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 3
   <<< answer:   ip.src: 172.22.9.2   ip.ttl: 62
      IP / ICMP / IPerror / UDPerror / DNS Qry "b'www.twitter.com.'" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 4
   <<< answer:   ip.src: 10.202.5.49   ip.ttl: 252
      IP / ICMP 10.202.5.49 > 185.*.*.* time-exceeded ttl-zero-during-transit / IPerror / UDPerror / Raw
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 5
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 6
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 7
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 8
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 9
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 1
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 10
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 2
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 11
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 3
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 12
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 4
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 13
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 5
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 14
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 6
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 15
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 7
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 16
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 8
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 17
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 9
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 18
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 10
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 19
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 11
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 20
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 12
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 21
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 13
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 22
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 14
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 23
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 15
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 24
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 16
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 25
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 17
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 26
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 18
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 27
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 19
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 28
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 20
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · · 
>>>request:   ip.dst: 8.8.8.8   ip.ttl: 29
   <<< answer:   ip.src: 8.8.8.8   ip.ttl: 21
      IP / UDP / DNS Ans "10.10.34.35" 
 · · · − − − · · ·     · · · − − − · · ·     · · · − − − · · ·
```
</div>

برای اطمینان از اینکه این رفتار چطور عمل می کند، که آیا افزایش نسبت به دفعات درخواست است و یا نسبت به TTL ارسالی، یک آزمایش بیشتر انجام می دهیم. 
برای این کار از همان کد قبلی استفاده می کنیم اما این بار range آن را از 30 تا 128 قرار می دهیم و مرتبه ی افزایش در هر بار را 3 قرار می دهیم.

<div dir="ltr">

![scapy copy dig packet twitter 8888 ttl 30-128 3-step](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/scapy-copy-dig-packet-twitter-8888-ttl-30-128-step-3.png)
</div>

با توجه به این آزمایش، به این نتیجه می رسیم که سیستم سانسور جمهوری اسلامی در صورت مواجهه با یک packet مرسوم DNS over UDP اگر آدرس درخواستی ممنوعه است، مقدار TTL دریافتی را در جواب DNS با محتوای آدرس صفحه ی سانسور قرار می دهد و آن را به سمت کاربر ارسال می کند.

دلیل اینکه TTL ارسالی باید بیشتر از 8 باشد، این است که سیستم سانسور در این شبکه، در Hop پنجم قرار دارد. 
به این صورت که ما با TTL با مقدار 9 را ارسال می کنیم، سیستم سانسور TTL با مقدار 5 را دریافت می کند، آن را کپی کرده و در پاسخ به ما ارسال می کند و مقدار 1 به ما می رسد. 
و این بدین معنی است که سیستم سانسور در تست های ما از TTL با مقدار 4 در حال دریافت packet های ما است، اما فقط پاسخ آن است که به ما نمی رسد.

این یافته، وسعت بیشتری دارد که در مطالب بعدی به آن بیشتر خواهیم پرداخت. 

## استفاده از TraceVis

برای ساده سازی و درک بهتر این مسئله و وضعیت شبکه و سانسور، اقدام به ساخت ابزاری کردیم به اسم [TraceVis](https://github.com/wikicensorship/tracevis/) که می توانید به راحتی تست های خود را انجام دهید.

ابتدا dependency ها را نصب کنید:

<div dir="ltr">

```sh
python3 -m pip install -r requirements.txt

```
</div>

و بعد تست را به این صورت اجرا کنید:

<div dir="ltr">

```sh
python ./tracevis.py --dns

```
</div>

و یا :

<div dir="ltr">

```sh
python ./tracevis.py --dns -n "test-prefix" -i "1.0.0.1,8.8.4.4,9.9.9.9,5.200.200.200"

```
</div>

به دلیل اینکه اولویت با دریافت جواب از Hop ها است و درستی نتیجه، ممکن است که این برنامه آهسته تر از توقع شما باشد.

<div dir="ltr">

![tracevis log](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-log.jpg)
</div>

بعد از پایان یافتن تست که سه بار تکرار می شود، شما دو فایل دریافت خواهید کرد.

<div dir="ltr">

```sh
 **********************************************************************
saving measurement data...
saved: ./output/test-prefix-dns-tracevis-20211212-1551.json
· · · - · -     · · · - · -     · · · - · -     · · · - · -
saving measurement graph...

saved: ./output/test-prefix-dns-tracevis-20211212-1551.html
· · · - · -     · · · - · -     · · · - · -     · · · - · -
```
</div>

یک فایل نتیجه ی آزمایش، از نوع json است که سازگار با ساختار RIPE Atlas است و یک فایل HTML که حاوی گراف.

<div dir="ltr">

![tracevis measurement json file](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-measurement-file.jpg)
</div>

این سازگاری به ما این امکان را می دهد تا علاوه بر داشتن یک ساختار استاندارد، به سادگی قابلیت ساخت گراف از طریق نتایج تست های traceroute در Probe های RIPE Atlas را نیز داشته باشیم.

در این گراف، به صورت پیشفرض رنگ های سرد برای آدرسی که فیلتر نیست در نظر گرفته شده و رنگ های گرم، برای آدرس فیلتر شده.

در تصویر زیر می بنید که هر دو مسیر به یک نقطه رسیدند و این به آن معنی است که جواب درخواست ها برای هر دو آدرس توسط یک دستگاه داده شده اند.

<div dir="ltr">

![tracevis graph html file 5.200.200.200](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-graph.jpg)
</div>

اما این وضعیت برای تمام آدرس ها صدق نمی کند. و اگر مسیر packet از سیستم سانسور بگذرد، شما حالتی متفاوت خواهید دید.

<div dir="ltr">

![tracevis graph 1.0.0.1 google.com](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-graph-1001-googlecom.jpg)
</div>

در این تصویر درخواست به آدرس گوگل از شبکه ی داخلی خارج و به سرور مقصد رسید.

اما در مورد آدرس توییتر، مسیر متفاوت است:

<div dir="ltr">

![tracevis graph 1.0.0.1 twitter.com middlebox](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-graph-1001-twittercom-middlebox.jpg)
</div>

وقتی در توضیحات علاوه بر اسم Middlebox، مقدار back-TTL عدد 6 قرار گرفت، یعنی Middlebox به اندازه ی 6 Hop فاصله دارد.

یعنی در این موقعیت هایی که با پیکان مشخص شده:

<div dir="ltr">

![tracevis graph middlebox](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-graph-middlebox.jpg)
</div>

[این ابزار](https://github.com/wikicensorship/tracevis/) متن باز، قابلیت ها و امکانات بیشتری دارد. مانند امکان trace route با هر نوع packet از نوع TCP و UDP و ساخت گراف.


<div dir="ltr">

![tracevis usage](/images/docs/measure-internet-censorship/DNS/find-DNS-hijacker/tracevis-usage.jpg)
</div>

منتظر مشارکت شما در بهبود و توسعه ی این پروژه هستیم.
