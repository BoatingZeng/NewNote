## 坑

### FLAG_ACTIVITY_NEW_TASK
在`ReactContextBaseJavaModule`的`ReactMethod`里打开新的Activity。

```java
// 只是单纯的打开新的Activity
Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
intent.addCategory(Intent.CATEGORY_OPENABLE);
intent.setType("image/*");
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK); // 因为不是在Activity里打开，所以要设置这个FLAG
reactContext.startActivity(intent);
```

```java
// 用startActivityForResult打开，这里如果设置FLAG_ACTIVITY_NEW_TASK，新打开的这个Activity立刻就会返回RESULT_CANCELED
int READ_REQUEST_CODE = 42;
Intent intent = new Intent(Intent.ACTION_OPEN_DOCUMENT);
intent.addCategory(Intent.CATEGORY_OPENABLE);
intent.setType("image/*");
reactContext.startActivityForResult(intent, READ_REQUEST_CODE, null);
```

## 组件

### Image
这个组件有点难用

https://www.hangge.com/blog/cache/detail_1542.html

* 可以把图片放到`android/app/src/main/res/drawable`目录下，打包，然后Image的uri直接填这个图片的名字(**必须不带文件后缀，带了反而出错**)。
* resizeMode可以作为组件的prop传入，也可以放在style里(可能是新版才支持)。
* 没有原生的placeholder功能，所以要自己代码实现。

## 原生模块
别人打包好的原生模块，npm或者yarn安装好，记录到package.json，gradle sync的时候，会读取package.json，把原生模块的包添加到PackageList.java里。这个是新版(0.60后)react-native的功能。[详情](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md)

### 注意
* 在js端调用@ReactMethod方法，参数类型和数量要和JAVA那端一致。

### 简单实例
下面是构造一个原生模块的最简单代码。

```java
// SimpleModule.java
// 这是模块本身
package com.rntest2;

import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

public class SimpleModule extends ReactContextBaseJavaModule {
  private static ReactApplicationContext reactContext;

  SimpleModule(ReactApplicationContext context) {
    super(context);
    reactContext = context;
  }

  @Override
  public String getName() {
    return "SimpleModule"; // js那边通过这个名字获取这个模块
  }

  // 供js端调用的方法，返回值类型必须是void，js和java这边的通讯必须是异步
  @ReactMethod
  public void doNothing() {}
}
```

```java
// SimplePackage.java
// 模块要包在package里输出到js那边
package com.rntest2;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class SimplePackage implements ReactPackage {
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new SimpleModule(reactContext)); // 模块放到这里
        return modules;
    }
}
```

```java
// MainApplication.java
// 把上面的package注册到应用里
@Override
protected List<ReactPackage> getPackages() {
  @SuppressWarnings("UnnecessaryLocalVariable")
  List<ReactPackage> packages = new PackageList(this).getPackages();
  packages.add(new SimplePackage());
  return packages;
}
```

```js
// 调用
import {NativeModules} from 'react-native';
NativeModules.SimpleModule.doNothing();
```

### JAVA和JS通讯
因为原生模块的方法，都没有返回值，所以两边需要异步通讯。

```java
// 自定义模块类
public class ExampleModule extends ReactContextBaseJavaModule {
    private static ReactApplicationContext reactContext;

    ExampleModule(ReactApplicationContext context) {
        super(context);
        reactContext = context;
    }

    @Override
    public String getName() {
        return "ExampleModule";
    }

    // 通过callback通讯
    @ReactMethod
    public void testCallback(int flag, Callback cb) {
        try {
            if(flag == 1) {
                throw new NullPointerException("test Callback error");
            } else {
                cb.invoke(null, "Callback success");
            }
        } catch(NullPointerException e) {
            cb.invoke(e.getMessage());
        }
    }

    // 通过Promise通讯
    @ReactMethod
    public void testPromise(int flag, Promise promise) {
        try {
            if(flag == 1) {
                throw new NullPointerException("test Promise error 1");
            } else {
                promise.resolve("Promise success");
            }
        } catch(NullPointerException e) {
            promise.reject("test Promise error 2", e);
        }
    }

    // 通过Event通讯
    @ReactMethod
    public void testEvent() {
        reactContext
            .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
            .emit("MyEvent", "event data");
    }

    // 开启一个线程，执行耗时任务，执行完后通过callback返回结果
    class MyThread implements Runnable {
        private Callback cb;

        public MyThread(Callback cb) {
            this.cb = cb;
        }

        @Override
        public void run() {
            try {
                Thread.sleep(3000);
                this.cb.invoke(null, "result from thread");
            } catch (InterruptedException e) {
                this.cb.invoke(e.getMessage());
            }
        }
    }

    @ReactMethod
    public void testThread(Callback cb) {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        executorService.submit(new MyThread(cb));
        executorService.shutdown();
    }
}
```

