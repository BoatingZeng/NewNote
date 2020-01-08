## 安装
安装nodejs
```
curl --silent --location https://rpm.nodesource.com/setup_10.x | bash -
yum install -y nodejs
```
安装yarn
```
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
yum install yarn
```

## pm2

传入参数

```
node --expose-gc bin/www arg1 arg2 arg3
```

```
pm2 start bin/www --node-args="--expose-gc" -- arg1 arg2 arg3
```

node参数放到--node-args里，普通参数放到--后面

## log4js

和pm2使用时，要`pm2 install pm2-intercom`，config里设置`pm2`为`true`，详见https://nomiddlename.github.io/log4js-node/faq.html

## koa

### 错误处理和404处理

和express相反，koa的这两个处理中间件要放在最开头。404和错误处理谁先谁后关系不大，因为这两个中间件都是要保证不能出错(或许错误处理应该放在404前面，以便捕获404中间件可能出现的错误)。后面的中间件可以通过`ctx.throw(555, 'my error');`抛出有状态码的错误，以便错误中间件识别。koa的机制是，如果中间件没有改变状态码，`ctx.response.status`就一直是404，所以404中间件就是利用这个特性来判断后面的路由是否命中。

```js
app.use(async (ctx, next) => {
    try{
        await next();
    } catch(e) {
        ctx.response.status = e.status || e.statusCode || 500;
        console.log('error', e);
        console.log('error status', e.status);
        console.log('status', ctx.response.status);
        ctx.response.body = 'my error page';
    }
});

app.use(async (ctx, next) => {
    await next();
    console.log('404 mid', ctx.response.status);
    if(parseInt(ctx.response.status) === 404) ctx.response.body = 'my 404';
});
```

## sequelize
这个库的坑很多，记得保持更新。旧版本遇到一个bug，设置unique就给你创建两个一样的唯一索引。

### associations用例
**实际上一般很少使用外键，所以在数据库里应该把外键约束去掉，模型里保留这个关联。不过sequelize把关联和约束混在一起，让人有点困惑，只看模型的话，可能会让人以为这里建立了外键约束，但实际上我们只是想定义关联。**

* https://itbilu.com/nodejs/npm/EkWJSmmFf.html
* https://lorenstewart.me/2016/09/12/sequelize-table-associations-joins/
* https://github.com/josie11/Sequelize-Association-Example
* https://grokonez.com/node-js/sequelize-one-to-many-association-nodejs-express-mysql

下面的例子都用这样的设定，主要是不带timestamps和version，并且freezeTableName固定表名
```js
const Sequelize = require('sequelize');

// 创建 sequelize 实例
const sequelize = new Sequelize('test', 'root', 'mysqlboating', {
  host: 'localhost',
  port: 3306,
  dialect: 'mysql',

  define: {
    timestamps: false,
    version: false,
    freezeTableName: true
  }
});
```

sequelize外键指向的键，最好使用主键，因为有几个关联关系都只支持指向主键（原因是设置里不支持targetKey和sourceKey），比如belongsToMany。

**belongsTo**和**hasMany**。下面的例子理解为，一个company有很多个user，但是user只属于一个company。

```js
// 定义User模型
var User = sequelize.define('user', {
	id:{type: Sequelize.BIGINT(11), autoIncrement:true, primaryKey : true },
	name: { type: Sequelize.STRING },
	sex: { type: Sequelize.INTEGER, allowNull: false, defaultValue: 0 },
	isManager: { type: Sequelize.BOOLEAN, field: 'is_manager', allowNull: false, defaultValue: false }
});

// 定义Company模型
var Company = sequelize.define('company', {
	id:{ type:Sequelize.BIGINT(11), autoIncrement:true, primaryKey : true },
	name: { type: Sequelize.STRING, unique: true, allowNull: false }
});

// 定义User-Company关联关系
User.belongsTo(Company, {as: 'company', foreignKey: 'company_name', targetKey: 'name'});
Company.hasMany(User, { as: 'users', foreignKey:'company_name', sourceKey: 'name'});

// 如果上面有定义关联，这样直接使用关联
var include = [{
	model: Company,
	as: 'company'
}];

// 如果上面没有定义关联，临时定义关联
var include = [{
	association: User.belongsTo(Company, {foreignKey:'company_name', as: 'company', targetKey: 'name'})
}];

User.findOne({include:include}).then((result) => {
	console.log(result.name + ' 是 '+result.company.name+' 的员工');
}).catch((err) => {
	console.error(err);
});
```

