## 组件

### Image
这个组件有点难用

https://www.hangge.com/blog/cache/detail_1542.html

* 可以把图片放到`android/app/src/main/res/drawable`目录下，打包，然后Image的uri直接填这个图片的名字(**必须不带文件后缀，带了反而出错**)。
* resizeMode可以作为组件的prop传入，也可以放在style里(可能是新版才支持)。
* 没有原生的placeholder功能，所以要自己代码实现。

## 原生模块

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