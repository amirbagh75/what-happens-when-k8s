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


در دست ترجمه ....

</div>