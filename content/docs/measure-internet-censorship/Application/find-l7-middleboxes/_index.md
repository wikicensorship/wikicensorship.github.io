---
title: "یافتن سیستم سانسور کننده در لایه ی کاربرد"
description: "چگونه اثرانگشت سیستم سانسور را در ترافیک خود شناسایی کنیم و همینطور موقعیت سیستم سانسور کننده در لایه ی کاربرد را در مسیر شبکه مشخص کنیم."
tags: ["سانسور", "Clubhouse", "کلاب‌هاوس", "packet injection", "سیستم سانسور", "اینستاگرام", "middlebox fingerprint", "اثرانگشت سیستم سانسور", "سیستم سانسور چینی", "تجهیزات نظارت چین", "فیلتر", "Middlebox", "فیلترنت", "اینترنت", "TraceVis", "بررسی سانسور اینترنت", "سانسور اینترنت"]
date: 2022-06-29
images:
- "/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-clubhouse-client-hello-blocking-mci.png"
weight: 10
---

# یافتن سیستم سانسور کننده در لایه ی کاربرد


در بخش «[بررسی سانسور در ارتباط در لایه ی کاربرد](../../Application/_index.md)» در مورد روش های مقدماتی شناسایی سانسور در این لایه صحبت شد، اما همیشه سانسور به این سادگی قابل شناسایی نیست.
مخصوصا زمانی که خود پروتکل هایی مانند TLS، HTTP و یا SSH در IP مورد هدف قرار گرفته شده باشد، نه مقدار hostname یا چیزی دیگر درون این packet ها.

رفتار ها:

- خطای timeout به سمت یک IP ی مشخص بعد از ارسال TCP یا UDP ی دارای payload
- فقط drop شدن یکسری packet های خاص، مانند Server Hello و FIN و پاسخ با ACK
- قطع شدن ارتباط بعد از مدتی. مانند SSH بعد از حدود 15 تا 60 ثانیه


در این حالت، انجام trace route های سنتی به تنهایی، بی معنی است و به چیزی نخواهید رسید.
چرا که این نوع سانسورها بعد از TCP handshake و یا با تاخیر رخ می دهند.
برای بررسی درست، دو راه دارید:

- یافتن fingerprint سیستم سانسور در پاسخ ها
- انجام application traceroute

در ادامه این موارد را توضیح می دهیم و بعد در مورد شناسایی موقعیت سیستم های سانسور در مسیر ارتباط را بیشتر توضیح می دهیم.
مانند انسداد متناوب اینستاگرام و مسدود کردن کروم به سمت آمازون از طریق TLS fingerprint.
همچنین به سانسورهای امنیتی سایت های میزبانی شده در داخل و خارج، همانند کلاب‌هاوس اشاره می کنیم.


## یافتن fingerprint سیستم سانسور در پاسخ ها

تمام middlebox ها به دلیل اینکه سعی در جعل هویت سرور را دارند تا توسط برنامه ها در client مورد پذیرش قرار بگیرند، خصوصیاتی دارند که در حالی که حتی random به نظر می رسند، از الگوی مشخص پیروی میکنند که میتوان آنرا اثر انگشت سیستم سانسور کننده قلمداد کرد.
در این مطلب، به این رفتارها و الگوهای قابل سنجش fingerprint، می گوییم.

[در چین fingerprint سه دستگاه مختلف سانسور در مقاله ای مرتبط با DNS، ](https://github.com/net4people/bbs/issues/47) به ثبت رسیده است.
با کمک این مقاله و مقاله های مشابه، می توانیم در تمام کشورهایی که تجهیزات مدیریت، نظارت و سانسور حکومت چین در آنها نصب شده است، سانسورها در شبکه را از طریق یافتن چنین fingerprint هایی، شناسایی کنیم.

برای ساده سازی فقط یک مورد از fingerprint ها را در ادامه توضیح می دهیم و اگر نیاز بود، به مرور این بخش بروزرسانی خواهد شد.

### شناسایی از طریق IP.ID

برای شناسایی دستی از طریق `IP.ID`، نیاز به نصب [Wireshark](https://www.wireshark.org/
) دارید.
آموزش wireshark در اینترنت بسیار وجود دارد، در نتیجه در ادامه در نظر میگیریم که شما اطلاعات اولیه در مورد انتخاب شبکه و شروع capture کردن را دارید.
و به طریق زیر می توانید `IP.ID` را به لیست ستون های wireshark اضافه کنید تا راحت تر بتوانید کار شناسایی را انجام دهید:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/wireshark-set-ip-id-as-column.png)

