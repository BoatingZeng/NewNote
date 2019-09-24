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