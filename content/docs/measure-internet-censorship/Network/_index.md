---
weight: 30
bookFlatSection: true
title: "بررسی سانسور در ارتباط با سرور در لایه‌ی شبکه"
description: "بررسی وجود سانسور و یا امکان دسترسی به سرور در لایه ی شبکه. معنای نتایج ping و traceroute و محدودیت های آنها در اندازه گیری سانسور."
tags: ["سانسور", "IP blocking", "RST injection", "null route", "فیلتر", "فیلترنت", "آی پی", "ICMP", "trace", "traceroute", "ping", "UDP", "اینترنت", "TCP", "شبکه", "Network", "بررسی سانسور اینترنت", "سانسور اینترنت"]
images:
- "/images/docs/measure-internet-censorship/Network/reverse-traceroute-to-firewall.png"
---

# بررسی سانسور در ارتباط با سرور در لایه‌ی شبکه

برای بررسی دسترسی به سرور معمولا از دو دستور ping و یا traceroute استفاده می شود. باید توجه داشته باشید که وقتی به صورت عادی از این دو دستور استفاده می کنید، شما قصد بررسی دسترسی IP دارید و اگر از آدرس دامنه برای آزمایش استفاده کنید، قبل از شروع درخواست، به صورت خودکار، یک بار از طریق DNS تنظیم شده در سیستم عامل شما، آدرس دامنه را به آدرس IP تبدیل می کند. در نتیجه اگر سانسور از طریق دستکاری DNS انجام شده باشد شما از طریق آدرس دامنه نمی توانید در دسترس بودن سرور از طریق IP را مورد بررسی قرار دهید. 

## دستور ping

برای اجرای دستور ping در لینوکس می توانیم به صورت زیر عمل کنیم:

<div dir="ltr">

`$ ping -c9 google.com`
<br/>

![linux ping google](/images/docs/measure-internet-censorship/Network/linux-ping-google.png)
</div>

دستور ping در همه ی سیستم عامل ها از پروتکل ICMP استفاده می کند. 

TTL به معنای Time to live است و مقدار آن نمایانگر حداکثر دفعاتی است که یک packet می تواند در شبکه پرش کند. به طور معمول، TTL ها با مقدار 64 در لینوکس و Mac OS، مقدار 128 در ویندوز و مقدار 255 در روتر ها ارسال می شوند. در اینجا مقدار 111 می تواند به این معنا باشد که سرور با شما به اندازه ی 17 تا hop فاصله دارد. به این صورت که در صورت دستکاری نشدن packet در شبکه، احتمالا سرور با مقدار 128 آن را ارسال کرده و اگر 128 را منهای 111 کنیم، 17 باقی می ماند. (توضیحات بیشتری در این رابطه می تواند وجود داشته باشد که در آینده مطرح خواهد شد.)

time به معنای زمان سپری شده در ارسال و دریافت جواب به میلی ثانیه است.

امکانات مفید برای این دستور:

- -v برای نمایش اطلاعات بیشتر
- -D برای نمایش زمان انجام آزمایش.
- -n برای resolve نکردن DNS در هر درخواست
- -c برای مشخص کردن حداکثر تعداد درخواست

(نکته: بدون این موارد هم آزمایش به درستی انجام خواهد شد)

اجرای دستور ping در ویندوز:
<div dir="ltr">

`> ping -n 9 google.com`
<br/>

![windows ping google](/images/docs/measure-internet-censorship/Network/windows-ping-google.png)
</div>

امکانات مفید برای این دستور در ویندوز:

- -t برای ping به صورت بی پایان
- -n برای مشخص کردن تعداد ping (پیشفرض 4 تا)

همانطور که در ابتدا گفته شد، انجام دستور به شیوه ی زیر برای دامنه هایی که از طریق دستکاری DNS سانسور شده اند، درست نیست:

<div dir="ltr">

`$ ping -c9 twitter.com`
<br/>