</div>

در ابتدا،‌ طبق مقاله، DNS را بررسی می کنیم.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/dns-middlebox-fingerprint.png)

</div>

به دلیل مجاز بودن درخواست اول، در پاسخ ما IP ی درست دریافت کردیم و مقدار `IP.ID` نیز random است.
اما در درخواست دوم، یک سایت مسدود شده را تست کردیم و در پاسخ، IP های سیستم فیلترینگ جمهوری اسلامی را دریافت کردیم.
مقدار `IP.ID`  (ستون Identification) در اینجا تکرار `IP.ID` ی packet ای است که از کلاینت فرستاده است.
این یعنی، طبق مقاله، سیستم سانسور در بعضی موارد، مقادیر را فقط انعکاس می دهد.

حال، به جای DNS درخواست های HTTP را تست می کنیم.
جایی که به جای UDP از TCP استفاده می کنیم اما همچنان می توانیم محتوایی مربوط به سیستم سانسور جمهوری اسلامی را دریافت کنیم.
(در صورت ای که در شبکه [PEP](https://datatracker.ietf.org/doc/html/rfc3135) یا cache proxy پیاده سازی نشده باشد.)

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/http-example-dot-com.png)

</div>

در این مورد، ما امنیت بخش درخواست DNS را افزایش دادیم و در نتیجه، «شخص ناشناس» نمی تواند پاسخ DNS را دستکاری کند اما همچنان HTTP آسیب پذیر است.
پاسخ درست دریافت شده و مقدار `IP.ID` نیز منعکس نشده.

به همین صورت، یک آدرس مسدود شده تست می کنیم:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/http-middlebox-fingerprint-reddit-blocked.png)

</div>

اما در درخواست یک سایت مسدود شده، که در پاسخ دریافتی، محتوای مربوط به سیستم سانسور جمهوری اسلامی در آن قرار دارد، می بینیم که مقدار `IP.ID` در پاسخ و در RST، انعکاسی از frame قبلی از درخواست کاربر است.
و تمام packet های ارسالی بعدی، بدون پاسخ می مانند.
که این یعنی وارد سیاهچاله شده اند.

اگر درخواست را به این صورت تکرار کنیم، که از سرور `example.com` استفاده شود و اما مقدار Host برابر با reddit باشد، باز هم چنین رفتاری را مشاهده خواهیم کرد.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/http-middlebox-fingerprint-reddit-example-dot-com-ip-blocked.png)

</div>

با این موارد، می توان مطمئن شد که رفتار سیستم سانسور نسبت به تمام پروتکل ها یکسان است.


در HTTPS معمولا روش هایی مثل null routing و RST injection شناخته شده است.
اما روش های دیگری نیز وجود دارند.
در ادامه روشی را توضیح می دهیم که چند سالی است که توسط جمهوری اسلامی به سایت های زیادی اعمال می شود.

به عنوان مثال سایت `stackexchange.com` سالهاست که با این روش عموما ناشناخته مسدود شده است:

<div dir="ltr">

[![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/ooni-stackexchange-dot-com-blocked-2019-2020.png)](https://explorer.ooni.org/chart/mat?probe_cc=IR&test_name=web_connectivity&domain=stackexchange.com&since=2019-08-01&until=2020-05-01&axis_x=measurement_start_day)



[![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/ooni-stackexchange-dot-com-blocked-2021-2022.png)](https://explorer.ooni.org/chart/mat?probe_cc=IR&test_name=web_connectivity&domain=stackexchange.com&since=2021-06-27&until=2022-06-27&axis_x=measurement_start_day)

</div>

در این نوع سانسور، شما در DNS مشکلی نمی بینید:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/dns-stackexchange-dot-com.png)

</div>

خود آدرس هم مشکلی ندارد:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/https-stackexchange-dot-com-with-example-dot-com-ip.png)

</div>

