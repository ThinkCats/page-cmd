---
title: ES6 Promise & Generator
date: 2016-05-31 13:30:52
tags: ES6
categories : JavaScript
---
最开始从Promise做起，用它来解决回调问题。Generator 搞了好久，当时没懂是啥意思，看语法倒是蛮简单的，就是不知道怎么去解决回调问题的。

Generator 的具体语法，网上很多。这里主要是演示如何使用Generator 和 Promise 结合，解决 Callback Hell 的问题。

举个栗子，模拟一个场景。

如果我们要使用 js 去进行数据库的增删改查，会怎么写代码呢？ 肯定是先获取数据库的连接，然后查询到数据，在进行处理，再进行删改等操作。

用 js 去实现代码：

##### 1.最直观的方式(该栗子纯扯淡的伪代码)
```
function op1{
  Db.getConnection((connection) =>{
    //得到connection
    connection.find((queryParam , table),(resultSet) =>{
      //查询到resultSet数据
      //好，拿到数据了，开始大刀阔斧，做事情。。
      let processResult = MyProcess.doSomething();
      //做完事了，该更新了
      connection.update((processResult, table), (result) =>{
          //更新完毕了，接着做更新后该做的其他事情
          ....
        });
      })
    });
}
```

如果这个处理比较长的话，那么代码要一直这么回调下去了，回调地狱嘛。

##### 2.使用 Promise(该栗子也是纯扯淡的伪代码)
```
let process = new Promise((resolve) => resolve('dbURL'));

//获取连接
function getConnection(url){
  //开始做点自己要做的，然后拿到结果，返回出来
  return DBUtil.getConnection(url);
}

//查询
function query(conn,param){
  //查出来一堆东西，处理，做自己想做的事情
  MyProcess.doSomething();
  return result;
}

//开始进入正题，处理主流程
process.then((url) => getConnection(url)).then((connection) => query(connection,param)).then((result) =>{
    //得到最终的结果result
  }).catch(err){
    //捕获错误
  }


```
同样的逻辑，这个流程看起来能明了点，也没有了那些长长的回调了，取而代之的是链式的 then 函数。一定程度上解决了 Callback Hell 。

##### 3.使用 Generator 的方式（这个栗子是真实可以Run的栗子，不是啥伪代码了，免得被骂）
这个栗子，用的是 mongodb 的库，在本地建了一个 test 库，还有一个 users 表（collection）。

```
import mongodb from 'mongodb';

const MongoClient = mongodb.MongoClient;
const DB_URL = 'mongodb://127.0.0.1:27017/test';

function Connect(url){
    MongoClient.connect(url,(err,db) =>{
      console.log('db');
      if (err){
        console.error(err);
        it.throw(new Error('Db Error'+err.message));
      }
      it.next(db);
    })
}

function query(db){
    db.collection('users').find().toArray((err,doc) =>{
      console.log('doc:',doc);
        it.next(doc);
    });
}

function* main(){
  try{
    let db = yield Connect(DB_URL);
    console.log('now db:');
    let doc = yield query(db);
    console.log('now doc:',doc);
  }catch (err){
    console.log('catch err:',err.message);
    return;
  }

}

let it = main();
it.next();
```
可能多数人和我一样，从网上的教程里看到的，测试 Generator 的时候，都是不断的调 next() 函数。 `这里在具体的每一步里面，都调用一次声明的 it [ main() 函数] 的 next（）函数，然后在入口，在掉一次 it.next()  ` ,这样，一连串的 next() 就会逐步触发了，整个流程就走下去了。

关于传参，it.next() 可以这一步的结果返回出来。

这样，代码貌似就清爽多了。回头一看，这不一股java的既视感。

可能有人也发现了一个问题，要是多个人分工写代码的话，A 写Connect , B 写 Query ,那不得都得预先知道main() 这个函数被声明的变量名？ 这不扯淡的么。。。

确实，上面代码太扯淡了。

##### 4.使用 co 框架
第三步里面的代码太扯淡了，还有，到现在，还没有看到 Generator 和 Promise 一起用的栗子啊。

所以，这里用 co 了，这东西是TJ大神写的，比较好用。当然，也可以自己写一个 runGenerator() ，网上栗子也蛮多的。

改造上面的代码，结合 Promise：
```
import mongodb from 'mongodb';
import co from 'co';

const MongoClient = mongodb.MongoClient;
const DB_URL = 'mongodb://127.0.0.1:27017/test';

function Connect(url){
    return new Promise((resolve,reject) =>{
        MongoClient.connect(url,(err,db) =>{
            if (err){
                reject(new Error('can not connect to db'));
            }else {
             resolve(db);
            }
        })
    });
}


function query(db){
    return new Promise((resolve,reject) =>{
        db.collection('users').find().toArray((err,doc) =>{
            if (err) reject(new Error('can not find users'));
            else  {
                resolve(doc);
            }
        });
    });
}

function* main() {
    let db = yield Connect(DB_URL);
    let doc = yield query(db);
    console.log('main result:',doc);
}

co(main()).catch((err) =>{
    console.log('catch err:',err.message);
});

```
这样，大家一起写代码的话，就个玩个的去吧 ！

关于这两段代码的环境啥的，已经放在 github了，[传送门](https://github.com/ThinkCats/GeneratorAsync)。