![ping wrong way](/images/docs/measure-internet-censorship/Network/ping-wrong-way.png)
</div>


به این دلیل که با این کار در حال ping به IP ی صفحه ی پیوندها خواهیم بود، نه IP ی توییتر!

برای انجام درست این درخواست، ابتدا باید IP ی اصلی را به شیوه ای که درخواست DNS ما سانسور نمی شود دریافت می کنیم:

<div dir="ltr">

```sh
$ dig twitter.com +short +tcp @8.8.8.8
104.244.42.1
104.244.42.129
```
</div>
بعد آن را به جای دامنه جایگذاری می کنیم:
<div dir="ltr">

`$ ping -c9 104.244.42.1`
<br/>

![ping twitter ip](/images/docs/measure-internet-censorship/Network/ping-twitter-ip.png)
</div>

### دستور ping با TCP

در مواردی ممکن است سرور و یا شبکه ای که سرور در آن قرار دارد تمام درخواست های ICMP را نادیده بگیرد و هیچ جوابی دریافت نکنید. برای اینکه درخواستی مشابه با ping داشته باشیم اما با پروتکل TCP، پروتکلی که برای ارتباط HTTP نسخه های قبل از 3 استفاده می شود، ابزارهایی وجود دارند مانند hping3 در لینوکس:

<div dir="ltr">

`$ sudo hping3 -S -p 443 -c 9 google.com`
<br/>

![linux ping tcp hping3 google](/images/docs/measure-internet-censorship/Network/linux-ping-tcp-hping3-google.png)
</div>

به معنای 

- -S انجام درخواست از نوع TCP با پرچم SYN

- -p تنظیم شماره ی Port

- -с تنظیم تعداد دفعات تست


