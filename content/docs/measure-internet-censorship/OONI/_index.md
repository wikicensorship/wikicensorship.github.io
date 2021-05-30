---
weight: 1
bookFlatSection: true
title: "بررسی سانسور اینترنت از طریق کاوشگر OONI"
description: "کاوشگر OONI چیست و چطور می توان از آن استفاده کرد. هر نتچیه چه معانی هایی دارند. آنالیز پیشرفته ی نتایج OONI"
tags: ["سانسور", "OONI", "OONI Probe", "کاوشگر OONI", "فیلتر", "فیلترنت", "تحریم", "اینترنت", "بررسی سانسور اینترنت", "سانسور اینترنت"]
images:
- "/images/docs/measure-internet-censorship/OONI/OONI-run-web-generate.png"
---

# بررسی سانسور اینترنت از طریق کاوشگر OONI
<center>

![ooni-logo](/images/docs/measure-internet-censorship/OONI/OONI-logo.png)
</center>
OONI و یا Open Observatory of Network Interference، ترجمه شده به فارسی: «رصد خانه آزاد دخالت در شبکه»، یک پروژه ی نرم افزاری متن باز و آزاد با  هدف توانمند سازی تلاش های غیرمتمرکز در افزایش شفافیت سانسور اینترنت در سراسر جهان است. OONI یک سَمَن یا سازمان غیردولتی است که توسط پروژه ی Tor راه اندازی شد و اکنون به صورت نیمه مستقل فعالیت می کند. 

