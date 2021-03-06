## Activity

### onActivityResult
在react-native的`ReactContextBaseJavaModule`中，可以通过`BaseActivityEventListener`来监听`onActivityResult`。

```java
// 在ReactContextBaseJavaModule内
private static ReactApplicationContext reactContext;
private final ActivityEventListener mActivityEventListener = new BaseActivityEventListener() {
    @Override
    public void onActivityResult(Activity activity, int requestCode, int resultCode, Intent data) {
        // 这里第一个参数activity，就是接收Result的Activity，一般就是public class MainActivity extends ReactActivity。因为这里的onActivityResult其实是收到Activity的一个转发，Activity的onActivityResult方法是没这个参数的。
    }
};

SimpleModule(ReactApplicationContext context) {
    super(context);
    reactContext = context;
    reactContext.addActivityEventListener(mActivityEventListener);
}
```

## Service
* 查看服务状态(adb shell内执行)：dumpsys activity services com.rntest.MyRemoteService

那些在后台的服务，即使已经销毁(也就是上面命令已经找不到该服务)，但是通过top命令，还是会看到那个进程依然存在，不过是处于睡眠状态。

### 简单的Service实例

```java
// TestService2.java
// 服务类
package com.rntest2;

import android.app.Service;
import android.util.Log;
import android.os.IBinder;
import android.os.Binder;
import android.content.Intent;

public class TestService2 extends Service {  
    private final String TAG = "TestService2";
    private boolean quit;
    private int count;

    private MyBinder binder = new MyBinder();  
    public class MyBinder extends Binder  
    {  
        public int getCount()  
        {  
            return count;  
        }  
    }  

    //必须实现的方法,绑定改Service时回调该方法  
    @Override  
    public IBinder onBind(Intent intent) {  
        Log.i(TAG, "onBind方法被调用!");
        return binder;  
    }
  
    //Service被创建时回调  
    @Override  
    public void onCreate() {  
        super.onCreate();  
        Log.i(TAG, "onCreate方法被调用!");  
        //创建一个线程动态地修改count的值  
        new Thread()  
        {  
            public void run()   
            {  
                while(!quit)  
                {  
                    try  
                    {  
                        Thread.sleep(1000);  
                    }catch(InterruptedException e){e.printStackTrace();}
                    count++;  
                }  
            };  
        }.start();  
    }

    //Service被关闭前回调
    @Override  
    public void onDestroy() {  
        super.onDestroy();  
        this.quit = true;  
        Log.i(TAG, "onDestroyed方法被调用!");
        Log.i(TAG, ""+count);
    }

    //Service断开连接时回调  
    @Override  
    public boolean onUnbind(Intent intent) {  
        Log.i(TAG, "onUnbind方法被调用!");  
        return true;  
    }  

    @Override  
    public void onRebind(Intent intent) {  
        Log.i(TAG, "onRebind方法被调用!");  
        super.onRebind(intent);  
    }  
}
```

```xml
<!-- AndroidManifest.xml 添加上面的服务类-->
<service android:name="com.rntest2.TestService2" />
```

```java
// 调用方式，因为界面用react-native，所以调用是在ReactMethod里执行。

// startService方式
@ReactMethod
public void testService2(int flag) {
    Intent intent = new Intent(reactContext, TestService2.class);
    if(flag == 1) {
        reactContext.startService(intent);
    } else {
        reactContext.stopService(intent);
    }
}

// bindService方式，要先建立连接，并且储存好binder
TestService2.MyBinder binder;
private ServiceConnection conn = new ServiceConnection() {
    //Activity与Service连接成功时回调该方法
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        System.out.println("------Service Connected-------");
        binder = (TestService2.MyBinder) service;
    }

    //Activity与Service断开连接时回调该
    @Override
    public void onServiceDisconnected(ComponentName name) {
        System.out.println("------Service DisConnected-------");
    }
};

@ReactMethod
public void bindService2(int flag, Callback cb) {
    Intent intent = new Intent(reactContext, TestService2.class);
    if(flag == 1) {
        reactContext.bindService(intent, conn, Service.BIND_AUTO_CREATE);
    } else if(flag == 2) {
        cb.invoke(binder.getCount());
    } else {
        reactContext.unbindService(conn);
    }
}
```

### IntentService
普通Service不是一个单独的线程或者进程，所以不应该在普通Service里执行耗时任务。IntentService会开启一个工作线程来处理耗时任务，所以执行耗时任务的话，应该用IntentService。发送到IntentService的任务，会依次执行，一次只执行一个。

