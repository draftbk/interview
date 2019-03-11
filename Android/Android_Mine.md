# 安卓相关

### 四大组件

四大组件包括了 Activity Service Content Provider 以及 BroadcastReceiver

Activity: 也就是一般来说用户看的的界面，可以显示控件，是主要用于和用户交互的地方。

Service: 是在 Android 中实现程序后台运行的解决方案，适合执行不用和用户交互但是需要长期执行的任务，例如听音乐等。

Content Provider: 是用于实现不同程序之间数据共享的功能，允许一个程序访问另一个程序中的数据，并且还保证数据的安全性。·

BroadcastReceiver: 一种用于响应系统范围广播通知的组件，应用可以使用它对对感兴趣的外部事件(如当电话呼入时，或者数据网络可用时)进行接收并做出响应，还可以发送广播

### Activity 生命周期

![](images/activity_lifecycle.png)

生命周期有七个:

onCreate(), onStart(), onResume()在启动应用时顺序执行，这三个分别对应 onPause(), onStop(), onDestory()
还有一个 onRestart()，加起来七个。

- onCreate() 表示应用正在被创建，常用来初始化工作，例如调用 setContentView加载界面布局资源，初始化Activity所需数据等
- onStart()：表示Activity正在被启动，此时Activity可见但不在前台，还处于后台，无法与用户交互；
- onResume()：表示Activity获得焦点，此时Activity可见且在前台并开始活动，这是与onStart的区别所在；
- onPause()：表示Activity正在停止，此时可做一些存储数据、停止动画等工作，但是不能太耗时，因为这会影响到新Activity的显示，onPause必须先执行完，新Activity的onResume才会执行；
- onStop()：表示Activity即将停止，可以做一些稍微重量级的回收工作，比如注销广播接收器、关闭网络连接等，同样不能太耗时；
- onDestroy()：表示Activity即将被销毁，这是Activity生命周期中的最后一个回调，常做回收工作、资源释放；

##### 不同场景

- 启动Activity：`onCreate()` --> `onStart()` --> `onResume()`，Activity进入运行状态。
- Activity退居后台：当前Activity转到新的Activity界面或按Home键回到主屏：`onPause()` --> `onStop()`，进入停滞状态；这里有一种特殊情况，如果新Activity采用了透明主题，那么当前Activity不会回调`onStop()`。
- Activity返回前台：`onRestart()` --> `onStart()` --> `onResume()`，再次回到运行状态。
- Activity退居后台，且系统内存不足，系统会杀死这个后台状态的Activity，若再次回到这个Activity，则会走`onCreate()` --> `onStart()` --> `onResume()`。
- 锁定屏与解锁屏幕只会调用`onPause()`，而不会调用`onStop()`方法，开屏后则调用`onResume()`。
- Activity A 启动 Activity B 时：A.onPause() --> B.onCreate()，B.onStart()，B.onResume() --> A.onStop()，如果B是个透明的，或者是对话框的样式，就不会调用A.onStop()。


### Service 的启动方式

- startService()：只是启动Service，Activity和Service并没有绑定，只有当Service调用stopService()服务才会终止。
- bindService()：这种启动方式Activity和Service进行了绑定，启动Service的组件可以通过回调获取Service的代理对象和Service交互；当启动方销毁时，Service也会自动进行unBind()操作，当发现所有绑定都进行了unBind()时才会销毁Service。

### Activity中启动，绑定 Service

Activity通过bindService(Intent service，ServiceConnection conn，int flags)跟Service进行绑定，当绑定成功的时候Service会将代理对象通过回调的形式传给conn，这样我们就拿到了Service提供的服务代理对象。
在Activity中可以通过startService()和bindService()方法启动Service。一般情况下如果想获取Service的服务对象那么肯定需要通过bindService()方法，比如音乐播放器，第三方支付等。如果仅仅只是为了开启一个后台任务那么可以使用startService()方法。

### Service 生命周期


![](images/service_lifecycle.png)

- onCreate()：如果service没被创建过，调用startService()后会执行onCreate()回调；如果service已处于运行中，调用startService()不会执行onCreate()方法。也就是说，onCreate()只会在第一次创建service时候调用，多次执行startService()不会重复调用onCreate()，此方法适合完成一些初始化工作；
- onStartComand()：服务启动时调用，此方法适合完成一些数据加载工作，比如会在此处创建一个线程用于下载数据或播放音乐；
- onBind()：服务被绑定时调用；
- onUnBind()：服务被解绑时调用；
- onDestroy()：服务停止时调用；

