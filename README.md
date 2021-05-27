> **storagex-js 1.4.0**

## 首先需要安装引入 storagex-js
```js
//使用npm安装
npm install storagex-js
//导入 需要什么就导入什么 所有方法皆是同步方法
import {
    removeStorageItem,
    storageX,
    StorageX,
    localStorageX,
    LocalStorageX,
    sessionStorageX,
    SessionStorageX,
    uniStorageX,
    UniStorageX,
    wxStorageX,
    WxStorageX,
} from "storagex-js";
```
## 储存模式（下面会用到）
mode :可选值 local,session,uni,wx
local：localStorage,session:sessionStorage,uni:uniApp的本地储存,wx:微信小程序的本地储存

## 操作方式
皆是使用以上引入的方法生成代理对象，对代理对象的操作会相应的将该代理的对象储存到本地，操作该代理对象有两种模式
##### 1：只是对键值进行操作
```javascript
const storageObj= <调用方法生成代理对象>;  //下面会详细说明如何生成
storageObj.a = {b:1};  //'a' 对应的值 {'b':'1'}
storageObj.a.b = 2  //无效，不允许深度修改
//需要这样
storageObj.a = {b:2};
```
##### 2：深度的对象化存储（单纯的只对值进行操作）
```javascript
//因为考虑到如果需要深度代理那么值一定是个对象
let state = {  //需要初始化的值
    a:1,
    b:{
        c:3,
    },
};
const storageObj = <调用方法生成可深度代理对象>;
//当storageObj的属性发生变化时会相应的将state存储到本地。
storageObj.a = 1;
storageObj.a = {b:{c:{d:2}}};
storageObj.a.c.d = 3;
console.log(storageObj.a.c.d);  //打印 3
//只针对值为对象时，而且只会对该对象属性进行修改添加，也就是说像下面这样不管用
storageObj = null;
storageObj = 1;
storageObj = {a:1};
//了解js基础都知道，这样只是改变该变量的指向，并不是对原对象进行修改。
//如果想修改键的值请使用上面的那种方式。
```

> ## 各种方法的说明，相似的跳过
## 一：removeStorageItem 的使用方法
```javascript
//key :需要删除数据的键，mode：储存模式（默认:local）
removeStorageItem(<key>,<mode>);
```
## 二：storageX 的使用方法
##### 1：生成对键值进行操作的代理对象
```javascript
const storageObj= storageX(undefined,undefined,<mode>);  //mode必填
```
##### 2：生成可深度操作的代理对象
```javascript
let state = {
    a:1,
    b:{
        c:3,
    },
};
const storageObj = storageX(
    "state",  //储存所对应的键
    state,  //初始化的值，如果该键有数据且为对象时优先代理
    <mode>,  //存储模式
);
```
##### 3：删除
```javascript
storageObj.removeItem(<key>,<mode>);
```

## 三：StorageX 的使用方法（对象式创建）
##### 1：生成对键值进行操作的代理对象
```javascript
const storageObj= new StorageX({
    mode:<mode>,  //必填
});
```
##### 2：生成可深度操作的代理对象
```javascript
let state = {
    a:1,
    b:{
        c:3,
    },
};
const storageObj = new StorageX({
    key:'state',
    target:state,
    mode:<mode>,
});
```
## 四：localStorageX 的使用方法
##### 1：生成对键值进行操作的代理对象
```javascript
const storageObj= localStorageX();
```
##### 2：生成可深度操作的代理对象
```javascript
let state = {
    a:1,
    b:{
        c:3,
    },
};
const storageObj = localStorageX(
    "state",  //储存所对应的键
    state,  //初始化的值，如果该键有数据且为对象时优先代理
);
```

## 五：LocalStorageX 的使用方法（对象式创建）
##### 1：生成对键值进行操作的代理对象
```javascript
const storageObj= new LocalStorageX();
```
##### 2：生成可深度操作的代理对象
```javascript
let state = {
    a:1,
    b:{
        c:3,
    },
};
const storageObj = new LocalStorageX({
    key:'state',
    target:state,
});
```
##### 3：删除
```javascript
LocalStorageX.removeItem(<key>);
```

##其他几种模式的方法都差不多了，这里就不一一赘述了。

**皆不支持函数式储存，因为函数无法JSON化，既然是数据的话为什么要存函数呢？**

**可能遇到的问题：因为深度对象化代理难免遇到大数据量的循环修改，每修改一次就存一次太消耗性能，所以一次任务有多次修改的话，那么最多会存两次，一是第一次修改存一次，二是创建了一个宏任务等待被执行后保存（跟js写的防抖函数类似）。**

> ## 开发日志

## 1.4.1 对不是深层次的代理对象修改后立即更新到本地储存

## 1.4 添加类的形式引入

## 1.3 修复BUG
存储字符串的情况下会多出两个冒号（致命错误）

## 1.2 修复BUG
1：储存的是数组时，取出来重新赋值会丢失map,filter等自带的方法。原因：json化数据时触发了代理的set属性，会调用clone方法，因为当时考虑到数据的储存简单clone一下就行，现在删除了clone。bug解决。
2：直接调用localStorageX()生成对象从而控制键值，以及更深层次的值，我认为这样不好。解决：取消深层代理。