```js
// 在js端的调用

// 通过callback
ToastExample
  .testCallback(1,
    (err, res) => {
      if (err) {
        console.error(err);
      } else {
        console.log(res);
      }
    });

// 通过Promise
async function testPromise() {
  let res;
  try {
    res = await ToastExample.testPromise(2);
    console.log(res);
  } catch (e) {
    console.log(res);
    console.error(e);
  }
}

// 通过Event
// 注册event的listener，类内部触发MyEvent，自然就会调用listener
const eventEmitter = new NativeEventEmitter(ToastExample);
eventEmitter.addListener('MyEvent', (event) => {
    console.log(event);
});

// 因为类内部是通过testEvent方法触发MyEvent事件的，所以主动调用它
ToastExample.testEvent();
```

### ReactMethod的线程问
正如react-native官方文档所说，如果要执行耗时任务，应该开启一个工作线程去执行。下面做一个简单测试。

创建两个ReactContextBaseJavaModule，包含以下方法。
```java
@ReactMethod
public void threadInfo() {
    Log.d("threadTest", "threadInfo myTid = " + Process.myTid());
    Log.d("threadTest", "threadInfo getId = " + Thread.currentThread().getId());
    Log.d("threadTest", "threadInfo getName = " + Thread.currentThread().getName()); // 线程名：mqt_native_modules
}

// 这两个方法是一样的，都是执行耗时任务
@ReactMethod
public void longMethod1(){
    Log.w("threadTest", "longMethod1 start");
    Log.d("threadTest", "longMethod1 myTid = " + Process.myTid());
    Log.d("threadTest", "longMethod1 getId = " + Thread.currentThread().getId());
    for(int i=0; i<5; i++){
        Log.d("threadTest", "longMethod1");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    Log.w("threadTest", "longMethod1 end");
}

@ReactMethod
public void longMethod2(){
    Log.w("threadTest", "longMethod2 start");
    Log.d("threadTest", "longMethod2 myTid = " + Process.myTid());
    Log.d("threadTest", "longMethod2 getId = " + Thread.currentThread().getId());
    for(int i=0; i<5; i++){
        Log.d("threadTest", "longMethod2");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    Log.w("threadTest", "longMethod2 end");
}
```

调用这些方法，有如下结论：

* 不同的ReactContextBaseJavaModule的(@ReactMethod)方法，是在同一个线程(`mqt_native_modules`)里运行的
* 在一个耗时任务运行时，调用其他方法，后调用的方法会等到前面的方法返回才执行
* 用按钮调用耗时任务，会发现，UI上，要等到耗时任务执行完毕(返回)，按钮的按下动画才会触发
* `adb shell ps -T <进程ID>`查看线程信息，可以找到线程名为`mqt_native_modules`的线程。而且它不是主线程。主线程名在这个命令里看，是包名，在程序里打印，则为main。

其他参考
* https://zhuanlan.zhihu.com/p/45836822

## 原生UI组件
* https://hackernoon.com/react-native-bridge-for-android-and-ios-ui-component-782cb4c0217d

按照上面的教程可以实现简单的原生UI组件，因为教程代码不好看，下面贴一下具体代码。