### 如何保证 Service 不被杀死
进程优先级由高到低：前台进程 一 可视进程 一 服务进程 一 后台进程 一 空进程
可以使用startForeground将service放到前台状态，这样低内存时，被杀死的概率会低一些；
有下面这些方法：

- onStartCommand方式中，返回START_STICKY或则START_REDELIVER_INTENT

 - START_STICKY：如果返回START_STICKY，表示Service运行的进程被Android系统强制杀掉之后，Android系统会将该Service依然设置为started状态（即运行状态），但是不再保存onStartCommand方法传入的intent对象
 - START_NOT_STICKY：如果返回START_NOT_STICKY，表示当Service运行的进程被Android系统强制杀掉之后，不会重新创建该Service
 - START_REDELIVER_INTENT：如果返回START_REDELIVER_INTENT，其返回情况与START_STICKY类似，但不同的是系统会保留最后一次传入onStartCommand方法中的Intent再次保留下来并再次传入到重新创建后的Service的onStartCommand方法中


- 提高Service的优先级
在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广；
在onDestroy方法里重启Service
当service走到onDestroy()时，发送一个自定义广播，当收到广播时，重新启动service；
- 提升Service进程的优先级

- 系统广播监听Service状态
- 将APK安装到/system/app，变身为系统级应用

### Handler, Looper, Message

一个handler持有一个消息队列的引用和它构造时所属线程的Looper的引用.
也就是说,一个handler必定有它对应的消息队列和Looper,
一个线程至多能有一个Looper,也就至多能有一个消息队列.
一个线程中可以有多个handler.

在主线程中new了handler对象后,这个handler对象自动和主线程自动生成的Looper以及消息队列关联上了.
子线程中拿到主线程中handler的引用,发送消息后,消息对象就会发送到target属性对应的那个handler对应的消息队列中去,由对应的Looper来取出处理(子线程msg–>主线程handler–>主线程messageQueue–>主线程Looper—>主线程Handler的handMessage).
而消息发送到主线程handler,那么也就是发送到主线程的消息队列,用主线程中的Looper轮询.

##### 为什么要用 Handler

因为安卓更新ui只能在主线程中执行(只是大多数的时候出于安全方面的考虑，我们经常在主线程更新 ui)，而在实际情况中经常要在子线程访问UI。为什么系统不允许子线程访问UI呢，因为AndroidUI不是线程安全的，如果在多线程操控UI可能会导致UI控件处于不可预知的状态；那为什么系统不给UI控件的访问加锁呢？首先上锁机制会让UI访问的逻辑变得复杂，其次加锁机制会降低访问效率。因此，安卓之所以提供Handler是为了解决在子线程不能更新UI的矛盾。

##### 对于Message Queue:
指的是消息队列，，即存放供handler处理的消息。MessageQueue主要包括两个操作:插入和读取。读取本身会伴随删除操作。虽然名字叫队列，但其实内部实现是通过单链表的形式实现的（单链表在插入和删除上比较有优势，不是连续存储）。

##### 对于Looper:
Looper在消息机制中进行消息循环，像一个泵，不断地从MessageQueue中查看是否有新消息并提取，交给handler处理。Handler机制一定要Looper,在线程中通过Looper.prepare()为当前线程创建一个Looper,并使用Looper.loop()来开启消息的读取。为什么在平常Activity主线程使用时没有使用到Looper呢？因为对于主线程(UI线程)，会自动创建一个Looper 驱动消息队列获取消息，所以Looper可以通过getMainLooper获取到主线程的Looper。

通过quit/quitSafely可以退出Looper,区别在于quit会直接退出，quitSafely会把消息队列已有的消息处理完毕后才退出Looper。

##### 对于Handler
Handler可以发送和接收消息。例如使用sendMessage就可以发送向消息队列插入一条消息，MessageQueue的next()方法就将消息返回给Looper，Looper收到消息后交由Handler处理，调用handMessage处理消息。


### Android 中用到的设计模式

Application 是单例模式