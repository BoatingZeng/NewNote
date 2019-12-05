## Service

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