IntentService和普通Service的主要不同，是有onHandleIntent方法。下面是一个重写它的示例。
```java
@Override
protected void onHandleIntent(Intent intent) {
    //Intent是从Activity发过来的，携带识别参数，根据参数不同执行不同的任务
    String action = intent.getExtras().getString("param");
    if(action.equals("s1"))Log.i(TAG,"启动service1");
    else if(action.equals("s2"))Log.i(TAG,"启动service2");
    else if(action.equals("s3"))Log.i(TAG,"启动service3");

    //让服务休眠2秒
    try{
        Thread.sleep(5000);
    }catch(InterruptedException e){e.printStackTrace();}
}
```

### 前台服务
因为服务在一般在后台进行，可能会被系统杀掉，所以可以让服务在前台进行，让它不那么容易被杀掉。
* http://blog.huangyuanlove.com/2018/12/27/Android-O-%E9%80%82%E9%85%8DNotificationChannel/

```java
// 重写onCreate方法，开启一个Notification，让Service在前台运行
@Override
public void onCreate() {
    Log.i(TAG,"onCreate");
    super.onCreate();
    Notification.Builder localBuilder = new Notification.Builder(this);
    localBuilder.setContentIntent(PendingIntent.getActivity(this, 0, new Intent(this, MainActivity.class), 0));
    localBuilder.setAutoCancel(false);
    localBuilder.setSmallIcon(R.mipmap.ic_launcher_round);
    localBuilder.setTicker("Foreground Service Start");
    localBuilder.setContentTitle("测试TestService3");
    localBuilder.setContentText("正在运行...");
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) { // API 26(安卓8)以上才有下面setChannelId等这些方法
        String channelId = "TestService3_channel";
        String channelName = "测试TestService3";
        int importance = NotificationManager.IMPORTANCE_HIGH;
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        NotificationChannel channel = notificationManager.getNotificationChannel(channelId);
        if (channel == null) {
            channel = new NotificationChannel(channelId, channelName, importance);
        }
        notificationManager.createNotificationChannel(channel); // 如果channel已经在，就不会重复创建
        if (channel.getImportance() == NotificationManager.IMPORTANCE_NONE) { // 如果通知被用户关了，提示用户打开它
            Intent intent = new Intent(Settings.ACTION_CHANNEL_NOTIFICATION_SETTINGS);
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            intent.putExtra(Settings.EXTRA_APP_PACKAGE, getPackageName());
            intent.putExtra(Settings.EXTRA_CHANNEL_ID, channel.getId());
            startActivity(intent);
            Toast.makeText(this, "请手动将通知打开", Toast.LENGTH_SHORT).show();
            return;
        } else {
            localBuilder.setChannelId("TestService3_channel");
        }
    }
    startForeground(1, localBuilder.build());
}
```

### 远程服务
https://www.jb51.net/article/158224.htm
https://www.cnblogs.com/demodashi/p/8481566.html
https://blog.csdn.net/luoyanglizi/article/details/51594016

#### aidl实现
注意下面的代码涉及到两个app，不过代码基本一样。一个的包名是com.rntest(提供服务的app)、另一个是com.rntest3。

提供服务的app，创建aidl文件，并且make一下project让as生成接口文件(aidl本质上就是这个接口的模板)。想使用这个服务的app，也要把这个文件，连同目录(包)复制到自己工程的aidl目录下，并且make project。aidl文件如下：

```java
// IProcessInfo.aidl
package com.rntest;
interface IProcessInfo {
    void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
    int getProcessId();
}
```

实现上面这个接口。下面的这个getProcessId方法，主要是为了观察进程。
```java
package com.rntest;

import android.os.Process;
import android.os.RemoteException;
import android.util.Log;

public class IProcessInfoImpl extends IProcessInfo.Stub {
    @Override
    public int getProcessId() throws RemoteException {
        Log.d("TestRemoteService", "IProcessInfoImpl.java pid = " + Process.myPid());
        Log.d("TestRemoteService", "IProcessInfoImpl.java thread id = " + Thread.currentThread().getId());
        return Process.myPid();
    }

    public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
                          double aDouble, String aString) throws RemoteException {

    }
}
```

创建service
```java
package com.rntest;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.util.Log;
import android.os.Process;

import androidx.annotation.Nullable;

public class MyRemoteService extends Service {
    IProcessInfoImpl mProcessInfo = new IProcessInfoImpl();
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.d("TestRemoteService", "MyRemoteService.java pid = " + Process.myPid());
        Log.d("TestRemoteService", "MyRemoteService.java thread id = " + Thread.currentThread().getId());
        return mProcessInfo;
    }
}
```

在AndroidManifest.xml注册这个service。因为要供外部调用，所以设置intent-filter
```xml
<service
    android:name=".MyRemoteService"
    android:process=":remote" >
    <intent-filter>
        <action android:name="com.rntest.service.bind" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</service>
```

