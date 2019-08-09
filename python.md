## python基础

dict的get和iteritems：
```py
d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
d['Thomas'] # 报错，这个跟js不同
d.get('Thomas', -1) # 返回-1，第二个参数是key不存在时的返回值
d.items() # 返回[(key, value)]
d.iteritems() # 跟上面差不多，但它返回迭代器
```

sorted函数和dict：
```py
import operator
d = {'Michael': 95, 'Bob': 75, 'Tracy': 85}
sorted(d.iteritems(), key=operator.itemgetter(1))
# 返回一个list，按照d的value排序，默认是从小到大排
# [('Bob', 75), ('Tracy', 85), ('Michael', 95)]
```

*args：
表示不定参数，以zip()函数为例，zip()接受不定参数：
```py
xyz = [(1, 4, 7), (2, 5, 8), (3, 6, 9)]
zip(*xyz) == zip((1, 4, 7), (2, 5, 8), (3, 6, 9)) # *号把xyz这个数组拆开成三个参数了
```

在函数调用中使用*list/tuple的方式表示将list/tuple分开，作为位置参数传递给对应函数（前提是对应函数支持不定个数的位置参数）

类变量：https://www.jianshu.com/p/3aca78a84def

```py
class Service(object):
    data = []
    def __init__(self, other_data):
        self.other_data = other_data

s1 = Service(['a', 'b'])
s2 = Service(['c', 'd'])
s1.data.append(1)

s1.data
## [1]
s2.data
## [1]

s2.data.append(2)

s1.data
## [1, 2]
s2.data
## [1, 2]
```

## 工具

### pip

#### pip源
-i https://mirrors.aliyun.com/pypi/simple/

### conda

#### Anaconda镜像

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```

### conda或pip逐个安装requirenments.txt里的依赖
```shell
while read requirement; do conda install --yes $requirement || pip install $requirement; done < requirements.txt
```
说明：

* read：读取一行到后面的参数(requirement)
* <：原本read应该是从标准输入(键盘)读取，用这个可以重定向到requirement.txt文件

## 各种库

### pyecharts
* python端生成简单的图表(html)：https://github.com/pyecharts/pyecharts

### Numpy
* 关于masked array：https://currents.soest.hawaii.edu/ocn_data_analysis/_static/masked_arrays.html

### matplotlib
基本操作
```py
plt.plot(x, y, color='red', label='label')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Some extension of Receiver operating characteristic to multi-class')
plt.legend(loc="lower right")
```

日期做横坐标
```py
from datetime import datetime

import matplotlib.dates as mdates
import matplotlib.pyplot as plt

# 生成横纵坐标信息
dates = ['01/02/1991', '01/03/1991', '01/04/1991']
xs = [datetime.strptime(d, '%m/%d/%Y').date() for d in dates]
ys = range(len(xs))
# 配置横坐标
plt.gca().xaxis.set_major_formatter(mdates.DateFormatter('%m/%d/%Y'))
plt.gca().xaxis.set_major_locator(mdates.DayLocator())
# Plot
plt.plot(xs, ys)
plt.gcf().autofmt_xdate()  # 自动旋转日期标记
plt.show()
```

### Tensorflow

#### GPU控制
选择使用哪些GPU
```py
import os
os.environ['CUDA_VISIBLE_DEVICES'] = '0' # 多个：'0,1'; 不用GPU：'-1'
```

控制显存
```py
import tensorflow as tf
config = tf.ConfigProto()
config.gpu_options.per_process_gpu_memory_fraction = 0.5  # 决定每个可见GPU应分配到的内存占总内存量的比例
config.gpu_options.allow_growth = True  # 根据运行时的需要来分配GPU内存，这个选项是优先的，设置这个就不用设置per_process_gpu_memory_fraction

# 另外，在keras，需要这样设置全局session
from keras.backend.tensorflow_backend import set_session
set_session(tf.Session(config=config))
```