(این تست با IP ی سایت `example.com` است اما از `stackexchange.com` در SNI استفاده می شود.
توضیحات بیشتر در مورد این تکنیک را در بخش «[بررسی سانسور در ارتباط با سرور در لایه ی کاربرد](../../Application/_index.md)» می توانید بخوانید.) 


اما درخواست مستقیم با خطا مواجه می شود:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/https-middlebox-fingerprint-stackexchange-dot-com.png)

</div>

در اینجا می بینیم که به ظاهر یک packet به با مشخصه ی `PSH,ACK` از سرور ارسال شده اما اگر به مقدار `IP.ID` دقت کنیم، همانند سانسورهای قبلی، انعکاسی از `IP.ID` ی packet ارسالی از سمت کلاینت است.
همچنین بعد از آن هیچ پاسخی دریافتی نمی شود.
در حقیقت وارد سیاهچاله می شود.

در نتیجه، با توجه به به مقاله ای که در ابتدا به آن اشاره شد و این توضیحات گام به گام، این packet از سمت سیستم سانسور جمهوری اسلامی ارسال شده است، نه سرور.
و این fingerprint ای از تجهیزات نظارت و سانسور چین است.

اگر برعکس دو تست قبلی، از IP ی `stackexchange.com` استفاده کنیم و مقدار SNI را `example.com` قرار دهیم، باز هم این خطا را مشاهده خواهیم کرد:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/https-middlebox-fingerprint-example-dot-com-with-stackexchange-dot-com-ip.png)

</div>

می توان نتیجه گرفت که در این نوع سانسور،‌ خود IP مورد هدف قرار می گیرد، و تعداد وبسایت ها و سرویس های بسیار بیشتری تحت تاثیر این نوع سانسور قرار خواهند گرفت.

اما در این نوع بخصوص از سانسور، سیاهچاله هدف اصلی نیست و سیستم سانسور جمهوری اسلامی در تلاش است تا کاربران را در استفاده از سرویس های خارجی، در مدت طولانی معطل کند:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/https-middlebox-fingerprints-ajax-aspnetcdn-dot-com.png)

</div>

در این تصویر با توجه به مقادیر `IP.ID` می بینید که سیستم سانسور packet های سمت کلاینت و سرور را در میانه ی یک stream/session مسدود یا drop می کند و خودش با کلاینت شروع به ارتباط می کند.


به دلیل اینکه:

- کلاینت در انتظار دریافت دیتا از سرور است؛
- معمولا [timeout خاصی در نرم افزار های کلاینت برای قطع ارتباط در زمان read bytes وجود ندارد](https://github.com/ooni/probe/issues/1609)، تا در شرایط سرعت پایین هم نرم افزار به کارکرد خودش ادامه دهد؛
- وجود پاسخ هایی جعلی ACK از سمت سیستم سانسور برای فعال نگه داشتن اتصال ای که هیچ داده ای رد و بدل نمی شود و جلوگیری از بسته شدنش اش توسط سیستم عاملِ کلاینت،

ارتباط تا مدت طولانی و نامشخصی ادامه پیدا خواهد کرد.

در بعضی موارد نیز، بعد از چند ثانیه یا دقیقه ارتباط مسدود می شود.
به عنوان مثال، در اینجا SSH بعد از 33 ثانیه مسدود و وارد سیاهچاله شد:


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/ssh-middlebox-fingerprint-start.png)



![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/ssh-middlebox-fingerprint-blocking-delay.png)

</div>


مقدار `IP.ID` در ردیف های 316 و 317 که از سمت سرور ارسال شده اند (source port: 22)، برابر با مقدار `IP.ID` ای است در ردیف 314 قرار دارد و از سمت کلاینت ارسال شده است.
(source port: 47696)
درنتیجه این دو packet از سمت سیستم سانسور ارسال شده اند، نه سرور.

### انجام application traceroute
همانطور که در ابتدا اشاره شد، شناسایی و یافتن منشا سانسورهایی که در لایه ی کاربرد انجام می شوند،‌ از طریق ابزار های trace route سنتی و ping ممکن نیست.
هرچند، داشتن داده هایی بیشتر از مسیر شبکه، بسیار کمک کننده است.

یک روش از شناسایی، در بخش «[یافتن دستگاه دستکاری کننده ی DNS](../../DNS/find-DNS-hijacker/_index.md)»‌ توضیح داده شد.
جایی که از یک packet درست و کامل DNS استفاده می شود با query های مختلف.