در ویندوز نیز می توان از [tcping](https://www.elifulkerson.com/projects/tcping.php) استفاده کرد:

<div dir="ltr">

`> .\tcping.exe -p 443 104.20.225.46`
<br/>

![windows ping tcp tcping clubhouse api ip](/images/docs/measure-internet-censorship/Network/windows-ping-tcp-tcping-clubhouse-api-ip.png)
</div>


<div dir="ltr">

`> .\tcping.exe -p 443 104.20.225.45`
<br/>

![windows ping tcp tcping clubhouse api same ip range](/images/docs/measure-internet-censorship/Network/windows-ping-tcp-tcping-clubhouse-api-same-ip-range.png)
</div>




## دستور traceroute 

دستور traceroute در لینوکس (پکیج traceroute، نه دیگری) استفاده می شود و در ویندوز از tracert. هر دوی اینها به شما این امکان را می دهند تا مسیر رفت (آپلود) تا سرور (نه مسیر بازگشت یا دانلود که احتمالا کمی متفاوت است) و [hop](https://en.wikipedia.org/wiki/Hop_(networking)) های در میانه ی ارتباط را شناسایی کنید. و در صورت امکان، دلیل و یا مکان احتمالی اختلال و یا سانسور را متوجه شوید. اما دو تفاوت مهم بین این دو ابزار وجود دارد:

- در traceroute به صورت پیشفرض از پروتکل UDP استفاده می شود اما در tracert فقط از پروتکل ICMP.

- در traceroute امکان تغییر و سفارشی کردن پروتکل (ICMP, TCP و غیره) و پورت ها وجود دارد، اما در tracert نه.

همچنین نکات مهمی در این نوع بررسی وجود دارد که در انتها به آن پرداخت خواهد شد. اما چیزی که از قبل باید بدانید این است که زمان پاسخ، نمی تواند مطمئنا به معنای میزان تاخیر hop مورد نظر باشد. در اکثرمواقع routing ها به صورت سخت افزاری انجام می شوند و پردازش زیادی بر packet ها انجام نمی شود اما برای جواب دهی ممکن است میزان جواب های در صف و یا قدرت پردازش و همچنین اندازه ی جواب (شامل بودن و یا شامل نبودن تمام packet ارسالی) دخیل باشند.

برای اجرای دستور traceroute می توانیم به صورت زیر عمل کنیم:

<div dir="ltr">

`$ traceroute google.com`
<br/>

![linux tracerute google](/images/docs/measure-internet-censorship/Network/linux-tracerute-google.png)
</div>


در اینجا زمان پاسخ hop دوم حدود 30 میلی ثانیه است. و hop چهارم که در همان شبکه است با حدود 15 ثانیه تاخیر نسبت به پاسخ قبلی همراه بوده است. این می تواند ناشی از تاخیر در پاسخ دادن خود hop سوم باشد و یا مشکل در routing بین hop های دوم و سوم.

تا اولین پاسخ hop هفتم روند تاخیر افزایشی بوده و بعد حدود 25 میلی ثانیه کاهش در تاخیر مشاهده شده. این مسئله می تواند ناشی از کاهش بار شبکه باشد و یا تغییر routing در مسیری که بار کمتری بر روی آن است. باید توجه داشت که با توجه به تغییر سریع و غیر قابل پیشبینی مسیر ها در این شبکه، نمی توان مطمئن بود که جواب اول و دوم در hop هفتم، دقیقا از مسیر هایی رفت و یا برگشت ای عبور کردند که ما در hop های قبل از آن مشاهده کردیم.

در hop های هشتم و نهم، یکسان هستند. معمولا این حالت و موارد بازگشتی دیگر نشان از شبکه ی wall-gardened دارد. یعنی این نقطه، دیوار بین شبکه ی درون و بیرون است. جایی که نظارت و کنترل بر ورود و خروج محتوا صورت می گیرد. با توجه به ساختار شبکه ی ایران، احتمالا این فقط یکی از این دیوار ها و یا firewall های درون شبکه است.

در ادامه ی hop دوازدهم، ارتباط بیش از حد طولانی شده است و تمام پاسخ ها و یا درخواست ها سانسور شدند. این کار ممکن است توسط خود hop دوازدهم انجام شود و یا توسط hop سیزدهم. این روش احتمالا برای از بین بردن شواهد و مخفی کردن ادامه ی مسیر است. 

همینطور در انتها، در پاسخ سرور مورد بررسی ما، درخواست دوم با timeout مواجه شده است. (علامت ستاره * ) که ممکن است به دلیل سانسور موقتی و یا افزایش مسیر در آن لحظه باشد. به این صورت که همچنان با سی hop در آن لحظه packet ارسالی ما به سرور نرسیده و اما جواب ICMP همچون قبل از آن، در مسیر بازگشت سانسور شده است.

در ابتدا IP هایی که با 172 شروع می شوند، متعلق به شرکت شاتل هستند. بررسی با [Looking glass مرکز فیزیک نظری](http://lg.iranet.ir) :

<div dir="ltr">

![reverse traceroute to user-ip](/images/docs/measure-internet-censorship/Network/reverse-traceroute-to-user-ip.png)
</div>

و در ادامه مواردی مانند hop دوازدهم که با 10 شروع می شوند متعلق است به شرکت ارتباطات زیرساخت:

<div dir="ltr">

![reverse traceroute to firewall](/images/docs/measure-internet-censorship/Network/reverse-traceroute-to-firewall.png)
</div>

همانطور که گفته شد اگر سرویسی از طریق DNS مسدود است و ما DNScrypt/DoT/DoH ای نداریم، باید از IP برای تست استفاده کنیم. به عنوان مثال برای تست یکی از IP های توییتر:

<div dir="ltr">

`$ traceroute 104.244.42.1`
<br/>

![linux traceroute twitter ip](/images/docs/measure-internet-censorship/Network/linux-traceroute-twitter-ip.png)
</div>


نکته ی جالب در این دو traceroute که در شبکه ی شاتل انجام شده است این است که مسیر مربوط به دامنه ی گوگل بیشتر از توییتر است. دلیل این مسئله به احتمال زیاد bandwidth-throttling است. تکنیکی برای کاهش سرعت یک سرویس. اما اثبات این تکنیک، راحت نیست. چرا که از طرفی می بینیم که پاسخ از سرور توییتر زمان بیشتری نسبت به پاسخ سرور گوگل صرف کرده است.

البته باید توجه داشته باشید که هر ISP نسبت به هر پروتکل ممکن است رفتار متفاوتی نشان دهد. و یا این رفتار فقط در بعضی زمان ها اتفاق بیفتد و یا نسبت به تمام IP های یک سرویس چنین رفتاری نشان ندهد. تست دوباره:
<div dir="ltr">

`$ traceroute google.com`
<br/>

![linux repeat traceroute google](/images/docs/measure-internet-censorship/Network/linux-repeat-traceroute-google.png)
</div>


همانطور که در تصویر مشخص است، IP متفاوت، رفتار متفاوتی دارد. اما در انتها دو درخواست اولیه با timeout مواجه شدند که قبلا در مورد این مسئله توضیح داده شده است. (یا سانسور و یا تغییر route)

پیش تر گفته شده که پروتکل پیشفرض traceroute در سیستم عامل های Unix-like مانند لینوکس، macOS و غیره، UDP است. اما این امکان وجود دارد که با افزایش دسترسی، از پروتکل های دیگر نیز برای این کار استفاده کنید.

به عنوان مثال پروتکل ICMP :

<div dir="ltr">

`$ sudo traceroute google.com --icmp`
<br/>

![linux tracerute google icmp](/images/docs/measure-internet-censorship/Network/linux-tracerute-google-icmp.png)
</div>


و همانطور که گفته شد، در ویندوز با tracert فقط می توانید با پروتکل ICMP این کار را انجام دهید:
<div dir="ltr">

`> tracert google.com`
<br/>

![windows tracert google](/images/docs/measure-internet-censorship/Network/windows-tracert-google.png)
</div>


اما چیزی که بیش از هر چیز ممکن است نیازتان شود، پروتکل TCP است:
<div dir="ltr">

`$ sudo traceroute google.com --tcp`
<br/>

![linux traceroute tcp](/images/docs/measure-internet-censorship/Network/linux-traceroute-tcp.png)
</div>

در اینجا می بینیم که سومین hop به گوگل رسیدیم. یعنی مستقیم بعد از ISP ! آن هم در کمتر از 40 میلی ثانیه که با توجه به آزمایش قبلی (و بعدی) که حدود 150 میلی ثانیه بود و با اختلالات ساختاری و خلاقیت های شبکه ی داخلی ایران، این سرعت کاملا دور از واقعیت است. دلیل این اتفاق چیز دیگریست که در بخش بعدی مربوط به ارتباط HTTP توضیح داده خواهد شد.

به دلیل اینکه port پیشفرض برای TCP مقدار 80 است، در مورد بعدی، علاوه بر تغییر port به 443، برای ساده سازی نمایش، تعداد هر بار query در هر TTL را هم به 1 تغییر دادیم و همینطور در بخش option تنظیم کردیم که اطلاعات بیشتری از جواب های از نوع TCP برای برای ما نشان دهد:

<div dir="ltr">

`$ sudo traceroute google.com --tcp --queries=1 -O info --port=443`
<br/>

![linux traceroute google tcp port 443 1 query and show info](/images/docs/measure-internet-censorship/Network/linux-traceroute-tcp-443-info-google.png)
</div>


در اینجا می بینید که مسیر در پورت 443 متفاوت است و بعد از hop دوازدهم، تمام پاسخ های ICMP سانسور شدند. درخواست ارسالی TCP است و با TTL به مقدار 20 به سرور رسیده و جواب SYN,ACK دریافت شده. اما hop های میانه که با TTL صفر مواجه می شوند، جواب ICMP بر می گردانند. در نتیجه، در اینجا، این جواب های ICMP هستند که سانسور می شوند.

تا سال گذشته در بیشتر مواقع در جواب درخواست به IP های مسدود شده، به جای پاسخ هایی با پرچم SYN,ACK، پاسخ با پرچم RST,ACK دریافت می شده است. این packet دریافتی، گاهی دستکاری جواب SYN,ACK سرور بود و گاهی مستقیما از سیستم سانسور ارسال می شد. که برای اطمینان در این موارد باید ترافیک کل stream آنالیز شود. اما RST همیشه به معنی سانسور نیست. گاهی سرور در اثر باز نداشتن آن port و یا تنظیم اشتباه، این پیام را بر می گرداند. به عنوان مثال از یک ISP ای دیگر در ایران به سمت سایت همراه اول traceroute انجام می دهیم:

<div dir="ltr">

`$ sudo traceroute --tcp --port=443 -O info -m 50 mci.ir`
<br/>

![linux traceroute mci tcp 443](/images/docs/measure-internet-censorship/Network/linux-traceroute-mci-tcp-443.png)
</div>

در اینجا می بینیم که سایت همراه اول نسبت به درخواست ها از غیر از شبکه ی خودش حساس است و از آن امتناع می کند. همچنین دو درخواست اولیه همراه با SYN,ACK نبودند. این ممکن است که جواب دریافتی به جای TCP از نوع ICMP بوده باشد و این بدان معنی است که احتمالا رفتاری از firewall باشد.

 این تست را چند بار تکرار می کنیم:
<div dir="ltr">

<br/>

![linux repeat traceroute mci tcp 443](/images/docs/measure-internet-censorship/Network/linux-repeat-traceroute-mci-tcp-443.png)
</div>

در تست دوباره، فقط یک بار در بار دوم SYN,ACK به درستی دریافت شد و دو مورد دیگر RST,ACK بودند.

اما در سیستم سانسور، اخیرا برای اتلاف بیشتر وقت و عمر کاربران، از ارسال هر جوابی به کاربر اجتناب می شود. به عنوان مثال، IP ی مربوط به آدرس t.me را بررسی می کنیم:

<div dir="ltr">


`$ sudo traceroute 149.154.167.99 --tcp --port=443 -O info`
<br/>

![traceroute tcp telegram ip blackhole](/images/docs/measure-internet-censorship/Network/traceroute-tcp-telegram-ip-blackhole.png)
</div>

طبق trace های قبلی، تا hop هفتم همچنان در بستر ISP قرار داریم. این یعنی قبل از ورود به بستر شرکت ارتباطات زیرساخت، از ادامه ی ارتباط جلوگیری شده است و به عبارتی دیگر این مسدود سازی در سطح ISP انجام شده.

### سانسور مسیر ورودی

همانطور که در ابتدا گفته شد، مسیر رفت با مسیر بازگشت ممکن است متفاوت باشد و همیطور ممکن است سانسور فقط در یکی از این دو صورت بگیرد.

در مواردی مانند سانسور Clubhouse شاهد این بودیم که در بعضی شبکه ها، بعد از خروج از کشور دیگر جوابی دریافت نمی شود:

<div dir="ltr">

`$ sudo traceroute 104.20.225.46 --tcp --port=443`
<br/>

![linux traceroute tcp 443 clubhouse api ip tci](/images/docs/measure-internet-censorship/Network/linux-traceroute-tcp-443-clubhouse-api-ip-tci.png)
</div>


اما در همان IP range در hop دهم به جواب می رسیدیم:

<div dir="ltr">

`$ sudo traceroute 104.20.225.45 --tcp --port=443`
<br/>

![linux traceroute tcp 443 clubhouse api same ip range tci](/images/docs/measure-internet-censorship/Network/linux-traceroute-tcp-443-clubhouse-api-same-ip-range-tci.png)
</div>


و یا در شبکه ای دیگر:
<div dir="ltr">

`$ sudo traceroute 104.20.225.46 --tcp --port=443`
<br/>

![linux traceroute tcp 443 clubhouse api ip shatel](/images/docs/measure-internet-censorship/Network/linux-traceroute-tcp-443-clubhouse-api-ip-shatel.png)
</div>


این مسئله فقط می تواند به این دلیل باشد که تمام packet های ارسالی از سرور مورد تست ما مسدود می شوند. برعکس مثال های قبل که packet ارسالی از کلاینت مسدود می شدند. 

در traceroute همچنین این امکان وجود دارد تا مسیر برگشتی محاسبه شود. به این صورت که TTL های ارسالی از hop ها به صورت سه مورد اصلی 255 (روتر)، 128 (ویندوز) و 64 (لینوکس/یونیکس) در نظر گرفته می شوند و TTL جواب، منهای یکی این سه که محتمل تر هستند می شود. 

<div dir="ltr">

`$ sudo traceroute google.com --tcp --port=443 -O info --back`
<br/>

![traceroute google tcp 443 with back ttl](/images/docs/measure-internet-censorship/Network/traceroute-google-tcp-443-with-back-ttl.png)
</div>

به عنوان مثال، می بینیم که در hop هشتم مسیر برگشت یک واحد طولانی تر است و hop دهم نیز همانند hop هشتم است. (firewall یا سیستم سانسور ایران)


### سانسور پورت خاص

گاهی سانسور بر مبنای port است. طبق این توییت بعضی پورت ها یا پروتکل ها محدود شدند.


<!--  Embed tweet with: Opt-out of tailoring Twitter  -->
<blockquote class="twitter-tweet" data-lang="en" data-dnt="true" data-theme="dark"><p lang="fa" dir="rtl">قابل توجه :<br>&quot; <a href="https://twitter.com/hashtag/%D8%B4%D8%A7%D8%AA%D9%84?src=hash&amp;ref_src=twsrc%5Etfw">#شاتل</a> به صلاح دید خود، در راستای حفظ امنیت شبکه، می‌تواند بدون اطلاع رسانی دسترسیهای مشترک به پورت یا پروتکلها را محدود کند. در حال حاضر موارد محدود شده به شرح جدول زیر است. &quot;<br>وقتی میگیم تو این مملکت اینترنت نیست از همین جاها شروع میشه.<a href="https://twitter.com/hashtag/adsl?src=hash&amp;ref_src=twsrc%5Etfw">#adsl</a> <a href="https://t.co/ddTY26326H">pic.twitter.com/ddTY26326H</a></p>&mdash; Matin (@matinrco) <a href="https://twitter.com/matinrco/status/1240246443895652352?ref_src=twsrc%5Etfw">March 18, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



برای آزمایش، ما پورت مربوط به SMB ویندوز که برابر با 445 است را در TCP بررسی می کنیم:

<div dir="ltr">

`sudo traceroute google.com --tcp --port=445`
<br/>

![traceroute google tcp 445 shatel](/images/docs/measure-internet-censorship/Network/traceroute-google-tcp-445-shatel.png)
</div>


علامت !X در traceroute به معنی "communication administratively prohibited" است. یعنی این ارتباط، هرچه که هست، در hop چهارم ممنوع اعلام شده. (برخلاف !Z که فقط مقصد مورد توجه است)
حالا ارتباط UDP در Port 445 که در تصویر نیست:

<div dir="ltr">

`traceroute google.com --udp --port=445`
<br/>

![traceroute google udp 445 shatel](/images/docs/measure-internet-censorship/Network/traceroute-google-udp-445-shatel.png)
</div>

که باز هم نشان از این دارد که ISP ارتباط UDP به این سرور را هم مسدود کرده است.

*(SMB آسیب پذیری های زیادی داشته. اما بستن هر چیز به جای آموزش درست، راه حل نیست، حذف کردن صورت مسئله است. اگر واقعا برای جلوگیری از حمله باشد، این کار تاثیر چندانی ندارد. این حمله بدون استفاده از ISP ی شاتل هم می تواند در شبکه ی داخلی سازمان و یا کاربر اتفاق بیفتد.)*

### در چه صورت traceroute می تواند سندی از کاهش سرعت باشد؟

یک موضوع که ممکن است بتواند برای بررسی این مسئله مورد استفاده قرار بگیرد که کدام hop در حال ایجاد اختلال در ارتباط است، بررسی زمان پاسخ های آنها است. اما فقط به شرطی که hop های بعدی نیز حداقل به میزان hop قبلی تاخیر داشته باشند. این موضوع در مورد packet loss هم صدق می کند.

 به این نتیجه از برنامه ی MTR دقت کنید:

<div dir="ltr">

`$ mtr instagram.com`
<br/>

![mtr instagram shatel](/images/docs/measure-internet-censorship/Network/mtr-instagram-shatel.png)
</div>

با توجه به مقدار packet loss، در hop دهم هیچ جوابی دریافت نشد. یعنی 100٪ packet loss، اما این به معنی مسئول بودن در کاهش سرعت **نیست**. چرا که در hop بعدی میزان کمتر است و به 22.2٪ و در hop بعد از آن به 0٪ رسیده است. همین مسئله در مورد hop های بعدی مثل hop سیزدهم صدق می کند.

با توجه به میزان تاخیر در پاسخ، بجز تاخیر تا ارتباط با ISP در hop دوم، تاخیر جدی اولیه با میزان تاخیر بیش از ده ثانیه، که در امتداد مسیر ادامه دارد، hop سوم است. همچنین دراین hop می بینیم که StDev یا [انحراف معیار](https://en.wikipedia.org/wiki/Standard_deviation) به مقدار 5 است. این به معنی نوسان زیاد در پاسخ این دستگاه است.

با توجه به زمان تاخیر، احتمالا تاخیر جدی دوم، hop دهم است. درhop یازدهم این تاخیر نمایان می شود که حدود 70 میلی ثانیه تاخیر وجود دارد. همچنین یک تاخیر قابل توجه سوم ای به میزان 100 میلی ثانیه هم وجود که در hop دوازدهم و یا در بین hop های یازدهم و دوازدهم اتفاق افتاده است.

از hop دوازدهم به بعد، میزان تاخیر بسیار کم است اما همچنان میزان انحراف معیار بالا است.

این نکته باید تاکید شود که این اندازه گیری ها باید چند بار تکرار شوند تا از پایداری این شیوه های اختلال اطمینان حاصل شود. به دلیل اینکه همانطور که پیش تر گفته شد، ممکن است مسیر ارتباط بارها تغییر کند و در آن لحظه ی مشاهده ی تاخیر، packet های ارسالی از مسیری عبور نکند که قبل از آن لیست شده اند. ([در مورد شرایط احتمالی در mtr بیشتر بخوانید. ](https://www.linode.com/docs/guides/diagnosing-network-issues-with-mtr/))

در ویندوز ابزاری مشابه با کمی تفاوت وجود دارد به اسم pingpath :

<div dir="ltr">

`> pathping 34.234.142.160`
<br/>

![pingpath instagram ip tci](/images/docs/measure-internet-censorship/Network/pingpath-instagram-ip-tci.png)
</div>


این ابزار این امکان را فراهم می کند که تفاوت بین عدم پاسخ hop و ایجاد اختلال در ارتباط مشخص شود. البته این مسئله باز هم ممکن است تحت تاثیر موارد دیگری قرار بگیرد و نمی تواند قطعیت داشته باشد.  به عنوان مثال، این ابزار فقط تا hop ای اندازه گیری خواهد که که با timeout برخورد نکند. همچنین تغییر routing سریعی که در برخی نقاط در شبکه ی ایران اتفاق می افتد، سلامت داده ی این نوع اندازه گیری ها را زیر سوال می برد.

<div dir="ltr">

![traceroute intermittent route changing](/images/docs/measure-internet-censorship/Network/traceroute-intermittent-route-changing.png)
</div>