foreignKey是建立外键的那个表的字段，这个字段不需要在define里创建，定义association时会给你创建。在belongsTo和hasMany里，targetKey或者sourceKey是指被指向的表里的字段，这里指向的就是company表的name字段。如果不定义，默认就是主键。

关于hasOne，目前版本4.42.0，它没有sourceKey选项，所以hasOne只能用主键，github上说5.0.0版本后支持sourceKey选项。

同步后会得到下面的表

```sql
CREATE TABLE `company` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `user` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `sex` int(11) NOT NULL DEFAULT '0',
  `is_manager` tinyint(1) NOT NULL DEFAULT '0',
  `company_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `company_name` (`company_name`),
  CONSTRAINT `user_ibfk_1` FOREIGN KEY (`company_name`) REFERENCES `company` (`name`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**belongsToMany**的例子，下面的例子表示一个order包含很多个product，同时一个product也会出现在不同order里。

```js
var Product = sequelize.define('product', {
  id: { type: Sequelize.INTEGER, autoIncrement:true, primaryKey : true },
  name: Sequelize.STRING
});

var Order = sequelize.define('order', {
  id: { type: Sequelize.INTEGER, autoIncrement:true, primaryKey : true }
});

// 这个关联表的模型最好还是自己手动定义，例如要在这里添加主键，或者其他信息
var OrderProduct = sequelize.define('order_product', {
  id: { type: Sequelize.INTEGER, autoIncrement:true, primaryKey : true },
  quantity: Sequelize.INTEGER
});

Order.belongsToMany(Product, {
  through: model: 'order_product',
  foreignKey: 'order_id',
  // otherKey: 'product_id', // 这个一般不用写，Product.belongsToMany里的foreignKey和这个等价的
  as: 'product' // 这里填的是Product这个目标模型的别名!!!保险起见用下面那种as的定义形式。
});

Product.belongsToMany(Order, {
  through: model: 'order_product',
  foreignKey: 'product_id',
  // otherKey: 'order_id', // 这个一般不用写，Order.belongsToMany里的foreignKey和这个等价的
  as: {
    plural: 'order', // 这里填的是Order这个目标模型的别名!!!
    singular: 'order' // 因为不习惯复数形式，所以干脆显式定义单复数形式一样
  }
});
```

同步后得到下面的表

```sql
CREATE TABLE `order` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `product` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `order_product` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `quantity` int(11) DEFAULT NULL,
  `order_id` int(11) DEFAULT NULL,
  `product_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `order_product_product_id_order_id_unique` (`order_id`,`product_id`),
  KEY `product_id` (`product_id`),
  CONSTRAINT `order_product_ibfk_1` FOREIGN KEY (`order_id`) REFERENCES `order` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
  CONSTRAINT `order_product_ibfk_2` FOREIGN KEY (`product_id`) REFERENCES `product` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

往关联表添加关联的用例
```js
async function main(){
  await Order.create({}); // 创建一个order，这里也可以用include
  await Product.create({name: '代码创建'}); // 创建一个product
  let order = await Order.findOne({include: {model: Product, as: 'product'}}); // 查一个order，并且把它的product查出来，注意这里as和上面定义的一致
  await order.addProduct(1, {through: {quantity: 10}}); // 往刚才查出来的order上面添加product，addProduct是由关联创建的函数，参数里的1是指添加的product的id，它会先select检查这个order有没有这个product，没有才会添加
}
```

### transaction

配合async函数就变得很直观了

```js
// 这个函数模拟一个异步错误
function throw_err(){
  return new Promise(function(resolve, reject){
    setTimeout(function(){
      reject(new Error('fake error'));
    }, 1000);
  });
}

async function query_in_transaction(){
  let transaction
  try {
    transaction = await sequelize.transaction(); // 默认就是{ autocommit: false }
    await User.create({name:'张三', sex:1, isManager: true}, {transaction});
    await User.create({name:'李三', sex:1}, {transaction});
    // await throw_err(); // 这里可以测试中途出错rollback
    await transaction.commit();
  } catch(e){
    console.log(e);
    await transaction.rollback();
  }
}

query_in_transaction();
```

## egg

### 日志的问题
* https://www.jianshu.com/p/919f3e288e5a

## 子进程
exec和fork等，其实都是用spawn实现的，所以实际上就spwan一个方法。

把fork的实现抽出来，下面的`stdioStringToArray`和`fork`都是从nodejs源码拷贝出来的。p.js是父进程，c.js是子进程。
```js
// p.js
const { spawn } = require( 'child_process');
const util = require('util');

function stdioStringToArray(option) {
  switch (option) {
    case 'ignore':
    case 'pipe':
    case 'inherit':
      return [option, option, option, 'ipc'];
    default:
      throw new ERR_INVALID_OPT_VALUE('stdio', option);
  }
}

function fork(modulePath /* , args, options */) {

  // Get options and args arguments.
  var execArgv;
  var options = {};
  var args = [];
  var pos = 1;
  if (pos < arguments.length && Array.isArray(arguments[pos])) {
    args = arguments[pos++];
  }

  if (pos < arguments.length &&
      (arguments[pos] === undefined || arguments[pos] === null)) {
    pos++;
  }

  if (pos < arguments.length && arguments[pos] != null) {
    if (typeof arguments[pos] !== 'object') {
      throw new ERR_INVALID_ARG_VALUE(`arguments[${pos}]`, arguments[pos]);
    }

    options = util._extend({}, arguments[pos++]);
  }

  // Prepare arguments for fork:
  execArgv = options.execArgv || process.execArgv;

  if (execArgv === process.execArgv && process._eval != null) {
    const index = execArgv.lastIndexOf(process._eval);
    if (index > 0) {
      // Remove the -e switch to avoid fork bombing ourselves.
      execArgv = execArgv.slice();
      execArgv.splice(index - 1, 2);
    }
  }

  args = execArgv.concat([modulePath], args);

  if (typeof options.stdio === 'string') {
    options.stdio = stdioStringToArray(options.stdio);
  } else if (!Array.isArray(options.stdio)) {
    // Use a separate fd=3 for the IPC channel. Inherit stdin, stdout,
    // and stderr from the parent if silent isn't set.
    options.stdio = options.silent ? stdioStringToArray('pipe') :
      stdioStringToArray('inherit');
  } else if (options.stdio.indexOf('ipc') === -1) {
    throw new ERR_CHILD_PROCESS_IPC_REQUIRED('options.stdio');
  }

  options.execPath = options.execPath || process.execPath;
  options.shell = false;
  console.log('看看最终传给spawn的参数');
  console.log(options);
  // { stdio: [ 'pipe', 'pipe', 'pipe', 'ipc' ], ipc表示在父子进程间设置IPC信道
  // execPath: 'F:\\Program Files\\node-v10.11.0-win-x64\\node.exe',
  // shell: false }
  console.log(args); // [ './c.js' ]
  return spawn(options.execPath, args, options);
};

// 这样和下面执行fork是一样的
// const options = { 
//   stdio: [ 'pipe', 'pipe', 'pipe', 'ipc' ],
//   execPath: 'F:\\Program Files\\node-v10.11.0-win-x64\\node.exe',
//   shell: false 
// };
// const n = spawn('F:\\Program Files\\node-v10.11.0-win-x64\\node.exe', ['./c.js'], options);

const n = fork('./c.js', {stdio: 'pipe'});
// 因为设置了ipc信道，所以父子进程可以通过message事件和send方法通讯
n.on( 'message', ( m) => {
  console.log( 'PARENT got message:',  m);
});
n.send({ hello:  'world' });
// 因为上面设置了stdio是pipe，所以子进程打印的东西会传过来
n.stdout.on('data', (data)=>{
  // 这个data是个Buffer
  console.log('stdout：'+data);
});

// c.js
process.on( 'message', ( m) => {
  console.log( 'CHILD got message:',  m);
});
process.send({ foo:  'bar' });
```

## cluster
这个主题其实是跟子进程主题相关的。不过因为nodejs有一个cluster模块，所以单独出来。

参考：https://www.cnblogs.com/accordion/p/7207740.html

实际监听端口的其实是主进程，子进程的listen不是真正的监听端口。具体过程如下：

1. worker调用server的listen，告诉master要监听端口并建立调度(默认是Round-Robin)
2. master的server监听端口(server内部有一个叫handle的对象，它是调用底层创建的，是有真正的listen方法的)，并且把worker加进调度里，并且告诉worker完成这步了
3. worker的server会建立一个假的handle(它有假的listen方法)
4. 当有连接来到master时，master通过IPC发到worker处理

第4步的具体细节，要看看`process`对象的`send`方法`process.send(message[, sendHandle[, options]][, callback])`，这里注意第二个参数`sendHandle`，它是`<net.Server>|<net.Socket>`。另外看`round_robin_handle.js`的代码(在下面贴出)的`this.handle.onconnection`部分，`onconnection`最终收到的`handle`，会经过IPC传到子进程，所以子进程的`message`事件里收到的`sendHandle`对象，就是这个底层传过来的`handle`，子进程可以用它处理这个连接。~~之前有一个错误的认知。以为master和worker只是通过message通讯，master把要处理的东西通过message发到worker，然后worker把处理结果通过message发回给master，master再发会给client。这是错的！~~

下面在注释里简单说明Round-Robin的过程。按代码顺序看就是过程的顺序了。

```js
// lib\internal\cluster\round_robin_handle.js
function RoundRobinHandle(key, address, port, addressType, fd) {
  this.key = key;
  this.all = new Map();
  this.free = [];
  this.handles = [];
  this.handle = null;
  this.server = net.createServer(assert.fail);

  if (fd >= 0)
    this.server.listen({ fd });
  else if (port >= 0)
    this.server.listen(port, address);
  else
    this.server.listen(address);  // UNIX socket path.

  this.server.once('listening', () => {
    this.handle = this.server._handle;
    // 这里的onconnection是底层代码的回调函数，这里的handle代表了这个连接
    // 有连接来就调distribute方法并且把handle传进去
    this.handle.onconnection = (err, handle) => this.distribute(err, handle);
    this.server._handle = null;
    this.server = null;
  });
}
// 每次调distribute会把一个handle放到待处理队列，并且让一个空闲worker进入到工作状态
RoundRobinHandle.prototype.distribute = function(err, handle) {
  // 无论怎样，先把这个handle放发哦待处理队列
  this.handles.push(handle);
  const worker = this.free.shift(); // 拿队头的进程
  // 如果有空闲worker，就让它工作
  if (worker)
    this.handoff(worker);
};
// 每handoff一次分派一个handle给一个worker处理
RoundRobinHandle.prototype.handoff = function(worker) {
  if (this.all.has(worker.id) === false) {
    return;  // Worker is closing (or has closed) the server.
  }
  // 从队列里取一个要处理的
  const handle = this.handles.shift();

  if (handle === undefined) {
    this.free.push(worker);  // 如果待处理队列空，那这个worker就先回到空闲状态
    return;
  }

  const message = { act: 'newconn', key: this.key };

  // master把这个handle发给worker处理，worker收到就会再回一个消息，不管这个连接是否处理完的
  sendHelper(worker.process, message, handle, (reply) => {
    if (reply.accepted)
      // 反正worker接收了，master就不管这个handle了
      handle.close();
    else
      // 如果worker没回复说接收了，就把这个handle放回到待处理队列。事实上，只要worker活着，worker一收到newconn信息，就会立刻回复已接收的
      this.distribute(0, handle);  // Worker is shutting down. Send to another.

    this.handoff(worker); // 刚才这个worker刚处理完，又会立刻给它下一个请求，所以handles队列为空之前，这个worker都不会闲下来的
  });
};
```

Round-Robin是不适合用在要保持连接的情景的，比如使用socket.io这个库的时候。详细参考：https://www.cnblogs.com/accordion/p/6930152.html

这篇文章指出nodejs的IPC通讯性能有问题，用node10在win10下测试过(代码在下面贴出)，确实用IPC的延迟会一直增长，而用socket的不会。文章链接：https://zhuanlan.zhihu.com/p/27069865

```js
// master.js
const child_process = require('child_process');

let child = child_process.fork('./child.js');
let data = Array(1024 * 1024).fill('0').join('');

setInterval(() => {
  let i = 100;
  while(i--) child.send(`${data}|${Date.now()}`);
}, 1000);

// child.js
let i = 0;

process.on('message', (str) => {
  let now = Date.now();
  let [data, time] = str.split('|')
  console.log(i++, now - Number(time));
});

// server.js
const axon = require('axon'); // 第三方模块
const sock = axon.socket('push');

sock.bind(3000);
console.log('push server started');
let data = Array(1024 * 1024).fill('0').join('');

setInterval(function(){
  let i = 100;
  while(i--) sock.send(`${data}|${Date.now()}`);
}, 1000);

// client.js
const axon = require('axon');
const sock = axon.socket('pull');

sock.connect(3000);

let i = 0;
sock.on('message', function(msg){
  let now = Date.now();
  let [data, time] = msg.split('|')
  console.log(i++, now - Number(time));
});
```

## 关于require
`require('a.js')`到底发生了什么？

很明显，解析一个js文件的字符并执行，然后导出一个模块对象，这些操作并非单纯的js代码就可以完成的，它肯定依赖了nodejs底层的某些功能。下面点到即止地从源码上走一下require的过程。(因为不懂nodejs的底层，所以c语言写的那部分，视为大爆炸前的事，归上帝管)

代码是node10.x的源码，是`lib\internal\modules\cjs`目录下的`loader.js`和`helper.js`文件。代码有删减且加了中文注释。被删减的部分有注释提示。

loader.js文件里有Module这个类，它的实例就代表模块本身。

模块里的require函数，其实是Module类的require方法，源码如下。它并非全局函数，之后有详细解释。
```js
Module.prototype.require = function(id) {
  validateString(id, 'id');
  if (id === '') {
    throw new ERR_INVALID_ARG_VALUE('id', id,
                                    'must be a non-empty string');
  }
  return Module._load(id, this, /* isMain */ false);
};
```

可以看到require主要是调用`_load`函数，下面再看看`_load`函数。

```js
Module._load = function(request, parent, isMain) {
  if (parent) {
    debug('Module._load REQUEST %s parent: %s', request, parent.id);
  }
  // 先不管它怎么找到目标文件。因为我们探讨的是到底怎么处理被加载的文件，所以之后盯紧这个filename
  var filename = Module._resolveFilename(request, parent, isMain);
  // 也不管模块已经被加载过，在cache里的情况
  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    updateChildren(parent, cachedModule, true);
    return cachedModule.exports;
  }
  // 也不管加载的是nodejs内置模块的情况
  if (NativeModule.nonInternalExists(filename)) {
    debug('load native module %s', request);
    return NativeModule.require(filename);
  }

  // Don't call updateChildren(), Module constructor already does.
  var module = new Module(filename, parent);

  if (isMain) {
    process.mainModule = module;
    module.id = '.';
  }

  Module._cache[filename] = module;

  tryModuleLoad(module, filename);

  return module.exports;
};

// 上面用到的tryModuleLoad函数，在loader.js里
function tryModuleLoad(module, filename) {
  var threw = true;
  try {
    module.load(filename);
    threw = false;
  } finally {
    if (threw) {
      delete Module._cache[filename];
    }
  }
}
```

这里new了一个Module出来，代表的是要被加载的模块。然后tryModuleLoad里，这个要被加载的模块调用load函数去读取这个文件。所以接下来看看load函数(注意这个是没有下划线的)。

```js
Module.prototype.load = function(filename) {
  debug('load %j for module %j', filename, this.id);

  assert(!this.loaded);
  this.filename = filename;
  this.paths = Module._nodeModulePaths(path.dirname(filename));
  // 这个extension就是文件扩展名，后面会根据不同扩展名调用不同方法，上面讨论的是.js扩展名，所以这里最后调Module._extensions['.js']
  var extension = findLongestRegisteredExtension(filename);
  Module._extensions[extension](this, filename);
  this.loaded = true;

  // 后面的代码跟experimentalModules有关，不管它
  // 这里删减了代码...
};

// 上面Module._extensions[extension](this, filename);实际调用的方法
Module._extensions['.js'] = function(module, filename) {
  // 这个方法终于是读取文件内容了
  var content = fs.readFileSync(filename, 'utf8');
  module._compile(stripBOM(content), filename);
};
```

对于.js后缀文件，load函数最终调用`Module._extensions['.js']`。读取文件内容后，处理内容的函数是`_compile`。所以接下来是`_compile`函数。

```js
Module.prototype._compile = function(content, filename) {

  content = stripShebang(content);

  let compiledWrapper;
  if (patched) {
    // 不清楚patched什么含义，所以不管这个分支
    const wrapper = Module.wrap(content);
    compiledWrapper = vm.runInThisContext(wrapper, {
      filename,
      lineOffset: 0,
      displayErrors: true,
      importModuleDynamically: experimentalModules ? async (specifier) => {
        if (asyncESM === undefined) lazyLoadESM();
        const loader = await asyncESM.loaderPromise;
        return loader.import(specifier, normalizeReferrerURL(filename));
      } : undefined,
    });
  } else {
    // compileFunction来自loader.js文件开头的const { compileFunction } = internalBinding('contextify'); 是底层函数，所以没法进一步分析
    // 不过我们知道返回的compiledWrapper也是一个函数，看看后面实际对compiledWrapper的调用，那里再猜测这个compiledWrapper是一个怎样的函数
    compiledWrapper = compileFunction(
      content,
      filename,
      0,
      0,
      undefined,
      false,
      undefined,
      [],
      [
        'exports',
        'require',
        'module',
        '__filename',
        '__dirname',
      ]
    );
    // 这里的代码跟experimentalModules有关，不管它
    // 这里删减了代码...
  }

  var inspectorWrapper = null;
  // 这里的代码跟inspector有关，不管它
  // 这里删减了代码...

  var dirname = path.dirname(filename);

  // makeRequireFunction来自helper.js，它就是构造模块里使用的require函数的，看看下面makeRequireFunction的代码
  var require = makeRequireFunction(this);
  var depth = requireDepth;
  if (depth === 0) stat.cache = new Map();
  var result;
  if (inspectorWrapper) {
    result = inspectorWrapper(compiledWrapper, this.exports, this.exports,
                              require, this, filename, dirname);
  } else {
    // 这里解释了为什么模块文件内部有exports、module、__dirname、__filename、require这些变量
    // 其实它们都不是全局变量，而是在处理模块文件的时候用compiledWrapper包进去的
    result = compiledWrapper.call(this.exports, this.exports, require, this,
                                  filename, dirname);
  }
  if (depth === 0) stat.cache = null;
  return result;
};

// 上面用到的makeRequireFunction，在helper.js里
function makeRequireFunction(mod) {
  const Module = mod.constructor;
  // 其实模块里调用的require就是Module自己的require方法
  function require(path) {
    try {
      exports.requireDepth += 1;
      return mod.require(path);
    } finally {
      exports.requireDepth -= 1;
    }
  }
  // 这里删减了代码...
  return require;
}
```

`_compile`函数会用底层的compileFunction去处理.js文件的文本内容，并且会用compiledWrapper把模块包起来。代码解释到这里结束，之所以说点到为止，是因为compileFunction和compiledWrapper无法单纯地从js的层面去分析了，所以对这两个函数的解释是基于猜测的。compileFunction的代码是在`src\node_contextify.cc`里的`ContextifyContext::CompileFunction`，这里不贴出来。

## EventEmitter
虽然nodejs很多模块基于EventEmitter，但是对一般开发者而言，更想问的是：“我应该什么时候用它？什么场景用它？”

* 用于解耦，避免丢一堆callback进去：https://stackoverflow.com/questions/38881170/when-should-i-use-eventemitter
* 典型的观察者模式：https://codeburst.io/event-emitters-and-listeners-in-javascript-9cf0c639fd63

## webpack

### 参考连接
* Element的webpack配置分析：https://juejin.im/post/5cb12844e51d456e7a303b64

### 配置说明
因为webpack的配置实在太多了，所以每个配置的具体用法还是参考官方文档比较稳妥。

#### 导出库

```js
module.exports = {
  output:{
    library: '库名',
    libraryTarget: '导出给谁(默认var)',
    libraryExport: '导出什么(默认空，导出整个)'
  },
  externals: {
    './b.js': './b.js'
  }
};
```
* libraryExport：如果`entry`用了es6的`export default 默认模块`语法，而你需要在`bundle`导出默认模块，那么这里记得填`default`(它的工作方式，就是把默认模块作为`exports`的`default`属性，然后最终bundle里把`exports.default`导出来)。如果是用nodejs那种`module.exports`方式导出整个模块，那么记得留空，留空就是导出整个`exports`。
* libraryTarget：
  * var：那么bundle里就是`var 库名 = 导出的东西`
  * commonjs2：那么就是`module.exports = 导出的东西`
  * umd：则是umd规范的库，并且用`library`里设置的名字作为模块名。umd兼容amd和commonjs，所以一般用umd就好了。
* externals：表示不把这些打包到bundle，不过会在bundle里留下替换的引用，并且不同`libraryTarget`下替换方式不同
  * var：直接替换成`module.exports = ./b.js;`
  * commonjs2：替换成`module.exports = require("./b.js");`
  * umd：`module.exports = __WEBPACK_EXTERNAL_MODULE__1__;`。`__WEBPACK_EXTERNAL_MODULE__1__`是传进工厂函数的参数，每个依赖用一个参数来代替。

### 打包简例
主要看看bundle.js的注释

源文件和配置
```js
// webpack.config.js
module.exports = {
  entry:'./a.js',
  output:{
    filename:'bundle.js'
  }
};

// a.js
var b = require('./b.js');

console.log('a');

b.b1();

// b.js
exports.b1 = function () {
  console.log('b1')
};

exports.b2 = function () {
  console.log('b2')
};
```

结果
```js
// bundle.js
/******/ (function(modules) { // webpackBootstrap
/******/    // The module cache
/******/    var installedModules = {};
/******/    // The require function
/******/    function __webpack_require__(moduleId) {
/******/        // Check if module is in cache
/******/        if(installedModules[moduleId])
/******/            return installedModules[moduleId].exports;
/******/        // Create a new module (and put it into the cache)
                // 这个module对象的结构，也是和Node.js的module的结构一样的，忽略id和loaded属性，主要是module.exports
/******/        var module = installedModules[moduleId] = {
/******/            exports: {},
/******/            id: moduleId,
/******/            loaded: false
/******/        };
/******/        // Execute the module function。看看这里的传参和下面的模块1。
/******/        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/        // Flag the module as loaded
/******/        module.loaded = true;
/******/        // Return the exports of the module
/******/        return module.exports;
/******/    }
/******/    // expose the modules object (__webpack_modules__)
/******/    __webpack_require__.m = modules;
/******/    // expose the module cache
/******/    __webpack_require__.c = installedModules;
/******/    // __webpack_public_path__
/******/    __webpack_require__.p = "";
/******/    // Load entry module and return exports
/******/    return __webpack_require__(0);
/******/ })
/************************************************************************/
/**下面就是模块数组或者对象，每个模块都是一个函数，调用的时候把module, exports, __webpack_require__什么的传进去。对比之前对Node.js的require的分析，其实很像的。**/
/******/ ([
/* 0 */
/***/ function(module, exports, __webpack_require__) {
    var b = __webpack_require__(1);
    console.log('a');
    b.b1();
/***/ },
/* 1 模块1*/
/***/ function(module, exports) {
    // 这里module、module.exports、exports这些变量，就和Node.js的结构一致。所以写webpack模块的时候，就跟写普通Node.js模块一样。
    exports.b1 = function () {
      console.log('b1')
    };
    exports.b2 = function () {
      console.log('b2')
    };
/***/ }
/******/ ]);
```

### loader
https://webpack.js.org/api/loaders/

loader就是读取文件之后，经过一系列处理，最终给编译器返回一个String或者Buffer(最终也转成String)。loader可同步也可异步。对于一个读取代码的loader，从结果来看，实际上很简单，就是读取源文件，然后生成一个用于页面的代码字符串，这个字符串最终是页面模块(函数)的内容。