```java
// BulbView.java
// 组件本身
package com.rntest2;

import android.content.Context;
import android.graphics.Color;
import android.util.AttributeSet;
import android.view.View;
import android.widget.Button;

import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.uimanager.events.RCTEventEmitter;

public class BulbView extends Button {
    public Boolean isOn = false;

    public BulbView(Context context) {
        super(context);
        this.setTextColor(Color.BLUE);
        this.setText("This button is created from JAVA code");
        this.setOnClickListener(changeStatusListener);
        updateButton();
    }

    public BulbView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public BulbView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    public void setIsOn (Boolean initialBulbStatus){
        isOn = initialBulbStatus;
        updateButton();
    }

    private void updateButton() {
        // 这部分用于和js通讯，把event发到js那边
        WritableMap event = Arguments.createMap();
        event.putBoolean("isOn", isOn);
        ReactContext reactContext = (ReactContext)getContext();
        reactContext.getJSModule(RCTEventEmitter.class).receiveEvent(
                getId(),
                "statusChange",
                event);

        if (isOn) {
            setBackgroundColor(Color.YELLOW);
            setText("Switch OFF");
        } else {
            setBackgroundColor(Color.BLACK);
            setText("Switch ON");
        }
    }

    private OnClickListener changeStatusListener = new OnClickListener() {
        public void onClick(View v) {
            isOn = !isOn;
            updateButton();
        }
    };
}
```

```java
// BulbManager.java
// SimpleViewManager用来实例化和更新app里的views
package com.rntest2;

import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.annotations.ReactProp;

import com.facebook.react.common.MapBuilder;
import java.util.Map;

public class BulbManager extends SimpleViewManager<BulbView> {

    @Override
    public String getName() {
        return "Bulb";
    }

    @Override
    protected BulbView createViewInstance(ThemedReactContext reactContext) {
        return new BulbView(reactContext);
    }

    @ReactProp(name="isOn")
    public void setBulbStatus(BulbView bulbView, Boolean isOn) {
        bulbView.setIsOn(isOn);
    }

    @Override
    public Map getExportedCustomBubblingEventTypeConstants() {
        return MapBuilder.builder()
            .put(
                "statusChange",
                MapBuilder.of(
                    "phasedRegistrationNames",
                    MapBuilder.of("bubbled", "onStatusChange")))
            .build();
    }
}
```

```java
// BulbPackage.java
// 创建一个包，和NativeModule不同的是，UI组件是放到ViewManager列表里的
package com.rntest2;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class BulbPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.<ViewManager>singletonList(new BulbManager()); // 组件(manager)放到这里
    }
}
```

```java
// MainApplication.java
// 和NativeModule一样，要在MainApplication的ReactNativeHost的getPackages方法里，把package添加进去。教程是旧版的，下面代码是新版(0.61.4)的写法。
@Override
protected List<ReactPackage> getPackages() {
  @SuppressWarnings("UnnecessaryLocalVariable")
  List<ReactPackage> packages = new PackageList(this).getPackages();
  packages.add(new BulbPackage());
  return packages;
}
```

```js
// js那边调用上面的组件
import {requireNativeComponent} from 'react-native';
const Bulb = requireNativeComponent('Bulb');

<Bulb isOn={true} onStatusChange={e => console.log(e.nativeEvent)}/>
```

## Headless JS

### 简单实例
经测试(测试是在debug模式下进行)，下面这个例子，只有app开着的时候，service会运作(打印)，待机或者把app丢到后台都不会运作。待机后解锁，进入app，它会接着之前的状态运行。如果把app关了，再进去，也不会接着运作。