این پروژه با ایجاد [نرم افزارهایی برای سیستم عامل های مختلف](https://ooni.org/install/) این امکان را فراهم می کند که یک شخص بدون نیاز به دانش فنی، فقط با اجرا کردن تست ها، شواهدی معتبر از سانسور تهیه کند.

<center>

![ooni mobile](/images/docs/measure-internet-censorship/OONI/OONI-mobile.png)
<br/>

![ooni desktop](/images/docs/measure-internet-censorship/OONI/OONI-desktop.png)
</center>

 به صورت پیشفرض این اطلاعات با حفظ حریم خصوصی کاربران، برای سرور OONI ارسال می شوند. اطلاعات دریافتی در سرور دوباره پردازش می شوند و در اکثر مواقع در کمتر از یک دقیقه در بخشی به نام Explorer تحت وب در دسترس همگان خواهد بود. 
<center>

![ooni explorer](/images/docs/measure-internet-censorship/OONI/OONI-explorer.png)
</center>

از سال 2012 تاکنون از بیش از 200 کشور نتایج اندازه گیری دریافت کرده.

## چگونگی کارکرد تست Websites
در Probe های OONI بخشی به نام Websites وجود دارد که در حال حاضر می تواند آدرس های http:// و https:// را مورد آزمایش قرار دهد. این تست که با اسم [Web connectivity](https://github.com/ooni/spec/blob/master/nettests/ts-017-web-connectivity.md) شناخته می شود به این صورت کار می کند که ابتدا یک بار آدرس درخواستی را از طریق backend خود مورد آزمایش قرار می دهد تا از فعال بودن آن اطمینان حاصل کند و همچنین جواب DNS و محتوای دریافتی در backend را با DNS و محتوای دریافتی در شبکه ی مورد تست مقایسه کند. به این معنی که اگر پاسخ تست ای، Anomaly و یا Confirmed بوده است، تست در backend و خارج از شبکه در همان لحظه با موفقیت انجام شده است. این مسئله صحت نتیجه در مورد وجود سانسور را تضمین می کند.

به صورت پیشفرض این تست از [لیست آدرس هایی استفاده می کند که توسط CitizenLab ساخته شده](https://github.com/citizenlab/test-lists/) و هر کسی میتواند [در بهبود آن مشارکت](https://ooni.org/get-involved/contribute-test-lists) داشته باشد. در هر کشور دو لیست مورد استفاده قرار میگیرد که یک [لیست اختصاصی آن کشور](https://github.com/citizenlab/test-lists/blob/master/lists/ir.csv) و یک [لیست جهانی](https://github.com/citizenlab/test-lists/blob/master/lists/global.csv) است. در زمان نوشتن این مقاله این دو لیست شامل 2242 آدرس است.
باید توجه داشته باشید که این لیست سایت هایی نیست که احتمال سانسور شدن آنها زیاد است. می توان گفت که این لیست ای از سرویس های پر استفاده است. اما نه لیست ای که توسط سرویس هایی مانند Alexa معرفی می شود، چون در بسیاری از کشور ها، مردم از ابزار های حفظ حریم خصوصی همانند VPN ها استفاده می کنند و یا در کشورهایی مثل ایران، اکثر سرویس های پراستفاده ی مردم، مسدود یا تحریم هستند و کاربر نمی تواند به صورت مستقیم به آنها دسترسی پیدا کند.
تست های پیشفرض در این بخش، به صورت پیشفرض فقط 90 ثانیه اجرا می شوند که از بخش تنظیمات این مقدار قابل تغییر است. مقدار 0 برابر با انجام اندازه گیری تمام آدرس های لیست پیشفرض است. 

البته این امکان وجود دارد تا آدرس ها و یا آدرس های سفارشی خودتان را نیز بدون نیاز به ثبت در لیست پیشفرض در شبکه ی خود و یا دوستانتان تست کنید. برای این کار دو روش وجود دارد:
- وارد کردن مستقیم لیست در اپ ها
- ایجاد لیست تست سفارشی


### وارد کردن مستقیم آدرس های سفارشی در نرم افزار
برای انجام این کار به صورت درون برنامه ای، وارد بخش Websites شوید و بر روی دکمه ی Choose websites و یا «انتخاب وبسایت ها» کلیک کنید :

<center>

![OONI mobile webconnectivity first page](/images/docs/measure-internet-censorship/OONI/OONI-mobile-webconnectivity-first-page.png)
<br/>

![OONI mobile webconnectivity first page](/images/docs/measure-internet-censorship/OONI/OONI-desktop-webconnectivity-first-page.png)
</center>
آدرس های مورد نظر خود را وارد کنید. و در انتها دکمه ی Run در پایین صفحه را بزنید.


<center>

![OONI mobile webconnectivity add urls page](/images/docs/measure-internet-censorship/OONI/OONI-mobile-webconnectivity-add-urls-page.png)
<br/>

![OONI desktop webconnectivity add urls page](/images/docs/measure-internet-censorship/OONI/OONI-desktop-webconnectivity-add-urls-page.png)
</center>

و در انتها بر روی دکمه ی Run و یا اجرا کلیک کنید تا اندازه گیری آغاز شود.

در نسخه ی cli، به این صورت می توان انجام داد:
اگر تابحال اجرا نکردید، کی بار مراحل و شرایط استفاده را بخوانید و در صورت موافقت تایید کنید:
> .\ooniprobe.exe
سپس لیست انتخابی خود را به این صورت اجرا کنید:
> .\ooniprobe.exe run websites --input=https://yahoo.com/ --input=https://yimg.com/

<center>

![OONI cli](/images/docs/measure-internet-censorship/OONI/OONI-cli.png)
</center>

در این حالت نتیجه ی هر تست و تفاوت ها به صورت لحظه ای نشان داده می شود.

### ایجاد لیست تست سفارشی
برای اینکه یک لیست سفارشی داشته باشید که بتوانید با دیگران به اشتراک بگذارید، می توانید به این آدرس رجوع کرده: 
https://run.ooni.io/

<center>

![OONI run web](/images/docs/measure-internet-censorship/OONI/OONI-run-web.png)
</center>

بعد از اضافه کردن آدرس های دلخواه از طریق دکمه ی Add URL دکمه ی Generate را بزنید.
<center>

![OONI run web generate](/images/docs/measure-internet-censorship/OONI/OONI-run-web-generate.png)
</center>

آدرس داده شده را کپی کرده و از طریق شبکه های اجتماعی برای دیگران بفرستید.

<center>

![OONI share social network](/images/docs/measure-internet-censorship/OONI/OONI-share-social-network.png)
</center>

کاربران در موبایل می توانند فقط با زدن بر روی لینک، اپ OONI Probe را انتخاب کرده:

<center>

![OONI open shared social network](/images/docs/measure-internet-censorship/OONI/OONI-open-shared-social-network.png)
</center>

و با زدن دکمه ی Run اندازه گیری را شروع کنند.

<center>

![OONI run mobile](/images/docs/measure-internet-censorship/OONI/OONI-run-mobile.png)
</center>


## آنالیز نتایج
آنالیز نتایج OONI بسیار ساده و آسان است. شما نیاز به داشتن دانش فنی برای درک وجود سانسور یا نبود آن ندارید. به عنوان مثال در این تصویر:

<center>

![OONI mobile websites test results](/images/docs/measure-internet-censorship/OONI/OONI-mobile-websites-test-results.png)
</center>

از 7 مورد آزمایش شده، دو مورد را با اطمینان می توان گفت که ارتباطشان به دلیلی در آن لحظه مسدود شده است و چهار مورد در دسترس بودند. 

همچنین یک مورد با رنگ خاکستری است که سه دلیل شایع می تواند داشته باشد: 
غیرفعال بودن سایت اصلی
قطع شدن لحظه ای ارتباط در زمان انجام آن تست
وجود باگ یا ناتوانی در تحلیل.

توضیحات آن نیز به این صورت است که علاوه بر مشاهده ی توضیحات بیشتر می توانید با زدن بر روی دکمه ی Try Again، دوباره اقدام به تست آن یک آدرس کنید:

<center>

![OONI mobile websites failed test again](/images/docs/measure-internet-censorship/OONI/OONI-mobile-websites-failed-test-again.png)
</center>

### آنالیز ساده ی نتایج
برای بررسی دقیقتر، در توضیحات هر نتیجه، دلیل آنها بیان شده است. چند مورد را که ممکن است بیشتر با آنها مواجه شوید در ادامه توضیح داده خواهد شد.

#### HTTP blocking (a blockpage might be served)
<center>

![OONI result HTTP blocking blockpage served](/images/docs/measure-internet-censorship/OONI/OONI-result-HTTP-blocking-blockpage.png)
</center>

در این تست احتمال می رود که ممکن است صفحه ی مربوط به سانسور به جای صفحه ی اصلی دریافت شده است. این اتفاق فقط در ارتباط های غیر رمزنگاری شده و مخصوصا HTTP ممکن است. در این حالت چهار دلیل ممکن است:
دریافت خطای 503، که خود نوعی سانسور است (در بخشی های بعدی توضیح داده خواهد شد)
حساسیت به آدرس سایت در بخش Host ارتباط HTTP
حساسیت به Request URI یا همان ادامه ی آدرس بعد از Hostname. (در این تست فقط / هست.)
حساسیت به یک کلمه در آن سایت. 

#### DNS tampering
<center>

![OONI result DNS tampering](/images/docs/measure-internet-censorship/OONI/OONI-result-DNS-tampering.png)
</center>

در این تست احتمال می رود که یک IP ی جعلی دریافت شده است. در این حالت، از DNS مسدود شده است.


#### TCP/IP based blocking
<center>

![OONI result tcp ip blocking](/images/docs/measure-internet-censorship/OONI/OONI-result-tcp-ip-blocking.png)
</center>

در این تست عمل TCP Handshake انجام نشده. که ممکن است به دلیل وارد سیاهچاله شدن ارتباط باشد و یا ارسال packet های RST. در این حالت IP مسدود شده است.



#### HTTP blocking (HTTP requests failed)
<center>

![OONI result HTTP blocking failed](/images/docs/measure-internet-censorship/OONI/OONI-result-HTTP-blocking-failed.png)
</center>

در این تست TCP Hnadshake انجام شده است (IP مسدود نیست) اما بعد از آن ناموفق بوده است. این خطا ممکن است در هر دو ارتباط HTTP و HTTPS مشاهده شود.




### آنالیز پیشرفته ی نتایج
برای آنالیز پیشرفته تر، در نرم افزار های OONI این امکان وجود دارد تا دیتای جمع آوری شده از هر تست را ببینید. 

<center>

![OONI android result menu](/images/docs/measure-internet-censorship/OONI/OONI-android-result-menu.png)
</center>

این دیتا از طریق آدرس اختصاصی Explorer نیز در دسترس است.

<center>

![OONI explorer result data](/images/docs/measure-internet-censorship/OONI/OONI-explorer-result-data.png)
</center>

در این دیتا، اطلاعات کلی و جزئی هر اندازه گیری به صورت مجزا وجود دارد.

توضیح بعضی از بخش  ها:


#### probe_ip
مقدار probe_ip که برابر با IP کاربر است. برای حفظ حریم خصوصی، به صورت پیشفرض در سمت کلاینت به 127.0.0.1 تغییر داده می شود.

#### resolver_ip
مقدار resolver_ip برابر IP سرور ای است که کاربر احتمالا در این تست از آن استفاده کرده است. این مقدار با [درخواست DNS مستقل](https://developer.akamai.com/blog/2018/05/10/introducing-new-whoami-tool-dns-resolver-information) به آدرس whoami.akamai.net محاسبه می شود. به صورت زیر:
<center>

![Akamai DNS resolver information whoami](/images/docs/measure-internet-censorship/OONI/Akamai-DNS-resolver-information-whoami.png)
</center>

#### test_keys
دیتای اصلی هر تست، در بخش test_keys قرار دارد:
<center>

![OONI result data test keys](/images/docs/measure-internet-censorship/OONI/OONI-result-data-test-keys.png)
</center>

#### network_events
بخش network_events شامل رویداد های ارتباطی مابین کلاینت و سرور است. ساده تر از چیزی که با capture کردن packet ها به دست می آوریم. برای اینکه ببینیم که مسدود شدن بعد از Client hello بوده است و یا بعد از TLS handshake.
<center>

![OONI result data network events](/images/docs/measure-internet-censorship/OONI/OONI-result-data-network-events.png)
</center>

#### tls_handshakes
بخش tls_handshakes شامل certificate هایی است که در هر بار request از سرور دریافت می شود.
<center>

![OONI result data tls handshakes](/images/docs/measure-internet-censorship/OONI/OONI-result-data-tls-handshakes.png)
</center>

در صورت حمله ی MITM این دیتا بسیار مفید خواهد بود.
 و در صورت خطا در زمان TLS handshake، توضیح آن بیان می شود:
<center>

![OONI result data tls handshakes error](/images/docs/measure-internet-censorship/OONI/OONI-result-data-tls-handshakes-error.png)
</center>


#### queries
بخش queries شامل درخواست های DNS و جواب های دریافت شده می شود:
<center>

![OONI result data queries](/images/docs/measure-internet-censorship/OONI/OONI-result-data-queries.png)
</center>

#### dns_consistency
بخش dns_consistency مربوط می شود به یکسان بودن و یا نبودن جواب DNS در تست انجام شده در سرور OONI و در این شبکه ی فعلی در همان لحظه.

#### control
بخش control مربوط می شود به تست های انجام شده در سرور OONI که همزمان صورت می گیرد.
<center>

![OONI result data control](/images/docs/measure-internet-censorship/OONI/OONI-result-data-control.png)
</center>


#### tcp_connect
بخش tcp_connect مربوط می شود به تست TCP handshake تمام IP هایی که از درخواست DNS به دست آمد.
<center>

![OONI result data tcp connect](/images/docs/measure-internet-censorship/OONI/OONI-result-data-tcp-connect.png)
</center>

#### requests
بخش requests مربوط می شود به درخواست ها و جواب هایی که در سطح HTTP انجام می شوند.
<center>

![OONI result data requests](/images/docs/measure-internet-censorship/OONI/OONI-result-data-requests.png)
</center>
در اینجا درخواست انجام نشده.
<center>

![OONI result data requests error](/images/docs/measure-internet-censorship/OONI/OONI-result-data-requests-error.png)
</center>

در اینجا جوابی با iframe از صفحه ی سیستم سانسور جمهوری اسلامی بازگردانده شده.

#### بیشتر
بخش های دیگر نیز نسبت به HTTP و یا HTTPS بودن ارتباط متفاوت است. 

در اینجا نشان می دهد که تبادل HTTP در HTTPS به درستی انجام نشده است:
<center>

![OONI result data match HTTPS reset](/images/docs/measure-internet-censorship/OONI/OONI-result-data-match-HTTPS-reset.png)
</center>
 
و یا مقادیر دو تست درون و بیرون شبکه در ارتباط HTTP یکسان نیست:
<center>

![OONI result data match HTTP different](/images/docs/measure-internet-censorship/OONI/OONI-result-data-match-HTTP-diff.png)
</center>

### تشخیص تحریم
یکی از سوال های مهم که در هر بار عدم دسترسی به سرویس ای یک ایرانی از خودش می پرسد این است که «آیا این سایت فیلتر شده و یا تحریم؟»
برای پاسخ به این سوال نیز می توان از OONI کمک گرفت. چرا که علاوه بر اینکه موارد زیادی را آنالیز می کند، همزمان هم در شبکه ی فعلی و هم از طریق سرور backend، اتصال به آدرس مورد نظر را بررسی و مقایسه می کند.
اگر آدرس وارد شده HTTPS باشد، به دلیل اینکه توسط شخص ناشناس قابل دستکاری نیست، نتیجه قابل اعتمادتر خواهد بود.
در ادامه یک آدرسی را که به دلیل استفاده از کلاد گوگل در ایران قابل استفاده نیست را به صورت نمونه بررسی می کنیم.

<center>

![OONI result data sanction ok](/images/docs/measure-internet-censorship/OONI/OONI-result-data-sanction-ok.png)
</center>

در نتیجه ی آنالیز می بینیم که به رنگ سبز است و این یعنی مشکلی در ارتباط وجود ندارد.

<center>

![OONI result data sanction failure null](/images/docs/measure-internet-censorship/OONI/OONI-result-data-sanction-failure-null.png)
</center>

در بخش Failure نیز می بینیم که تمام آنها null هستند و این یعنی هیچ دستکاری ای در شبکه انجام نشده است.

<center>

![OONI result data sanction control](/images/docs/measure-internet-censorship/OONI/OONI-result-data-sanction-control.png)
</center>

در بخش داده ها و در زیرمجموعه ی control می بنیم که مقدار عنوان درست است و همینطور کد دریافتی برابر با 200 است. که این یعنی سرور سایت به شبکه ای دیگر در خارج از کشور به درستی جواب می دهد.

<center>

![OONI result data sanction response](/images/docs/measure-internet-censorship/OONI/OONI-result-data-sanction-response.png)
</center>


اما در بخش requests کد دریافتی و همچنین response دارای مقدار متفاوتی است.

کد 403 و گاهی کد 404 اگر در این شرایط گفته شده (ارتباط HTTPS و متفاوت با نتیجه ی خارج از کشور) باشد، به معنای تحریم است.

همچنین در انتهای بخش test_keys مقایسه هایی صورت میگیرد که نتیجه ی کلی از تمام موارد گفته شده در بالا را نشان می دهد.

<center>

![OONI result data sanction match](/images/docs/measure-internet-censorship/OONI/OONI-result-data-sanction-match.png)
</center>

**ذکر این نکته مهم است که طی یک سال اخیر مورد ای از تحریم مشاهده نشده که بدون نمایش خطای 403 و یا 404 در ارتباط HTTPS از درخواست ها به دلیل تحریم جلوگیری کنند. در نتیجه اگر نتیجه غیر از سبز بود، احتمالا سایت به طریقی در شبکه ی شما سانسور شده است.**


## تنظیم داخلی فیلترشکن فقط برای ارتباط با backend
به دلیل سانسور های دسته ای سرویس ها در ایران و بدون در نظر گرفتن عواقب آن و زیر پا گذاشتن حق‌الناس توسط حکومتی که مدعی آن است، ارتباط با سرور OONI در موبایل، به دلیلی که در بخش «شناسایی مسدود شدن از طریق fingerprint» توضیح داده شد، امکان پذیر نیست.

<center>

![OONI probe service failed](/images/docs/measure-internet-censorship/OONI/OONI-probe-service-failed.png)
</center>

برای حل این مشکل، تیم OONI دو تغییر مهم ایجاد کردند:
تغییر TLS fingerprint برای تقلید fingerprint مرورگر
اضافه کردن بخش Proxy برای ارتباط با backend

برای فعال کردن proxy، به بخش تنظیمات رفته و OONI backend Proxy را انتخاب می کنیم:
<center>

![OONI android settings backend proxy](/images/docs/measure-internet-censorship/OONI/OONI-android-settings-backend-proxy.png)
</center>

در این بخش (در زمان نوشتن این مقاله) سه گزینه ی اصلی داریم:
از هیچ proxy ای استفاده نشود
از Psiphon استفاده شود
از یک Proxy شخصی استفاده شود. (در حال حاضر فقط SOCKS5)

<center>

![OONI android backend proxy settings](/images/docs/measure-internet-censorship/OONI/OONI-android-backend-proxy-settings.png)
</center>

تنظیماتی که در تصویر می بینید (127.0.0.1:9050) متعلق به SOCKS5 ای است که توسط Orbot ایجاد می شود.
<center>

![Orbot connected SOCKS5](/images/docs/measure-internet-censorship/OONI/Orbot-connected-SOCKS5.png)
</center>

توجه داشته باشید که برای استفاده از OONI، نباید حالت VPN فعال باشد.
اگر از Tor Browser استفاده می کنید، port آن برای SOCKS5 برابر با 9150 است.

بعد از فعال کردن یکی از این گزینه ها. ارتباط با backend توسط آن proxy صورت می گیرد اما تست ها از طریق خود شبکه:
<center>

![OONI android runnig test with Psiphon as proxy](/images/docs/measure-internet-censorship/OONI/OONI-android-Psiphon-as-proxy.png)
</center>

در تصویر می بینید که ابتدا به Psiphon متصل می شود.
<center>

![OONI android measurement result page](/images/docs/measure-internet-censorship/OONI/OONI-android-measurement-result-page.png)
</center>

اما تست ها از طریق شبکه ی کاربر انجام می شود.