اما با توجه به وسعت زیادی پروتکل ها، رمزنگاری ها و تکنیک ها، ساختن هر packet از طریق یک نرم افزار، امکان پذیر نیست.
حتی در TLS به دلیل مسدود سازی ها از طریق [TLS fingerprint](https://engineering.salesforce.com/tls-fingerprinting-with-ja3-and-ja3s-247362855967)، ساخت یک packet ثابت راهکار خوبی برای شناسایی سانسور در لایه ی کاربرد نیست.



برای حل این موضوع،‌ در TraceVis این امکان اضافه شد تا کاربران بتوانند packet های مورد نظر خودشان را به این ابزار بدهند و در صورت نیاز، TCP handshake نیز انجام دهند.
و یا در صورت نیاز، تنها packet ای را که در حال retransmit است را با همان مشخصات اصلی trace route کنند.
مانند مورد SSH که بعد از 33 ثانیه ارتباط قطع شده بود.

در بخش های مختلف و به ترتیب، روش ها و نکات استفاده، توضیح داده خواهد شد: 

- نصب و اجرا
- کپی کردن یک packet از Wireshark
- استفاده از file های config
- انجام trace route بعد از مشاهده ی وارد سیاه چاله شدن
- تکنیک paris traceroute
- توضیحات بیشتر


شناسایی پاسخ middlebox از طریق fingerprint آنها به صورت اتوماتیک توسط TraceVis در زمان packet injection گسترده:


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-blocked-server-packet-injection.png)

</div>

(علامت ستاره به معنای پاسخ از سیستم سانسور است و دایره های خاکستری به معنای عدم پاسخ در آن hop)

در زمان رفع شدن سانسور:


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-unblocked-server-no-packet-injection.png)

</div>



#### نصب و اجرا
این ابزار در سیستم عامل های Linux, Windows, macOS تست شده.
برای اجرا،‌ نیاز به داشتن [Python](https://www.python.org/) نسخه ی 3 دارید.
و در ویندوز، نیاز به نصب npcap که با نصب [Wireshark](https://www.wireshark.org/)، به صورت اتوماتیک نصب خواهد شد.

در لینوکس، نسخه کرنل قبل از 5 نیاز به نصب iptables نیز دارید.

بعد از دانلود و یا clone کردن با git، قبل از استفاده باید از نیازمندی های برنامه، که scapy، pyvis و requests هستند را نصب کنید:

<div dir="ltr">

```bash
python3 -m pip install -r requirements.txt
```

</div>

یا با docker:

<div dir="ltr">

```bash
docker pull ghcr.io/wikicensorship/tracevis
```

</div>

برای تست عملکرد درست برنامه، میتوانید تست عادی DNS را اجرا کنید:


<div dir="ltr">

```bash
python3 ./tracevis.py --dns
```

</div>

یا با docker:

<div dir="ltr">

```bash
docker run ghcr.io/wikicensorship/tracevis --dns
```

</div>

اگر خطایی مشاهده نکردید و نتیجه درست بود، یعنی کار را به درستی انجام دادید و این ابزار آماده ی استفاده است.


#### دریافت یک packet از Wireshark و استفاده در TraceVis

در بخش «[یافتن دستگاه دستکاری کننده ی DNS](../../DNS/find-DNS-hijacker/_index.md)»‌ توضیح داده شد که چطور در tcpdump مقدار hexdump از packet ها بعد از لایه ی IP را مشاهده کنید و به TraceVis بدهید.
اما با Wireshark به دلیل گرافیکی بودن ابزار، دریافت packet درست، راحت تر است.

در آخرین نسخه از TraceVis این امکان فراهم شده است که هم packet کامل که از لایه ی Ethernet شروع شده است دریافت کند، و هم از لایه ی IP.
بنابراین، یک packet به صورت زیر می تواند دریافت شود:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/wireshark-copy-hexdump.png)

</div>


(هر دو مورد Hex+ASCII Dump و Hex Dump برای TraceVis یکسان هستند)

در TraceVis دستور را به صورت زیر اجرا می کنید:

<div dir="ltr">

```bash
python3 ./tracevis.py -p 
```

</div>

و مقدار کپی شده را paste می کنید:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-paste-packet.png)