```java
// MyTaskService.java
package com.rntest2;

import android.content.Intent;
import android.os.Bundle;
import com.facebook.react.HeadlessJsTaskService;
import com.facebook.react.bridge.Arguments;
import com.facebook.react.jstasks.HeadlessJsTaskConfig;
import javax.annotation.Nullable;

public class MyTaskService extends HeadlessJsTaskService {
  @Override
  protected @Nullable HeadlessJsTaskConfig getTaskConfig(Intent intent) {
    Bundle extras = intent.getExtras();
    if (extras != null) {
      return new HeadlessJsTaskConfig(
          "SomeTaskName",
          Arguments.fromBundle(extras),
          5000, // timeout for the task
          true // 这里需要让service在前台运行，因为例子里是用按钮主动触发的
        );
    }
    return null;
  }
}
```

```java
// 找个原生模块方法，主动触发这个service
@ReactMethod
public void startService() {
    Intent service = new Intent(reactContext, MyTaskService.class);
    Bundle bundle = new Bundle();

    bundle.putString("foo", "bar"); // 传入参数，{"foo": "bar"}
    service.putExtras(bundle);

    reactContext.startService(service);
}
```

```xml
<!-- AndroidManifest.xml 里添加，service是application的子元素 -->
<uses-permission android:name="android.permission.WAKE_LOCK" />

<service android:name="com.rntest2.MyTaskService" />
```

```js
// HeadLess.js
// 这个就是要运行的js代码
module.exports = async (taskData) => {
  console.log('headless start');
  function sleep(){
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log('sleep');
        console.log(taskData); // 这里打印{"foo": "bar"}
        resolve();
      }, 5000);
    });
  }
  for (let i = 0; i < 5; i++) {
    await sleep();
  }
  console.log('headless end');
};
```

```js
// index.js
// react native 程序的entry
import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);
AppRegistry.registerHeadlessTask('SomeTaskName', () => require('./Headless'));
```

## 第三方库

### react-native-sqlite-storage
文档不清晰(根本找不到文档)。下面列出用例。

```js
// 要在react-native环境下用，不过这里只写出主要代码
import SQLiteStorage from 'react-native-sqlite-storage';
SQLiteStorage.DEBUG(true);

var database_name = 'test.db'; //数据库文件，看react-native-sqlite-storage的openDatabase这个函数，这里也就这个参数是用了的
var database_version = '1.0';
var database_displayname = 'MySQLite';
var database_size = -1;
var db = null;

// 打开数据库
db = SQLiteStorage.openDatabase(
                database_name,
                database_version,
                database_displayname,
                database_size,
                ()=>{
                    this._successCB('open');
                },
                (err)=>{
                    this._errorCB('open',err);
                });

// 执行操作，可以用db.transaction或者直接db.executeSql

// 创建用户表
// 注意executeSql和transaction的成功回调和错误回调的顺序
db.transaction((tx)=> {
    tx.executeSql('CREATE TABLE IF NOT EXISTS USER(' +
        'id INTEGER PRIMARY KEY  AUTOINCREMENT,' +
        'name varchar,' +
        'age VARCHAR,' +
        'sex VARCHAR,' +
        'phone VARCHAR,' +
        'email VARCHAR,' +
        'qq VARCHAR)'
        , [], ()=> {
            // executeSql的成功回调
        }, (err)=> {
            // executeSql的错误回调
        });
}, (err)=> {
    // transaction的错误回调
}, ()=> {
    // transaction的成功回调
});

// 插入数据
// 这里的userData是个数组，下面是在一个事务里插入多条记录
let len = userData.length;
db.transaction((tx)=>{
    for (let i = 0; i < len; i++){
        let user = userData[i];
        let {name, age, sex, phone, email, qq} = user;
        let sql = 'INSERT INTO user(name,age,sex,phone,email,qq)' + 'values(?,?,?,?,?,?)';
        tx.executeSql(sql, [name,age,sex,phone,email,qq]); // 这里executeSql不设置回调也行
    }
}, (err)=>{
    // transaction的错误回调
}, ()=>{
    // transaction的成功回调
});

// 查询数据
db.executeSql('select * from user', [], (results) => {
    var len = results.rows.length;
    for (let i = 0; i < len; i++){
        var u = results.rows.item(i); // item是个函数
        console.log(u);
    }
}, (err) => {
    console.error(err);
});

// 关闭数据库
db.close();
```