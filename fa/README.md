<div dir='auto'>

# چی میشه وقتی که ... اینبار برای کوبرنتیز!

</div>

<div dir='auto'>
  
تصور کنید میخوایم به وسیله کوبرنتیز چند تا کانتینر Nginx رو دپلوی کنیم. دستور زیر رو وارد میکنیم:
  
</div>

<code>
kubectl run nginx --image=nginx --replicas=3  
</code>

<div dir='auto'>
  
و بعد enter میزنیم. چند ثانیه بعد ۳ تا pod از nginx ساخته میشه و بین سرور ها پخش میشه.این عالیه! اما بیاین ببینیم واقعا چه اتفاقی رخ میده؟

یکی از چیزهای قابل توجه درباره کوبرنتیز اینه که عملیات دپلوی روی زیرساخت رو به وسیله یک API راحت و قابل فهم به ما ارائه میکنه. پیچیدگی، زیر یک لایه ساده و انتزاعی پنهان شده است. اما برای درک کامل ارزشی که کوبرنتیز خلق کرده، ‌بهتر و مفیده که بدونیم داخلش چه خبره. این راهنمایی به شما یک دید کامل از چرخه ای که یک درخواست در کوبرنتیز طی میکند ارائه میدهد، و هرجا نیاز باشد به کد منبع مرتبط با اون بخش اشاره میکنه تا نشون بده که واقعا چه اتفاقی داره میفته.

این دایکیومنت در جریان است (یعنی دائما در حال به روز شدن و تکمیل تر شدن است). اگر شما هم تمایل دارید که در تکمیل آن کمک کنید میتوانید contribute کنید!

## فهرست مطالب
</div>

1. [kubectl](#kubectl)
   - [validation and generators](#validation-and-generators)
   - [API groups and version negotiation](#api-groups-and-version-negotiation)
   - [client-side auth](#client-auth)
2. [kube-apiserver](#kube-apiserver)
   - [authentication](#authentication)
   - [authorization](#authorization)
   - [admission control](#admission-control)
3. [etcd](#etcd)
4. [initializers](#initializers)
5. [control loops](#control-loops)
   - [deployments controller](#deployments-controller)
   - [replicasets controller](#replicasets-controller)
   - [informers](#informers)
   - [scheduler](#scheduler)
6. [kubelet](#kubelet)
   - [pod sync](#pod-sync)
   - [CRI and pause containers](#cri-and-pause-containers)
   - [CNI and pod networking](#cni-and-pod-networking)
   - [inter-host networking](#inter-host-networking)
   - [container startup](#container-startup)
7. [wrap-up](#wrap-up)

## kubectl


### Validation and generators

<div dir='auto'>

خب، بیاین شروع کنیم. ما فقط یک enter توی terminal زدیم. خب چی میشه؟

اولین کار اعتبار سنجی سمت کلاینت است. با این کار، kubectl مطمئن میشه که ریکوئستی که اشکال داره قبل از این که به دست api server برسه fail میشه و فشار الکی و مضاعف به api server وارد نمیشه. با این کار به خاطر این که پردازش بیهوده ای توسط api server رخ نمیده، performance بهتر میشه.

بعد از موفقیت‌آمیز بودن اعتبار سنجی، kubectl شروع میکنه به ساختن یک HTTP request که به سمت api server ارسالش کنه. همیشه همه درخواست ها و تلاش ها برای اتصال به کوبرنتیز به هر منظوری از طریق api server انجام میشه، که api server هم به نحو خودش با پایگاه داده key-value ای به اسم etcd صحبت میکنه. kubectl هم از این قاعده مستثنی نیست. برای درست کردن این درخواست، kubectl از [generator](https://kubernetes.io/docs/reference/kubectl/conventions/#generators) ها در کوبرنتیز کمک میگیرد که اصطلاحا وظیفه serialization کردن را دارد. (منظور اینه که درخواست ما رو به چیزهای قابل فهم توسط کوبرنتیز تبدیل میکنه)

چیزی که ممکنه واضح نباشه اینه که ما به وسیله کامند `kubectl run` میتونیم resource های مختلف کوبرنتیزی رو اجرا کنیم و محدود به Deployment ها نیستیم. 
 در صورتی که واضح و شفاف از طریق `generator--` به kubectl نگفته باشیم چی میخوایم، kubectl از طریق روش هایی resource مدنظر رو [حدس](https://github.com/kubernetes/kubernetes/blob/v1.14.0/pkg/kubectl/cmd/run/run.go#L319-L339) میزنه.

 برای مثال، درخواست هایی که توشون `restart-policy=Always--` وجود داشته باشه، به عنوان Deployments در نظر گرفته میشوند و درخواست هایی که داخلشون `restart-policy=Never--` وجود داشته باشه به عنوان Pod در نظر گرفته میشوند.
همچنین kubectl چیزهای دیگه ای رو هم مد نظر میگیره، مثلا ذخیره کردن تاریخچه درخواست ها (برای rollouts و auditing) یا این که با دادن ‍`dry-run--` از ثبت و ضبط شدن تاریخچه صرف نظر میکنه.

بعد از این که kubectl متوجه شد ما میخوایم یک Deployment راه بندازیم، از `DeploymentAppsV1` generator استفاده میکنه تا بتونه یک runtime object از درخواست بسازه. runtime object یک اصطلاح کلی برای resource های کوبرنتیز میباشد.

</div>

### API groups and version negotiation

<div dir='auto'>

در دست ترجمه ...

</div>

### Client auth

<div dir='auto'>

یکی از مراحل مهم در ارسال درخواست به api server توسط kubectl، بحث احراز هویت یا همان auth میباشد.

اطلاعات مهم و ضروری ای که برای فرایند احراز هویت نیاز است تا از سمت kubectl به api server ارسال شود در فایلی به نام `kubeconfig` ذخیره شده است. این فایل میتونه توی سیستم شما قرار گرفته باشه یا هرجای دیگه ای! kubectl برای پیدا کردن آدرس فایل سه تا کار میکنه:

- اگر توی flag مربوطه یعنی، `kubeconfig--` آدرس فایل داده شده باشد
- اگر این flag داده نشده بود، از طریق متغیر محیطی یا همان env ای به نام `KUBECONFIG$`
- یا هم در نهایت داخل [دایرکتوری](https://github.com/kubernetes/client-go/blob/master/tools/clientcmd/loader.go#L52) `kube.`  به دنبال `kubeconfig` میگردد.

بعد از این که kubectl فایل `kubeconfig` رو پیدا کرد، اطلاعات لازم برای اتصال به api server رو سرجمع میکنه. چیزهایی مثل این که به چه cluster ای وصل بشه و با چه context ای. همچنین اطلاعات user ای که درخواست رو داره ارسال میکنه. بعد از این ها kubectl شروع میکنه به ساختن درخواست HTTP متناسب با سازوکار Auth:

  - ارسال گواهینامه عمومی که با استاندارد [`X.509`](https://fa.wikipedia.org/wiki/%D8%A7%D8%B3%D8%AA%D8%A7%D9%86%D8%AF%D8%A7%D8%B1%D8%AF_%D8%A7%DA%A9%D8%B3%DB%B5%DB%B0%DB%B9) ایجاد شده
- توکن های bearer هم در header مربوطه یعنی  `Authorization` قرار میگیرند.
- نام کاربری و رمزعبور ها به وسیله `HTTP basic authentication` ارسال میشن.
- و سازوکارهایی مثل `OpenID auth` هم از قبل توسط کاربر انجام میشود تا توکن مربوطه مثل توکن های bearer ارسال شود.

</div>