调用service。
```java
// 创建ServiceConnection
private ServiceConnection mRemoteServiceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {

        Log.d("TestRemoteService", "MyRemoteService onServiceConnected");
        // 通过aidl取出数据
        IProcessInfo processInfo = IProcessInfo.Stub.asInterface(service);
        try {
            Log.d("TestRemoteService", "SimpleModule.java from service pid = " + processInfo.getProcessId());
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        Log.d("TestRemoteService", "MyRemoteService onServiceDisconnected");
    }
};

// 提供服务的app，可以直接通过class调用。
@ReactMethod
public void aidlTest() {
    Intent intent = new Intent(reactContext, MyRemoteService.class);
    reactContext.bindService(intent, mRemoteServiceConnection, BIND_AUTO_CREATE);
}

// 其他app，通过action调用
@ReactMethod
public void aidlTest() {
    Intent intent = new Intent();
    intent.setAction("com.rntest.service.bind");//Service的action
    intent.setPackage("com.rntest");//App A的包名
    reactContext.bindService(intent, mRemoteServiceConnection, BIND_AUTO_CREATE);
}

// 用于观察app所在进程
@ReactMethod
public void doNothing() {
    Log.d("TestRemoteService", "doNothing pid = " + Process.myPid());
    Log.d("TestRemoteService", "doNothing thread id = " + Thread.currentThread().getId());
}
```

测试过程，两个app都打开，都调用doNothing和aidlTest。下面是logcat打印结果，只取重要(pid)部分。
```log
<!--  分别在两个app调用doNothing，打印他们的pid -->
com.rntest D/TestRemoteService: doNothing pid = 31487
com.rntest3 D/TestRemoteService: doNothing pid = 31007

<!-- 在com.rntest3调用aidlTest -->
com.rntest:remote D/TestRemoteService: MyRemoteService.java pid = 31580
com.rntest:remote D/TestRemoteService: IProcessInfoImpl.java pid = 31580
com.rntest3 D/TestRemoteService: SimpleModule.java from service pid = 31580

<!-- 在com.rntest调用aidlTest -->
com.rntest:remote D/TestRemoteService: MyRemoteService.java pid = 31580
com.rntest:remote D/TestRemoteService: IProcessInfoImpl.java pid = 31580
com.rntest D/TestRemoteService: SimpleModule.java from service pid = 31580
```

下面是开启`adb shell`后，执行`top | grep com.rntest`的结果：
```log
31487 u0_a272      10 -10 1.2G 140M 100M S 11.1   3.6   0:11.87 com.rntest
31580 u0_a272      20   0 1.0G  66M  51M S  0.0   1.7   0:00.92 com.rntest:remote
31007 u0_a273      20   0 1.2G 137M  99M S  0.0   3.5   0:11.91 com.rntest3
```

结论：服务在一个独立的进程里开启，名字叫com.rntest:remote，而且不只自己所在app可以调用，其他app也可以调用。

#### Messenger实现

提供服务的app
```java
// MessengerService.java
public class MessengerService extends Service {
    static final String TAG = "MessengerServiceTest";
    static final int MSG_SAY_HELLO = 1;

    class ServiceHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            Log.i(TAG, "MessengerService.java msg.what = " + msg.what);
            switch (msg.what) {
                case MSG_SAY_HELLO:
                    Messenger messenger = msg.replyTo;
                    if(messenger != null) {
                        Message messg = Message.obtain(null, MSG_SAY_HELLO);
                        try {
                            //向客户端发送消息
                            messenger.send(messg);
                        } catch (RemoteException e) {
                            e.printStackTrace();
                        }
                    }
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    final Messenger mMessenger = new Messenger(new ServiceHandler());

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "MessengerService.java onBind pid = " + Process.myPid());
        return mMessenger.getBinder();
    }
}
```

提供服务的app注册这个service
```xml
<service
    android:name=".MessengerService"
    android:process=":messenger" >
    <intent-filter>
        <action android:name="com.rntest.service.messanger" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</service>
```

