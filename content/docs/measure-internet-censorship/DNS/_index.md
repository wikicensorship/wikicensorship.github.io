---
weight: 20
bookFlatSection: true
title: "بررسی سانسور اینترنت در ارتباط DNS"
description: "انواع درخواست های DNS چیست و چطور میتوان سانسور و دستکاری صورت گرفته در آن را تشخیص داد."
tags: ["سانسور", "DNS", "DoH", "DoT", "دی ان اس", "Do53", "DNS over UDP", "DNS over TCP", "DNS over HTTPS", "DNS over TLS", "DNSCrypt", "بررسی DNS", "فیلتر", "فیلترنت", "تحریم", "اینترنت", "بررسی سانسور اینترنت", "سانسور اینترنت"]
images:
- "/images/docs/measure-internet-censorship/DNS/DNS-dig-twitter-at8.png"
---

# بررسی سانسور اینترنت در ارتباط DNS

اولین قدم برای دسترسی به اینترنت DNS است. وقتی شما آدرس نام دامنه را وارد می کنید، این نام باید به IP تبدیل شود تا دستگاه شما بتواند با سرور آن ارتباط برقرار کند. این کار توسط DNS انجام می شود. یکی از بهترین ابزار ها برای بررسی سانسور در DNS نرم افزار dig است. که علاوه بر [لینوکس و Mac OS، در ویندوز هم قابل استفاده است](https://www.isc.org/download/#BIND). در حال حاضر روش های مختلفی برای انجام تست DNS داریم. چند مورد پر استفاده و مهم از آنها عبارت اند از:
- DNS over UDP
- DNS over TCP
- DNS over HTTPS
- DNS over TLS


## DNS over UDP
DNS over UDP روش پیشفرض اکثر سیستم عامل ها است. این روش توسط شخص ناشناس قابل مشاهده و یا دستکاری است. اکثر سیستم های سانسور و بدافزار های شبکه، از این روش برای تغییر مسیر کاربران استفاده می کنند. در ایران معمولا جواب درخواست های DNS سایت های فیلتر شده به یکی از سه آدرس `10.10.34.34`، `10.10.34.35`، `10.10.34.36`، تغییر می کنند.

درخواست DNS از طریق مقدار پیشفرض DNS تنظیم شده در سیستم عامل:
<div dir="ltr">

`$ dig twitter.com`
<br/>

![dns dig twitter](/images/docs/measure-internet-censorship/DNS/DNS-dig-twitter.png)
</div>

درخواست DNS با تنظیم دستی سرور DNS:
<div dir="ltr">

`$ dig twitter.com @8.8.8.8`
<br/>

![dns dig twitter at 8.8.8.8](/images/docs/measure-internet-censorship/DNS/DNS-dig-twitter-at8.png)
</div>

نمایش این packet ها در نرم افزار wireshark:
<div dir="ltr">

![dns twitter wireshark packet](/images/docs/measure-internet-censorship/DNS/DNS-twitter-wireshark-packet.png)
</div>

انجام این درخواست با ابزار پیشفرض سیستم عامل ویندوز:
<div dir="ltr">

`> nslookup twitter.com`
<br/>

![dns nslookup twitter](/images/docs/measure-internet-censorship/DNS/DNS-nslookup-twitter.png)
</div>

و تنظیم آدرس سرور DNS اختصاصی:
<div dir="ltr">

`>nslookup twitter.com 1.1.1.1`
<br/>

![dns nslookup twitter at 1.1.1.1](/images/docs/measure-internet-censorship/DNS/DNS-nslookup-twitter-at1.png)
</div>


البته ممکن است همیشه چنین جوابی دریافت نکنید و درخواست های شما با جوابی همراه نباشند. در این شرایط درخواست ها وارد سیاه چاله می شوند.
<div dir="ltr">

`$ dig youtube.com`
<br/>

![dns dig youtube at 8.8.8.8 timeout](/images/docs/measure-internet-censorship/DNS/DNS-dig-youtube-at8-timeout.png)
</div>

در Wireshark هم می بینید که جوابی دریافت نشده:
<div dir="ltr">

![dns wireshark dig youtube at 8.8.8.8 timeout](/images/docs/measure-internet-censorship/DNS/DNS-wireshark-dig-youtube-at8-timeout.png)
</div>

و همینطور در سیستم عامل و سرور DNS متفاوت:
<div dir="ltr">

`> nslookup youtube.com 1.1.1.1`
<br/>

![dns nslookup youtube at 1.1.1.1 timeout](/images/docs/measure-internet-censorship/DNS/DNS-nslookup-youtube-at1-timeout.png)
</div>



نکته ی مهم این است که در ایران فرقی نمی کند که از چه سرویس ای درخواست می کنید. حتی فرقی نمی کند که آیا IP ای که درخواست می شود، واقعا سرویس DNS است یا خیر. و یا حتی فعال است یا خیر. به عنوان مثال، اگر از `1.1.1.1` استفاده می کنید و درخواستی دارید که فیلتر شده باشد، درخواست شما توسط سیستم سانسور ضبط می شود و به  `1.1.1.1` نمی رسد و جوابی مثل `10.10.34.34` برای شما ارسال می کند. 

برای امتحان، می توانید در سرور شخصی خود این دستور را اجرا کنید تا packet های ورودی مرتبط، به شما نشان داده شوند:
<div dir="ltr">

![tcpdump server port53](/images/docs/measure-internet-censorship/DNS/tcpdump-server-port53.png)
</div>

مقدار آخر برابر با IP ی دستگاه ایران شما است و  `eth0` نام interface اصلی شما که می توانید با اجرای دستور زیر آن را به دست بیاورید:
<div dir="ltr">

`$ ip link show`
</div>

در ادامه ی بحث اصلی، اگر درخواست را به صورت زیر وارد کنید:
<div dir="ltr">

![dig google at server](/images/docs/measure-internet-censorship/DNS/dig-google-at-server.png)
</div>

که مقدار آخر برابر با IP ی سرور خارج شما است. چنین packet هایی در سرور دریافت می کنید:
<div dir="ltr">

![tcpdump server port53 incoming dns requests](/images/docs/measure-internet-censorship/DNS/tcpdump-server-port53-incoming-dns-requests.png)
</div>

اما اگر یک سایتی که فیلتر شده را به جای آدرس `google.com` وارد کنید، packet ای در سمت سرور دریافت نمی شود و سیستم سانسور از سمت سرور ای که هیچ سرویس DNS در آن فعال نیست. IP ای از `10.10.34.3x` را بر می گرداند:
<div dir="ltr">

![wireshark hijacked dns dig twitter at server](/images/docs/measure-internet-censorship/DNS/wireshark-hijacked-dns-dig-twitter-at-server.png)
</div>

سیستم سانسور چین، معمولا یک IP ی random در جواب DNS سایت های سانسور شده برای کاربر ارسال می کند. از آنجایی که IP با آدرس درخواستی برابر نخواهد بود، نتیجتا باعث خطا می شود. یا سیستم سانسور بعضی از کشور ها `127.0.0.1` و یا `192.168.0.1` را در جواب سایت های سانسور شده ارسال می کنند. همچنین ممکن است بعضی از کشور ها از یک IP در دسترس اینترنت (IP public) برای سانسور استفاده کنند. مانند IP ای که ایران استفاده می کرده تا افرادی که سهوا و یا عمدا به سرویس های ممنوعه، همانند سایت قمار و شرطبندی، [تلگرام](https://persian.iranhumanrights.org/1398/01/rapid-response-center-of-the-office-of-the-prosecutors-cyberspace/)، [سرویس کوتاه کننده ی لینک](https://twitter.com/xhdix/status/1101586440025227264) و غیره رجوع می کردند، تحت پیگرد قانونی قرار دهد.

![prosecute the internet users for visiting a site in iran - pooyesh pardaz sorkh company](/images/docs/measure-internet-censorship/DNS/prosecute-for-visiting-site-iran-pooyesh-pardaz-sorkh.png)

## DNS over TCP
DNS over TCP هم از پورت مشابه DNS over UDP استفاده می کند و مشترکا Do53 معرفی می شوند. هر دوی اینها توسط شخص ناشناس قابل مشاهده است اما در ایران DNS over TCP  به سمت سرور های خارجی دستکاری نمی شود. توجه داشته باشید که اگر از یک DNS داخلی استفاده کنید، در اکثر آنها فرقی نمی کند که شما از چه پروتکل ای استفاده می کنید و جواب IP ی صفحه ی سانسور برای شما برگردانده می شود.

<div dir="ltr">

`$ dig youtube.com +tcp`
<br/>

![dns dig youtube tcp](/images/docs/measure-internet-censorship/DNS/DNS-dig-youtube-tcp.png)
</div>


استفاده از سرور خارجی:

<div dir="ltr">

`$ dig youtube.com +tcp @8.8.8.8`
<br/>

![dns dig youtube tcp at 8.8.8.8](/images/docs/measure-internet-censorship/DNS/DNS-dig-youtube-tcp-at8.png)
</div>

در ویندوز nslookup قابلیت DNS over TCP ندارد. خاطر همین ما از dig استفاده می کنیم:

<div dir="ltr">

`> .\dig youtube.com +tcp "@8.8.8.8"`
<br/>

![DNS windows dig youtube tcp at 8.8.8.8](/images/docs/measure-internet-censorship/DNS/DNS-windows-dig-youtube-tcp-at8.png)
</div>

گاهی مشاهده شده که بعضی درخواست هاتوسط سیستم سانسور وارد سیاهچاله می شوند.

<div dir="ltr">

```sh
$ dig +tcp youtube.com @1.1.1.1

;; Connection to 1.1.1.1#53(1.1.1.1) for youtube.com failed: timed out.
;; Connection to 1.1.1.1#53(1.1.1.1) for youtube.com failed: timed out.

; <<>> DiG 9.11.5-P4-5.1-Debian <<>> +tcp youtube.com @1.1.1.1
;; global options: +cmd
;; connection timed out; no servers could be reached
;; Connection to 1.1.1.1#53(1.1.1.1) for youtube.com failed: timed out.
```
</div>

## DNS over HTTPS
DNS over HTTPS یا به اختصار DoH، معروف ترین نوع DNS رمزنگاری شده است که اکنون استفاده می شود. پورت پیشفرض اش معمولا 443 است و در هر نرم افزاری که می تواند درخواست HTTPS انجام دهد قابل پیاده سازی است.

در شرایط عادی شخص ناشناس فقط می تواند IP و یا نام دامنه (در صورت وجود) همین سرویس را مشاهده کند. اینکه درخواست و یا جواب چیست از دید شخص ناشناس پوشیده است. توضیح اینکه در شرایط عادی، برای آدرس سرور DoH، از نام دامنه استفاده می شود. به عنوان مثال `https://dns.google/dns-query` . قبل از اینکه درخواست DNS اصلی شما از سرور درخواست شود، باید آدرس dns.google به IP تبدیل شود. برای این کار از DNS پیشفرض سیستم عامل استفاده می شود که معمولا DNS over UDP است. بعد از انجام TCP handshake و بعد TLS handshake، درخواست کاربر که به صورت HTTP است رمزنگاری و ارسال و جواب دریافت می شود.

در حال حاضر سیستم عامل های Windows، MacOS، iOS و همچنین مرورگر های بر پایه ی Chromium مانند Chrome، Edge، Opera، Brave و غیره و مرورگر Mozilla FireFox از DoH پشتیبانی می کنند. در کروم، روش پیشفرض به این صورت است که اگر DNS ای که کاربر تنظیم کرده است، قابل ارتقا به DoH را نیز داشته باشد، از آن استفاده می کند، مگر اینکه کاربر روش دیگری را انتخاب کرده باشد. در فایرفاکس در صورت فعال کردن DoH، روش پیشفرض به این صورت است که اگر درخواست DoH با شکست مواجه شد، از DNS پیشفرض سیستم استفاده کند. که یعنی اگر DNS پیشفرض سیستم عامل کاربر از نوع DNS over UDP باشد، در این شرایط شخص ناشناس می تواند درخواست و جواب را مشاهده و یا دستکاری کند.
امنیت ارتباط در این روش به اعتبار root CA سیستم عامل و یا برنامه ی مورد استفاده ی شما بستگی دارد. 

برای انجام تست، می توانید از curl برای DoH استفاده کنید و جواب json دریافت کنید. 

درخواست ساده از گوگل:

<div dir="ltr">

```sh
$ curl -s  "https://dns.google.com/resolve?name=www.who.int&type=A"

{"Status": 0,"TC": false,"RD": true,"RA": true,"AD": false,"CD": false,"Question":[ {"name": "www.who.int.","type": 1}],"Answer":[ {"name": "www.who.int.","type":
5,"TTL": 366,"data": "www.who.int.cdn.cloudflare.net."},{"name": "www.who.int.cdn.cloudflare.net.","type": 1,"TTL": 216,"data": "104.17.113.188"},{"name": "www.w
ho.int.cdn.cloudflare.net.","type": 1,"TTL": 216,"data": "104.17.112.188"}]}
```
</div>

از Cloudflare :

<div dir="ltr">

```sh
$ curl -H 'accept: application/dns-json' 'https://1.1.1.1/dns-query?name=www.who.int&type=A'  
              
{"Status":0,"TC":false,"RD":true,"RA":true,"AD":false,"CD":false,"Question":[{"name":"www.who.int","type":1}],"Answer":[{"name":"www.who.int","type":5,"TTL":839,"
data":"www.who.int.cdn.cloudflare.net."},{"name":"www.who.int.cdn.cloudflare.net","type":1,"TTL":239,"data":"104.17.112.188"},{"name":"www.who.int.cdn.cloudflare.
net","type":1,"TTL":239,"data":"104.17.113.188"}]}
```
</div>

( `-H` یک header به درخواست HTTP اضافه می کند.)

در بعضی نسخه های لینوکس می توان از دستور doh استفاده کرد:

<div dir="ltr">

`$ doh twitter.com`
<br>

![dns doh twitter](/images/docs/measure-internet-censorship/DNS/DNS-doh-twitter.png)
</div>

و یا اگر می خواهید در درخواست curl از DoH استفاده کنید:

<div dir="ltr">

```sh
$  curl --doh-url https://doh.powerdns.org/ https://www.google.com/.well-known/security.txt

Contact: https://g.co/vulnz
Contact: mailto:security@google.com
Encryption: https://services.google.com/corporate/publickey.txt
Acknowledgements: https://bughunter.withgoogle.com/
Policy: https://g.co/vrp
Hiring: https://g.co/SecurityPrivacyEngJobs
# Flag: BountyCon{075e1e5eef2bc8d49bfe4a27cd17f0bf4b2b85cf}
```
</div>

در بعضی ISP ها ممکن است با خطا مواجه شوید که به دلیل شیوه ای از سانسور اینترنت است که در بخش مربوط به بررسی سانسور در HTTPS توضیح داده شد.

## DNS over TLS
DNS over TLS یا به اختصار DoT، روشی رمزنگاری شده از DNS over TCP است. نوع ارتباط در ALPN از نوع dot است و طبق استاندارد، بر روی پورت 853 باید سرویس دهی شود. شناسایی و مسدود کردن این نوع توسط سیستم های سانسور بسیار راحت تر از DoH است. اما محتوای درخواست ها و جواب ها توسط شخص ناشناس قابل مشاهده نیست. این نوع نیز همانند DoH در صورت استفاده از نام دامنه، متکی به سرویس DNS over UDP پیشفرض است. 

در حال حاضر سیستم عامل های Android، Linux، iOS و MacOS از این نوع DNS پشتیبانی می کنند.

در آخرین نسخه از dig (نسخه ی تست شده:‌ `9.17.11` ) این امکان فراهم شد تا بتوان درخواست DoT داشت. به صورت استفاده از `+tls`:

<div dir="ltr">

`> .\dig telegram.org +tls "@dns9.quad9.net"`
<br/>

![dns windows dig dot tls at quad9](/images/docs/measure-internet-censorship/DNS/DNS-windows-dig-dot-tls-at9.png)
</div>

البته این قابلیت در dig هنوز به صورت کامل استاندارد ها را رعایت نکرده و به صورت رسمی در لحظه ی نوشتن این مقاله، این قابلیت رونمایی نشده.

توجه داشته باشید که DNS نقش حیاتی در امنیت ارتباطات شما دارد. اگر از DNS نامعتبر استفاده کنید و یا اینکه شخص ناشناس بتواند محتوای ارسالی و یا دریافتی آن را دستکاری کند، امنیت شما به خطر می افتد. به این دلیل همیشه سعی کنید از DNS های معتبر و با قابلیت رمزنگاری محتوا استفاده کنید. در ارتباطات رمزنگاری شده، یا شما مشکلی نخواهید داشت، و یا ارتباط با آن سرور تماما مسدود می شود.