</div>

بعد از یک Enter مشخصات packet برای شما نمایش داده خواهد شد:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-paste-packet.png)

</div>

اگر packet ای با flag های PSH,ACK را به TraceVis بدهید، مانند Client Hello و یا DNS over TCP، از شما سوال خواهد شد که آیا مایل به انجام TCP handshake قبل از ارسال این packet مورد نظر، هستید یا خیر.
(نکته: در DNS over TCP نیاز به TCP handshake نیست اما همچنان PSH,ACK است.
در نتیجه سوال می شود)


دلیل انجام این کار این است که اگر در شبکه، تجهیزاتی مانند [PEP](https://datatracker.ietf.org/doc/html/rfc3135) وجود داشته باشد، از عبور packet هایی مانند Client Hello بدون TCP session فعال، جلوگیری خواهد کرد و نتیجه ای اشتباه خواهید داد.
بنابراین، در TraceVis امکان انجام درست TCP handshake و بعد ارسال packet درخواستی کاربر با استفاده از TCP session به وجود آمده، فراهم شده است.
(در برنامه، ساز و کاری برای مشاهده ی hop های بعد از PEP هم در نظر گرفته شده است)

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-paste-packet.png)

</div>

اگر در لینوکس در حال تست هستید، به شما اطلاع داده می شود که نیاز دارید تا این rule برای iptables را اضافه کنید، تا از ارسال packet های RST توسط سیستم عامل کلاینت جلوگیری شود.
این کار فقط در kernel های **قبل** از 5 مورد نیاز است.
و می توانید از TraceVis بخواهید که به صورت اتوماتیک این کار را برای شما انجام دهد.
(اضافه و حذف بعد از پایان)
اگر در حال تست در سیستم عامل دیگری هستید، سوال بعدی پرسیده می شود برای اضافه کردن packet دوم:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-second-packet.png)

</div>

که دقیقا مراحل مثل قبل است.
و بعد از اتمام تست شروع می شود.


#### استفاده از file های config

بعد از انجام هر تست توسط TraceVis یک فایل config نیز برای اجرای دوباره ی همان دستور بدون نیاز به طی کردن مراحل قبلی ساخته می شود.
چند فایل نمونه هم در پوشه ی samples اضافه شده است.
به عنوان مثال اگر میخواهید تست NTP انجام دهید:

<div dir="ltr">

```bash
python3 ./tracevis.py --config ./samples/ntp.conf
```

</div>

و اگر قصد تغییر مقصد پیشفرض را دارید، کافی است تا آدرس را اضافه کنید:

<div dir="ltr">

```bash
python3 ./tracevis.py --config ./samples/ntp.conf -i “1.2.3.4”
```

</div>

توجه داشته باشید که امکان تغییر مواردی مانند SNI از طریق خط فرمان فعلا میسر نیست، در نتیجه، اگر قصد تست وبسایت جدیدی را دارید، باید یک packet جدید برایش بسازید.
چرا که، به عنوان مثال، فایل `clienthello.conf` در samples مقدار SNI اش آدرس `instagram.com` هست.
و همچنین ممکن استTLS fingerprint قدیمی Chrome باشد و سانسور بر مبنای TLS fingerprint وجود داشته باشد.


#### انجام trace route بعد از مشاهده ی وارد سیاهچاله شدن

در بخش قبلی در مورد SSH دیده بودید که بعد از 33 ثانیه، ارتباط وارد سیاهچاله شده بود، و در مواردی این حالت به صورت موقت وارد سیاهچاله می شد و دوباره ارتباط برقرار می شد.
در این شرایط امکان شناسایی منشاء این مشکل از طریق شیوه های قبلی وجود ندارد، چرا که فعلا امکان پیاده سازی هر پروتکل به صورت مجزا ممکن نیست.
برای شناسایی منشاء این مشکل در این حالت، فقط کافی است که یکی از packet هایی که در حال retransmission هستند را کپی کنیم:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/wireshark-copy-retransmission-packet.png)

</div>

و به صورت زیر به برنامه بدهیم:

<div dir="ltr">

```bash
python3 ./tracevis.py --rexmit
```
</div>