其他app只要通过对应的intent调用上面的服务就行。
```java
// 绑定并且收发信息

//向Service发送消息所用
private Messenger mService = null;
//接收Service的回复所用
private Messenger mMessenger = null;
private boolean mBound;

private ServiceConnection mConnection = new ServiceConnection() {
    public void onServiceConnected(ComponentName className, IBinder service) {
        //接收onBind()传回来的IBinder，并用它构造Messenger
        Log.d(TAG, "connected");
        mService = new Messenger(service);
        mBound = true;
    }

    public void onServiceDisconnected(ComponentName className) {
        Log.d(TAG, "disconnected");
        mService = null;
        mBound = false;
    }
};

@ReactMethod
public void messengerBind() {
    if(mMessenger == null) {
        // 在ReactContextBaseJavaModule类里创建handler，会报Can't create handler inside thread that has not called Looper.prepare()，所以把handler放到方法里创建。因为ReactContextBaseJavaModule是在一个临时的线程里创建的，创建之后这个线程就不存在了。而ReactMethod是在线程mqt_native_modules里运行的。
        Handler mHandler = new Handler() {
            public void handleMessage(android.os.Message msg) {
                switch (msg.what) {
                    case MSG_SAY_HELLO:
                        Log.i(TAG,"SimpleModule.java receive msg.what = " + msg.what);
                        break;
                    default:
                        break;
                }
            };
        };
        mMessenger = new Messenger(mHandler);
    }

    Log.d(TAG, "SimpleModule.java pid = " + Process.myPid());
    Intent intent = new Intent();
    intent.setAction("com.rntest.service.messanger");
    intent.setPackage("com.rntest");
    reactContext.bindService(intent, mConnection, BIND_AUTO_CREATE);
}

@ReactMethod
public void sendMessage() {
    if(mService == null) return;
    Message msg = Message.obtain(null, MSG_SAY_HELLO);
    msg.replyTo = mMessenger;    
    try {        //发送消息
        mService.send(msg);
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}
```

## BroadcastReceiver
```java
// 定义BroadcastReceiver类
public class MyBroadcastReceiver extends BroadcastReceiver{
    @Override
    public void onReceive(Context context, Intent intent) {}
}

// 下面这些代码都再Activity里
// 全局广播
// 动态注册
myReceiver = new MyBroadcastReceiver();
IntentFilter itFilter = new IntentFilter();
itFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
registerReceiver(myReceiver, itFilter);

// 取消广播
unregisterReceiver(myReceiver);

// 发送广播
sendBroadcast(new Intent("com.example.broadcasttest.MY_BROADCAST"));

// 本地广播，本地广播只能动态注册
// 本地广播注册
localBroadcastManager = LocalBroadcastManager.getInstance(this);
localReceiver = new MyBroadcastReceiver();
intentFilter = new IntentFilter();
intentFilter.addAction("com.example.broadcasttest.MY_BROADCAST");
localBroadcastManager.registerReceiver(localReceiver, intentFilter);

// 取消广播
localBroadcastManager.unregisterReceiver(localReceiver);

// 发送广播
Intent intent = new Intent("com.jay.mybcreceiver.LOGIN_OTHER");
localBroadcastManager.sendBroadcast(intent);
```
```xml
<!-- 静态注册。这里注册的是开机事件，是系统广播，所以要权限。 -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<receiver android:name=".BootCompleteReceiver">
    <intent-filter>
        <action android:name = "android.intent.cation.BOOT_COMPLETED">
    </intent-filter>
</receiver>

<!-- 这个是自定义广播 -->
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

## 权限
安卓6.0以上的权限控制。很多app在第一次打开的时候，就会询问各种权限，一开始就获取，免得日后用的时候再问，也是种方便的办法。下面是询问写入SD卡的权限的示例。

* https://developer.android.com/training/permissions/requesting.html

```java
@ReactMethod
public void fileTest() {
    // 因为这里是在@ReactMethod里的，所以有reactContext和getCurrentActivity()，具体参数的类型，请查看原文档
    if(ContextCompat.checkSelfPermission(reactContext, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED){
        // 没有授权，要询问
        if (ActivityCompat.shouldShowRequestPermissionRationale(getCurrentActivity(), Manifest.permission.WRITE_EXTERNAL_STORAGE)) {
            Toast.makeText(reactContext, "需要储存卡写入权限", Toast.LENGTH_SHORT).show();
            // 文档解释：shouldShowRequestPermissionRationale()。如果用户之前拒绝了该请求，该方法将返回 true；如果用户之前拒绝了某项权限并且选中了权限请求对话框中的不再询问选项，或者如果设备政策禁止该权限，该方法将返回 false。
        }
        // 如果用户拒绝了，并且勾选了不再询问，即使调用requestPermissions，也是不会再弹出授权询问框的。
        ActivityCompat.requestPermissions(getCurrentActivity(), new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 233);
    } else {
        // 已授权，正常执行操作
    }
}

// 在Activity里，还有一个回调，用于接收授权操作结果。如果准备执行对应操作时，才向用户申请相关操作的话，就要在这个回调里处理各种逻辑，因为授权是异步的，像上面的函数的写法，只能在授权结束后再调一次fileTest方法。所以应用第一次开启时便获取各种权限会比较方便。
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {}
```

react-native有[PermissionsAndroid](https://facebook.github.io/react-native/docs/permissionsandroid#requestmultiple)工具，可以在js端调用然后发起权限询问。**坑。**如果重写了`ReactActivity`的`onRequestPermissionsResult`，记得调用父类的方法，不然`PermissionsAndroid.request`返回的Promise就没下文了，因为react-native是在这个方法里通知到js那边的。骂了半天辣鸡react-native结果是自己的错，**重写方法记得调用父类方法**。