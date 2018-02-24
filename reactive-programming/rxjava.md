# RxJava

[RxJava](https://github.com/ReactiveX/RxJava) is implementation of [ReactiveX](http://reactivex.io). ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.

Callbacks are too low level and `CompletableFuture` deals only with single value, there comes streaming event driven and declarative API of RxJava.

### Observable

Observable is generalisation of Observer pattern. Observable adds stream-like API, API with common operations and back pressure or throttling.

Java streams should be used when we know what data are going to be processed, something like batch processing. While Observable is more for situations when we do not know what is coming, something like on demand computations.

```
import io.reactivex.Observable;
import org.reactivestreams.Subscriber;
import org.reactivestreams.Subscription;

class ObservableHelloWorld {

    public static void main(String[] args) {
        Observable<String> observable = Observable
            .create(emitter -> emitter.onNext("Hello World"));

        observable.subscribe(System.out::println);
    }
}
```

We create observable and emit a "Hello World" message. Then we subscribe to that observable and print the message out. 

```
Hello World
```

### Flowables

`Flowable` has similar API as `Observable`. `Observable` does not provide reactive pull back pressure. `Flowable` is the new `Observable` and `Flowable` should be used instead.

#### Interval

We can define interval that will execute consequent calls on flowable. In this case, we create interval that emits random long numbers every two seconds. 

```
import io.reactivex.Flowable;

class FlowableInterval {

    public static void main(String[] args) {
        Flowable<Long> flowable = Flowable
            .interval(2, TimeUnit.SECONDS)
            .map(x -> new Random().nextLong());

        flowable.blockingSubscribe(System.out::println);
    }
}
```

When we run that code, we get a random long every two seconds. Just imagine what threads would you have to create you have to create without Flowable.

```
9031197039531145666
2089494502320359488
-3142894608516344986
-371557809365757488
```

#### Zip

Zip helps to do operations on two flowables.

```
class FlowableZip {

    public static void main(String[] args) {
        Flowable<Integer> f1 = Flowable.fromArray(1, 2, 3, 4, 5);
        Flowable<Integer> f2 = Flowable.fromArray(1, 2, 3);

        Flowable<Integer> sum = f1.zipWith(f2, (x, y) -> x + y);
        sum.blockingSubscribe(System.out::println);
    }
}
```

When we run zipWith on two flowables, it creates inner join and maps the values using the function provided as second parameter. 

```
2
4
6
```

The first example of zipWith is really easy. Lets try to do something more interesting. Lets create two streams of numbers and then zip them together. Each stream will generate a number after two seconds. 

```
class FlowableZipStreams {

    public static void main(String[] args) {
        Flowable<Integer> f1 = Flowable
            .interval(1, TimeUnit.SECONDS)
            .map(x -> new Random().nextInt(10));
        Flowable<Integer> f2 = Flowable
            .interval(1, TimeUnit.SECONDS)
            .map(x -> new Random().nextInt(10));

        Flowable<Integer> sum = f1.zipWith(f2, (x, y) -> x + y);
        sum.blockingSubscribe(System.out::println);
    }
}
```

When we run this code, each stream generates number from 0 to 10. Then the zipWith method will make sum of those two numbers. 

```
10
11
10
15
7
10
```

#### Filtering Values

We can use filter method to filter values coming from flowable. We are going to filter values that are lower than 3 in the following example. 

```
class FlowableFilter {

    public static void main(String[] args) {
        Flowable
            .fromArray(1, 1, 2, 3, 2, 3, 4, 5)
            .filter(it -> it.compareTo(3) < 0)
            .distinct()
            .blockingSubscribe(System.out::println);
    }
}
```

Here is the output when we run the code. 

```
1
2
```

#### Publish & Subscribe

We can create well defined publishers and subscribers using RxJava. In this example, we are going to publish two messages and announce completion of production. 

```
class FlowablePubSub {
    public static void main(String[] args) {
        Flowable<String> stream = Flowable.fromPublisher(publisher -> {
            publisher.onSubscribe(new Subscription() {
                @Override
                public void request(long n) {

                }

                @Override
                public void cancel() {

                }
            });
            publisher.onNext("Starting publishing...");
            publisher.onNext("Another message published");
            publisher.onComplete();
//            publisher.onError(new RuntimeException("An exception"));
        });

        stream.subscribe(new Subscriber<>() {
            @Override
            public void onSubscribe(Subscription s) {
                System.out.println("[onSubscribe]");
            }

            @Override
            public void onNext(String s) {
                System.out.println("[onNext] " + s);
            }

            @Override
            public void onError(Throwable t) {
                System.out.println("[onError] " + t.getMessage());
            }

            @Override
            public void onComplete() {
                System.out.println("[onComplete]");
            }
        });

        stream.subscribe(s -> System.out.println("[accept] " + s));
    }
}
```

Here is the output. First subscriber will receive messages. Then another subscriber, who has only one method prints received messages. 

```
[onSubscribe]
[onNext] Starting publishing...
[onNext] Another message published
[onComplete]
[accept] Starting publishing...
[accept] Another message published
```

#### Testing

When it comes to testing, it might be tricky to test some features of Flowable. There are couple of methods that help to verify expected behavior. 

```
class FlowableTesting {
    public static void main(String[] args) {
        Flowable<Integer> numbers = Flowable.fromArray(1, 1, 2, 3, 2, 3, 4, 5);

        numbers
            .test()
            .assertValueCount(8);
    }
}
```

### Error Handling

Error handling is one of the most important features we are going to use when working with `Flowable`.  We might need to recover from network errors, when remote service fails and we need to retry couple of times. 

Or we need to recover from execution of some logic when an exception was thrown. Or we need to recover from system errors when, for example, an exception was thrown because of missing space on hard drive. 

The following code is going to try 5 times to recover from error state. When it fails 6th time, it handle error by returning human readable message. 

```
class FLowableErrorHandling {
    public static void main(String[] args) {
        // network errors, remote service fails -> we might want to retry
        Flowable<Object> stream = Flowable
            .interval(1, TimeUnit.SECONDS)
            .map(it -> {
                System.out.println(it);
                throw new RuntimeException();
            })
            .retry(5)
            .onErrorReturnItem("Sorry, can't do it, god knows I tried.");
        stream.blockingSubscribe(System.out::println);
    }
}
```

Here how the output of execution looks like. 

```
0
0
0
0
0
0
Sorry, can't do it, god knows I tried.
```

#### Fetching Data

Lets say we want to fetch data from some server. We start with list of URLs and we want to end up with list HTML that is fetched from web servers. Also we want to take care of error handling. If an URL does not exist, we want to return human readable message. 

```
class FlowableFetchingData {
    public static void main(String[] args) {
        String[] urls = {
            "https://google.com",
            "https://doesnotexist-or-at-least-shoudnt.com"
        };
        Flowable<String> content = Flowable.fromArray(urls)
            .map(url -> new URL(url))
            .map(url -> {
                URLConnection connection = url.openConnection();
                byte[] bytes = connection.getInputStream().readAllBytes();
                return new String(bytes);
            })
            .onErrorReturnItem("Invalid URL");

        content.blockingSubscribe(System.out::println);

        content.test().assertNoErrors().assertComplete();
    }
        }
```

Wehn we run the code, we get HTML from google and then we get an exception because the second page does not exist. 

```
        <!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="en"><head><meta content="Search the world's information, including webpages, images, videos and more. Google has many special features to help you find exactly what you're looking for." name="description"><meta content="noodp" name="robots"><meta content="text/html; charset=UTF-8" http-equiv="Content-Type"><meta content="/logos/doodles/2018/doodle-snow-games-day-16-5525914497581056.3-l.png" itemprop="image"><meta content="Day 16 of the Doodle Snow Games!" property="twitter:title"><meta content="Day 16 of the Doodle Snow Games! &#10052; #GoogleDoodle" property="twitter:description"><meta content="Day 16 of the Doodle Snow Games! &#10052; #GoogleDoodle" property="og:description"><meta content="summary_large_image" property="twitter:card"><meta content="@GoogleDoodles" property="twitter:site"><meta content="https://www.google.com/logos/doodles/2018/doodle-snow-games-day-16-5525914497581056.2-2xa.gif" property="twitter:image"><meta content="https://www.google.com/logos/doodles/2018/doodle-snow-games-day-16-5525914497581056.2-2xa.gif" property="og:image"><meta content="782" property="og:image:width"><meta content="440" property="og:image:height"><meta content="https://www.google.com/logos/doodles/2018/doodle-snow-games-day-16-5525914497581056.2-2xa.gif" property="og:url"><meta content="video.other" property="og:type"><title>Google</title><script nonce="Hu3wwfF1hMZr1E1gyEUkSg==">(function(){window.google={kEI:'5xCRWtrqGMOkjwPav7WwBA',kEXPI:'1301799,1354276,1354916,1355218,1355527,1355675,1355761,1356040,1356179,1356691,1356779,1357034,1357219,1357843,1358012,3700062,3700278,3700521,4029815,4031109,4041302,4043492,4045841,4048347,4081038,4097153,4097922,4097929,4098733,4098740,4102827,4103845,4104658,4107914,4109490,4109596,4115697,4116279,4116731,4117539,4118798,4119032,4119034,4119036,4119811,4119812,4119820,4120660,4121175,4122511,4124727,4124850,4125837,4126202,4126754,4127086,4127418,4127744,4128586,4128995,4129520,4129633,4130782,4131247,4131834,4133509,4133751,4133752,4135025,4135249,4136073,4137467,4137597,4137646,4139722,4139730,4140273,4140319,4141160,4141174,4141601,4141707,4141915,4142071,4142555,4142834,4143278,4143676,4143817,4144231,4144442,4144545,4144704,4144761,4144803,4145088,4145461,4145485,4145772,4145836,4146147,4146754,4147026,4147436,4147900,4147943,4148061,4148267,4148279,4148304,4148498,4148608,4148966,4149017,4149271,4149304,4149338,4150005,4150018,4150022,4150185,4150413,4150429,4151013,4151016,4151048,4151054,4151263,4151295,4151361,4151389,4151703,4151918,4152049,4152151,4152944,4153062,4153222,4153365,4153421,4153545,4153550,4153847,4153952,4154017,4154043,4154081,4154191,4154203,4154376,4154506,4154946,4155118,4155444,4155569,4155604,4155684,4156008,4156123,4156629,4156631,4156635,4156657,4156663,4156701,4156877,10200083,10201957,10202524,15807764,19000288,19000423,19000427,19001999,19002548,19002880,19003321,19003323,19003325,19003326,19003328,19003329,19003330,19003407,19003408,19003409,19004309,19004516,19004517,19004518,19004519,19004520,19004521,19004887,19004901,19005005,19005009,19005010,41317155',authuser:0,kscs:'c9c918f0_5xCRWtrqGMOkjwPav7WwBA',u:'c9c918f0',kGL:'US'};google.kHL='en';})();(function(){google.lc=[];google.li=0;google.getEI=function(a){for(var b;a&&(!a.getAttribute||!(b=a.getAttribute("eid")));)a=a.parentNode;return b||google.kEI};google.getLEI=function(a){for(var b=null;a&&(!a.getAttribute||!(b=a.getAttribute("leid")));)a=a.parentNode;return b};google.https=function(){return"https:"==window.location.protocol};google.ml=function(){return null};google.wl=function(a,b){try{google.ml(Error(a),!1,b)}catch(d){}};google.time=function(){return(new Date).getTime()};google.log=function(a,b,d,c,g){if(a=google.logUrl(a,b,d,c,g)){b=new Image;var e=google.lc,f=google.li;e[f]=b;b.onerror=b.onload=b.onabort=function(){delete e[f]};google.vel&&google.vel.lu&&google.vel.lu(a);b.src=a;google.li=f+1}};google.logUrl=function(a,b,d,c,g){var e="",f=google.ls||"";d||-1!=b.search("&ei=")||(e="&ei="+google.getEI(c),-1==b.search("&lei=")&&(c=google.getLEI(c))&&(e+="&lei="+c));c="";!d&&google.cshid&&-1==b.search("&cshid=")&&(c="&cshid="+google.cshid);a=d||"/"+(g||"gen_204")+"?atyp=i&ct="+a+"&cad="+b+e+f+"&zx="+google.time()+c;/^http:/i.test(a)&&google.https()&&(google.ml(Error("a"),!1,{src:a,glmm:1}),a="");return a};}).call(this);(function(){google.y={};google.x=function(a,b){if(a)var c=a.id;else{do c=Math.random();while(google.y[c])}google.y[c]=[a,b];return!1};google.lm=[];google.plm=function(a){google.lm.push.apply(google.lm,a)};google.lq=[];google.load=function(a,b,c){google.lq.push([[a],b,c])};google.loadAll=function(a,b){google.lq.push([a,b])};}).call(this);google.f={};var a=window.location,b=a.href.indexOf("#");if(0<=b){var c=a.href.substring(b+1);/(^|&)q=/.test(c)&&-1==c.indexOf("#")&&a.replace("/search?"+c.replace(/(^|&)fp=[^&]*/g,"")+"&cad=h")};</script><style>#gbar,#guser{font-size:13px;padding-top:1px !important;}#gbar{height:22px}#guser{padding-bottom:7px !important;text-align:right}.gbh,.gbd{border-top:1px solid #c9d7f1;font-size:1px}.gbh{height:0;position:absolute;top:24px;width:100%}@media all{.gb1{height:22px;margin-right:.5em;vertical-align:top}#gbar{float:left}}a.gb1,a.gb4{text-decoration:underline !important}a.gb1,a.gb4{color:#00c !important}.gbi .gb4{color:#dd8e27 !important}.gbf .gb4{color:#900 !important}
        </style><style>body,td,a,p,.h{font-family:arial,sans-serif}body{margin:0;overflow-y:scroll}#gog{padding:3px 8px 0}td{line-height:.8em}.gac_m td{line-height:17px}form{margin-bottom:20px}.h{color:#36c}.q{color:#00c}.ts td{padding:0}.ts{border-collapse:collapse}em{font-weight:bold;font-style:normal}.lst{height:25px;width:496px}.gsfi,.lst{font:18px arial,sans-serif}.gsfs{font:17px arial,sans-serif}.ds{display:inline-box;display:inline-block;margin:3px 0 4px;margin-left:4px}input{font-family:inherit}a.gb1,a.gb2,a.gb3,a.gb4{color:#11c !important}body{background:#fff;color:black}a{color:#11c;text-decoration:none}a:hover,a:active{text-decoration:underline}.fl a{color:#36c}a:visited{color:#551a8b}a.gb1,a.gb4{text-decoration:underline}a.gb3:hover{text-decoration:none}#ghead a.gb2:hover{color:#fff !important}.sblc{padding-top:5px}.sblc a{display:block;margin:2px 0;margin-left:13px;font-size:11px}.lsbb{background:#eee;border:solid 1px;border-color:#ccc #999 #999 #ccc;height:30px}.lsbb{display:block}.ftl,#fll a{display:inline-block;margin:0 12px}.lsb{background:url(/images/nav_logo229.png) 0 -261px repeat-x;border:none;color:#000;cursor:pointer;height:30px;margin:0;outline:0;font:15px arial,sans-serif;vertical-align:top}.lsb:active{background:#ccc}.lst:focus{outline:none}</style><script nonce="Hu3wwfF1hMZr1E1gyEUkSg=="></script><link href="/images/branding/product/ico/googleg_lodp.ico" rel="shortcut icon"></head><body bgcolor="#fff"><script nonce="Hu3wwfF1hMZr1E1gyEUkSg==">(function(){var src='/images/nav_logo229.png';var iesg=false;document.body.onload = function(){window.n && window.n();if (document.images){new Image().src=src;}
        if (!iesg){document.f&&document.f.q.focus();document.gbqf&&document.gbqf.q.focus();}
        }
        })();</script><div id="mngb"> <div id=gbar><nobr><b class=gb1>Search</b> <a class=gb1 href="https://www.google.com/imghp?hl=en&tab=wi">Images</a> <a class=gb1 href="https://maps.google.com/maps?hl=en&tab=wl">Maps</a> <a class=gb1 href="https://play.google.com/?hl=en&tab=w8">Play</a> <a class=gb1 href="https://www.youtube.com/?gl=US&tab=w1">YouTube</a> <a class=gb1 href="https://news.google.com/nwshp?hl=en&tab=wn">News</a> <a class=gb1 href="https://mail.google.com/mail/?tab=wm">Gmail</a> <a class=gb1 href="https://drive.google.com/?tab=wo">Drive</a> <a class=gb1 style="text-decoration:none" href="https://www.google.com/intl/en/options/"><u>More</u> &raquo;</a></nobr></div><div id=guser width=100%><nobr><span id=gbn class=gbi></span><span id=gbf class=gbf></span><span id=gbe></span><a href="http://www.google.com/history/optout?hl=en" class=gb4>Web History</a> | <a  href="/preferences?hl=en" class=gb4>Settings</a> | <a target=_top id=gb_70 href="https://accounts.google.com/ServiceLogin?hl=en&passive=true&continue=https://www.google.com/" class=gb4>Sign in</a></nobr></div><div class=gbh style=left:0></div><div class=gbh style=right:0></div> </div><center><br clear="all" id="lgpd"><div id="lga"><a href="/search?site=&amp;ie=UTF-8&amp;q=Winter+Olympics&amp;oi=ddle&amp;ct=doodle-snow-games-day-16-5525914497581056&amp;hl=en&amp;kgmid=/m/03tng8&amp;sa=X&amp;ved=0ahUKEwia-rvBgL7ZAhVD0mMKHdpfDUYQPQgD"><img alt="Day 16 of the Doodle Snow Games!" border="0" height="220" src="/logos/doodles/2018/doodle-snow-games-day-16-5525914497581056.3-l.png" title="Day 16 of the Doodle Snow Games!" width="391" id="hplogo" onload="window.lol&&lol()"><br></a><br></div><form action="/search" name="f"><table cellpadding="0" cellspacing="0"><tr valign="top"><td width="25%">&nbsp;</td><td align="center" nowrap=""><input name="ie" value="ISO-8859-1" type="hidden"><input value="en" name="hl" type="hidden"><input name="source" type="hidden" value="hp"><input name="biw" type="hidden"><input name="bih" type="hidden"><div class="ds" style="height:32px;margin:4px 0"><input style="color:#000;margin:0;padding:5px 8px 0 6px;vertical-align:top" autocomplete="off" class="lst" value="" title="Google Search" maxlength="2048" name="q" size="57"></div><br style="line-height:0"><span class="ds"><span class="lsbb"><input class="lsb" value="Google Search" name="btnG" type="submit"></span></span><span class="ds"><span class="lsbb"><input class="lsb" value="I'm Feeling Lucky" name="btnI" onclick="if(this.form.q.value)this.checked=1; else top.location='/doodles/'" type="submit"></span></span></td><td class="fl sblc" align="left" nowrap="" width="25%"><a href="/advanced_search?hl=en&amp;authuser=0">Advanced search</a><a href="/language_tools?hl=en&amp;authuser=0">Language tools</a></td></tr></table><input id="gbv" name="gbv" type="hidden" value="1"></form><div id="gac_scont"></div><div style="font-size:83%;min-height:3.5em"><br></div><span id="footer"><div style="font-size:10pt"><div style="margin:19px auto;text-align:center" id="fll"><a href="/intl/en/ads/">Advertising�Programs</a><a href="/services/">Business Solutions</a><a href="https://plus.google.com/116899029375914044550" rel="publisher">+Google</a><a href="/intl/en/about.html">About Google</a></div></div><p style="color:#767676;font-size:8pt">&copy; 2018 - <a href="/intl/en/policies/privacy/">Privacy</a> - <a href="/intl/en/policies/terms/">Terms</a></p></span></center><script nonce="Hu3wwfF1hMZr1E1gyEUkSg==">(function(){window.google.cdo={height:0,width:0};(function(){var a=window.innerWidth,b=window.innerHeight;if(!a||!b){var c=window.document,d="CSS1Compat"==c.compatMode?c.documentElement:c.body;a=d.clientWidth;b=d.clientHeight}a&&b&&(a!=google.cdo.width||b!=google.cdo.height)&&google.log("","","/client_204?&atyp=i&biw="+a+"&bih="+b+"&ei="+google.kEI);}).call(this);})();</script><div id="xjsd"></div><div id="xjsi"><script nonce="Hu3wwfF1hMZr1E1gyEUkSg==">(function(){function c(b){window.setTimeout(function(){var a=document.createElement("script");a.src=b;google.timers&&google.timers.load.t&&google.tick("load",{gen204:"xjsls",clearcut:31});document.getElementById("xjsd").appendChild(a)},0)}google.dljp=function(b,a){google.xjsu=b;c(a)};google.dlj=c;}).call(this);if(!google.xjs){window._=window._||{};window._DumpException=window._._DumpException=function(e){throw e};google.dljp('/xjs/_/js/k\x3dxjs.hp.en_US.ZGniKUPATng.O/m\x3dsb_he,d/am\x3dKEA/rt\x3dj/d\x3d1/t\x3dzcms/rs\x3dACT90oG0bRkB1ZYXMIzbfWTOX9UPBUjHzA','/xjs/_/js/k\x3dxjs.hp.en_US.ZGniKUPATng.O/m\x3dsb_he,d/am\x3dKEA/rt\x3dj/d\x3d1/t\x3dzcms/rs\x3dACT90oG0bRkB1ZYXMIzbfWTOX9UPBUjHzA');google.xjs=1;}google.pmc={"sb_he":{"agen":true,"cgen":true,"client":"heirloom-hp","dh":true,"dhqt":true,"ds":"","ffql":"en","fl":true,"host":"google.com","isbh":28,"jsonp":true,"msgs":{"cibl":"Clear Search","dym":"Did you mean:","lcky":"I\u0026#39;m Feeling Lucky","lml":"Learn more","oskt":"Input tools","psrc":"This search was removed from your \u003Ca href=\"/history\"\u003EWeb History\u003C/a\u003E","psrl":"Remove","sbit":"Search by image","srch":"Google Search"},"nds":true,"ovr":{},"pq":"","refpd":true,"rfs":[],"sbpl":24,"sbpr":24,"scd":10,"sce":5,"stok":"5usFUMrGMcumyzEyipRr-jCctwU"},"d":{},"ZI/YVQ":{},"YFCs/g":{}};google.x(null,function(){});(function(){var r=[];google.plm(r);})();(function(){var ctx=[]
        ;google.jsc && google.jsc.x(ctx);})();</script></div></body></html>
        Invalid URL
```

#### Throttling

Throttling is mechanism to avoid memory, network or CPU issues by reducing number of operations. In short, it is one of the mechanisms to avoid system overload. 

Lets create flowable that emits a random integer every 100 milliseconds. That could be a lot for our system and we can handle it by taking only one number every second. 

```
// throttling -> throw away some data to handle the load
Flowable
  .interval(100, TimeUnit.MILLISECONDS)
  .map(x -> new Random().nextInt())
  .sample(1, TimeUnit.SECONDS)
  .blockingSubscribe(System.out::println);
```

Another way to handle the load is to create buffer of certain size, 5 in our case, and then do operation on that buffer. We are going to take average number from every 5 values we receive. 

```
// throttling -> get average
Flowable
  .interval(100, TimeUnit.MILLISECONDS)
  .map(x -> new Random().nextInt())
  .buffer(5)
  .map(numbers -> numbers.stream().mapToDouble(d-> d).average().getAsDouble())
  .blockingSubscribe(System.out::println);
```

Another way to handle big load of values is to get only first value. We are going to Therea only first value that was received. 

```
// throttling -> throttle first value
Flowable
  .interval(100, TimeUnit.MILLISECONDS)
  .map(x -> new Random().nextInt())
  .throttleFirst(1, TimeUnit.SECONDS)
  .blockingSubscribe(System.out::println);
```

#### Scheduler

There are different types of schedulers when using Flowable. Choosing scheduler is going to inform flowable what kind of strategy it should use for processing the items. 

```
class Scheduler {
    public static void main(String[] args) {
        Flowable.range(1, 5)
            .map(x -> Thread.currentThread().getName())
            .subscribeOn(Schedulers.trampoline())
            // .subscribeOn(Schedulers.io())
            // .subscribeOn(Schedulers.computation())
            .subscribe(System.out::println);
    }
}
```

When we use trampoline scheduler, we receive the same thread 5 times. 

```
main
main
main
main
main
```

When we use io scheduler, and we do some IO operations, we will see IO threads were created. 