بعد از زدن کلید Enter، تست شروع می شود و سه بار تکرار می شود.
در این حالت، TraceVis مقادیر five-tuple را حفظ می کند و همگام با افزایش TTL از 1، مقدار `IP.ID` این packet را نیز یک مرتبه افزایش می دهد.
این حالت شبیه به یک retransmission واقعی است، اما در عین حال، می توانیم متوجه شویم که از چه بخشی از مسیر ارتباط null route می شود.

#### تکنیک paris

تکنیک Paris یا [Paris Traceroute](https://paris-traceroute.net/about/) به روشی که گفته می شود که در آن five-tuple هر packet در هر مرحله حفظ می شود تا هر بار در شبکه، از یک مسیر یکسان عبور کند.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/paris-traceroute-technique.png)

</div>

این روش، از دیدن loop اشتباه در شبکه جلوگیری می کند.



<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-standard-traceroute-problem.png)

</div>

که در صورت استفاده از تکنیک paris، به این صورت خواهد بود:


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-paris-traceroute.png)

</div>

در اینجا مشاهده می کنید که که در ابتدای مسیر، تعداد hop های مسیر سمت چپ کمتر از مسیر سمت راست است.
این حالت باعث شده بود که در حالت غیر paris،‌ تعداد زیادی حلقه مشاهده کنیم.


هرچند همانطور که در بخش «[بررسی سانسور در ارتباط با سرور در لایه‌ی شبکه و انتقال](../../Network/_index.md)» گفته شد، ممکن است حتی با تکنیک paris هم این حلقه ها را مشاهده کنید.
در این صورت، می تواند نشانی از walled garden باشد برای ایزوله کردن ارتباط داخل و خارج شبکه.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-loop-in-paris-technique.png)

</div>


نکته ی مهم:‌ اما اگر از این روش برای حالتی استفاده می کنید که از قبل نیاز به TCP handshake دارد،‌ ممکن است نتیجه ای که مشاهده می کنید با خطا مواجه شود.
به دلیل اینکه:
ابتدا با سرور اتصال برقرار می شود و TCP session فعال به وجود می آید
بعضی سرور ها در مدت کمی بعد از ایجاد TCP session، و در صورت دیافت نکردن packet، اقدام به بستن ارتباط با استفاده از packet های FIN یا RST می کنند.
که این خود نکاتی دارد:
سیستم سانسور نیز ممکن است اقدام به ارسال این packet ها کند که در بعضی شرایط میتوان از طریق fingerprint سیستم سانسور تفاوت را تشخیص داد.
اگر در شبکه PEP وجود داشته باشد و بعضی middlebox ها، بعد از مشاهده ی این packet ها از ادامه ی ارتباط جلوگیری می کنند.
و باعث اشتباه در نتیجه خواهند شد، در نتیجه نمیتوان وجود آنها را نادیده گرفت.
 
به دلیل اینکه در این حالت، مقدار five-tuple یکسان هستند، و همچنین اعمال سانسور، سیاهچاله یا packet injection، ممکن است با تاخیر همراه شود، نمیتوان با سرعت زیادی و یا به صورت همزمان با TTL های مختلف تست انجام داد.


هرچند در تلاش هستیم که هر روز بهینه سازی هایی انجام دهیم و راهکار های زیادی می توان در نظر گرفت.

یکی از موارد خوب در استفاده از این روش، در حالتی که سرور سریعا ارتباط را قطع نکند، تشخیص انسداد متناوب است.
در این حالت، چند اتصال مجاز به عبور و چند اتصال مسدود می شوند.
به طور مثال، آدرس مربوط به بخش دایرکت اینستاگرام را به صورت زیر تست می کنیم:

<div dir="ltr">

```bash
python3 ./tracevis.py -p -m 30 --paris -r 9
```
</div>

حداکثر تعداد 30 hop و تعداد تکرار 9 بار.
و packet مورد نظر داده می شود و درخواست TCP handshake هم می شود.
به همراه تکنیک paris، یعنی 9 مورد five-tuple مختلف.


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-paris-instagram-intermittent-blcoking-tci.png)

</div>

در این تصویر تست فقط جریان های 1،3،7،9 با موفقیت به مقصد رسیدند.
دیگر موارد در داخل قبل از خروج از کشور و با چقدر تلاش، از عبور آنها جلوگیری شد.

