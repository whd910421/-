Rxjava 笔记

RxJava 的异步实现，是通过一种扩展的观察者模式来实现的。
RxJava 有四个基本概念：Observable (可观察者，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

与传统观察者模式不同， RxJava 的事件回调方法除了普通事件 onNext() （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：onCompleted() 和 onError()。

onCompleted(): 事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发 onCompleted() 方法作为标志。
onError(): 事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。

Observer的接口要实现 onNext()  onCompleted() onError()
Subscriber 对 Observer 接口进行了一些扩展，但他们的基本使用方式是完全一样的,除了上述接口还有onStart()和unsubscribe()这两个不是必要的，有时候不想处理onCompletd 和 onError 可以用 Action0 来代替

onStart(): 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。我看onStart()方法总是在主线程调用的，应该是后来改了，如果初始化的想在非主线程做些事，可以用doOnSubscribe()

unsubscribe()：在这个方法被调用后，Subscriber 将不再接收事件。

Observable 即被观察者，它决定什么时候触发事件以及触发怎样的事件。 RxJava 使用 create() 方法来创建一个 Observable 也可以用just，from来创建。

创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：
observable.subscribe(subscriber);

线程控制 Scheduler 
Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。 * subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。 * observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

observeOn() 指定的是它之后的操作所在的线程。因此如果有多次切换线程的需求，
只要在每个想要切换线程的位置调用一次 observeOn() 即可
当使用了多个 subscribeOn() 的时候，只有第一个 subscribeOn() 起作用。
默认情况下， doOnSubscribe() 执行在 subscribe() 发生的线程；而如果在 doOnSubscribe() 之后有 subscribeOn() 的话，它将执行在离它最近的 subscribeOn() 所指定的线程。

变换 所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列
map(): 事件对象的直接变换，实现Func1<before， after> 将before类型转换成after类型，相当于一对一转换
flatMap():可以进行一对多的转化Func1<before，, Observable<after>> 转换后得到新类型的Observerble
merge():两个任务并发进行，全部处理完毕之后在更新数据
Observable.merge(obs1,obs2) //这里没用到create，因此不是单独生成一个Observable
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<String>() {}
可以直接单独定义Observable，然后分别放到merge函数中，注意：Observable subscribeOn了一个新线程，merge 后的observeOn，也要单独一个线程，不然就在subscribeOn中的线程进行操作。
Observable obs1=Observable.create(new Observable.OnSubscribe<String>(){
    @Override
    public void call(Subscriber<? super String> subscriber) {
        try {
            Thread.sleep(500);
            subscriber.onNext(" aaa");
            subscriber.onCompleted();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}).subscribeOn(Schedulers.newThread());
connect():Observable发送事件1-6，两个观察者同时观察这个Observable 
要求：每发出一个事件，观察者A和观察者都会收到，而不是先把所有的时间发送A,然后再发送给B
observable.subscribe(a1);
observable.subscribe(a2);
observable.connect();

take(): 取前i个事件
//取前四个
.take(4)
//取前四个中的后两个
.takeLast(2)

1.Observable和Subscriber可以做任何事情
Observable可以是一个数据库查询，Subscriber用来显示查询结果；Observable可以是屏幕上的点击事件，Subscriber用来响应点击事件；Observable可以是一个网络请求，Subscriber用来显示请求结果。

2.Observable和Subscriber是独立于中间的变换过程的。
在Observable和Subscriber中间可以增减任何数量的map。整个系统是高度可组合的，操作数据是一个很简单的过程。

对于连续事件，如果不想继续订阅一定要UnSubscribe，不然会内存泄露，其实对于非连续事件例如一次网络请求也最好加上因为如果意外中断，不至于内存泄露