به صورت مشابه میتوان تست DNS نیز برای شناسایی موقعیت سیستم سانسور انجام داد:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-dns-example-dot-com-and-twitter-tci.png)

</div>

دقیقا در موقعیتی که پاسخی در جریان های مربوط به اینستاگرام دریافت نشد، 

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-instagram-blocking-zoom-middlebox-location.png)

</div>

به آدرس های سانسور شده در DNS واکنش نشان داده می شود:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-dns-zoom-middlebox-location.png)

</div>

( مکان اصلی این سیستم سانسور: بعد از 10.21.249.201)



#### توضیحات بیشتر


روش های و امکانات زیادی در TraceVis قرار داده شده است که میتوانید از آنها بهره ببرید.
مانند اضافه کردن پیشوند برای فایل های نتایج.
اضافه کردن یادداشت به هر نتیجه ی مستقل با یک packet.
و یا دیدن نتایج به صورت جدولی در یک فایل CSV.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-help.png)

</div>

هم اکنون این ابزار در حال استفاده و قابلیت استفاده ی گسترده را دارد.
اما فعلا بهتر است توسط افرادی که از شبکه اطلاع دارند و این مطلب را خوانده اند تحلیل و بررسی شود.

در تلاشیم تا قبل از نسخه ی اول، بهینه سازی های زیادی انجام دهیم و امکانات بیشتری اضافه کنیم.

و همچنین تشخیص روش های سانسور را برای کاربران و خودمان راحت تر کنیم.

به عنوان مثال در شرایطی که حتی در TCP handshake هم سیستم سانسور خود را جای سرور قرار می دهد.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/wireshark-cdn-comodo-middlebox-from-start-to-end.png)

</div>

در اینجا می بینید که حتی `IP.ID` در `SYN,ACK` هم انعکاسی از `IP.ID` ی `SYN` است.

سیستم سانسور در اینجا ارتباط را با `SYN,ACK` شروع کرده، با `PSH,ACK` پاسخ داده*، و با `FIN,ACK` ارتباط را پایان داده.


[*] در اینجا، `PSH,ACK` دارای هیچ دیتایی نیست، اما مواردی با مقدار های زیر هم مشاهده شده است:

<div dir="ltr">

```json
"\x30\x32\x30\x30\x30\x30"
"\x6e\x96\x3f\xa9\xf8\xdd" 
"\xfe\xb3\x01\xa0\xbf\xcb"
```

</div>

بعد از به اصطلاح رفع شدن مشکل، اثری از پاسخ ها از سمت سیستم سانسور نیست اما کیفیت ارتباط به شدت پایین است:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/wireshark-cdn-comodo-semi-fixed.png)

</div>





## شناسایی موقعیت سیستم های سانسور دیگر در یک مسیر یکسان

در تکنیک paris موردی از سانسور متناوب اینستاگرام توضیح داده شده است.
در ادامه دو مورد از سانسور دیگر توضیح می دهیم:

- سیستم سانسور TLS fingerprint
- سانسورهای امنیتی

### سیستم سانسور TLS fingerprint
در این نوع سانسور،‌ شما ممکن است با یک برنامه یا مرورگر به وبسایتی دسترسی داشته باشید اما با یک مرورگر دیگر این امکان وجود نداشته باشد.

به طور مثال، سایت آمازون با کروم:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/chrome-tls-fingerprint-blocking-aws-except-china.png)

</div>

(آمازونِ چین جزو لیست سانسور نبوده)

اما با فایرفاکس مشکلی وجود نداشته:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/firefox-tls-fingerprint-accessibility.png)

</div>


از طریق TraceVis یک تست اجرا می کنیم.

یک Client Hello از Chrome را که آدرس `example.com` در SNI وجود دارد استفاده می کنیم.
اما به مقصد یکی از IP های AWS که توسط سایت آمازون استفاده می شود:

<div dir="ltr">

```bash
python3 ./tracevis.py -p --annot1 "example.com CH chrome" --paris -i "13.226.135.75"
```

</div>

به همین صورت یک تست اما Client Hello از Firefox اجرا می کنیم.
و در انتها دو نتیجه را با هم ترکیب می کنیم.
به این صورت:

<div dir="ltr">

```bash
Python3 ./tracevis.py  -f “./path/to/file-firefox.json” -f “./path/to/file-chrome.json”
```

</div>

نتیجه به این صورت خواهد بود:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-firefox-chrome-tls-fingerpring-blocking-aws.png)

</div>

رنگ صورتی مربوط به chrome است و رنگ فیروزه ای مربوط به Firefox.


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-firefox-chrome-tls-fingerprint-blocking-zoom-middlebox-location.png)

</div>

همانطور که می بینید،‌تمام درخواست ها بعد از `10.202.6.90` وارد سیاهچاله شده اند.


به دلیل Hijack شدن تمام درخواست های DNS، (همانطور که در بخش «[یافتن دستگاه دستکاری کننده ی DNS](../../DNS/find-DNS-hijacker/_index.md)» توضیح داده شد)، اگر تست DNS به سمت همین IP انجام دهیم:

<div dir="ltr">

```bash
python ./tracevis.py --dns -i "13.226.135.75" -m 30 --paris
```

</div>

که در اینجا به صورت اتوماتیک دو درخواست DNS برای آدرس های ` example.com` که سانسور نیست، و `twitter.com` که سانسور است انجام می شود.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-dns-aws-example-dot-com-and-twitter.png)

</div>

رنگ صورتی آدرس `twitter.com` و رنگ فیروزه ای آدرس `example.com` است.

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-dns-aws-example-dot-com-and-twitter-zoom-middlebox-location.png)

</div>

تمام درخواست های مربوط به `twitter.com` بعد از `10.21.249.201` مسدود می شوند.
(برخلاف مورد قبلی که بعد از `10.202.6.90` بود)

سانسور در SNI نیز انجام می شود، چه تفاوتی ممکن است بین مسدود سازی از طریق TLS fingerprint و SNI وجود داشته باشد؟‌ برای پاسخ به این سوال، یک تست مربوط به SNI نیز به سمت همین مقصد مورد نظر، انجام می دهیم:

<div dir="ltr">

```bash
python ./tracevis.py -p --annot1 "twitter.com chrome" -i "13.226.135.75"
```

</div>

این بار از Client Hello ی کروم استفاده می کنیم اما به جاز آدرس مجاز `example.com` از آدرس `twitter.com` استفاده می  کنیم.


<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-sni-blocking-twitter-with-aws-ip.png)

</div>

با زوم بیشتر:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-sni-blocking-twitter-with-aws-ip-zoom.png)

</div>


همانطور که می بینید، برخلاف مورد اول، قبل از اینکه به `10.202.6.90` برسد درخواست های بعدی وارد سیاهچاله شده است.

این نشان می دهد که که حتی [ساختار شبکه هم شبیه به چین](https://geneva.cs.umd.edu/papers/foci21.pdf) ساخته شده است.


### سانسورهای امنیتی

از «سانسورهای امنیتی»، در اینجا برای توضیح مواردی استفاده می کنیم که توسط سیستم های سانسور معمول انجام نمی شوند و معمولا شیوه ی سانسور آنها نیز متفاوت است و ممکن است فقط در یک شبکه و حتی یک middlebox مشاهده شوند (در صورت میزبانی شدن آن سرویس در داخل کشور).
به عنوان مثال سانسور کلاب‌هاوس در همراه اول، در همان ابتدای شبکه انجام می شود:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-clubhouse-client-hello-blocking-mci.png)

</div>

و حتی به سیستم سانسور معمول شبکه و یا سیستم های سانسور بعدی در زیرساخت نمی رسد.
موقعیت های حدودی سیستم اصلی سانسور در همراه اول:

<div dir="ltr">

![imagename](/images/docs/measure-internet-censorship/Application/find-l7-middleboxes/tracevis-dns-main-middlbox-location-mci.png)

</div>

در «سانسورهای امنیتی»، شما فقط زمانی سانسور را مشاهده می کنید که مقدار SNI و IP ی درستی انتخاب کنید.
اگر در SNI از `example.com` استفاده کنید و یا از یک IP ی متفاوت، سانسور را مشاهده نخواهید کرد.
اما از طریق TraceVis می توانید متوجه شوید که در چه موقعیت ای از شبکه این سانسور انجام می شود.



