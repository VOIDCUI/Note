# 2019.11.27  async/await用法与例子

## 平时通常用于解决异步操作的几个方法

1. 用promise
2. 嵌套回调
3. 使用Generators

在ES7之后推出async/awit方法，简单来说以同步的思想来解决异步操作，避免了回调地狱。

## 什么是回调地狱呢

”回调地狱“也叫”回调金字塔“，我们平时写代码的时候 js如果异步 回调是不可避免的
这会使得我们的代码可读性变差，出现问题 不好调试 也会导致性能下降
就是一个函数的参数得接受上一个函数完成运行后结果，然后函数会不断地嵌套，出现大量的 )}

例如下面 ajax不断的进行异步请求数据 回调方法里还要对数据进行处理，继续回调...形成回调地狱
```
$.ajax({
    url:"/user",
    type: 'POST',
    data: {
        username: 'xiaocui',
        password: '123456'
    },
    success: function(res) {
        if (res.data.identify == "admin") {
            $.ajax({
                url:'/user/admin',
                type:'GET',
                success: function(res) {
                    if (res) {
                        ajax({
                            url:'/post/admin/add',
                            type:'POST',
                            data: {
                                user: {
                                    username: 'xiaozhu',
                                    password: 123123
                                }
                            },
                            success: function(res) {
                                if(res.data.status = '200') {
                                    console.log('添加用户成功')
                                }
                            }
                        })
                    }
                } 
            })
        }
    }
})
```

一开始我们可能会采取Promise来解决这个问题
```
new Promise((reslove, reject) = > {
    ajax({
        url:"/user",
        type: 'POST',
        data: {
            username: 'xiaocui',
            password: '123456'
        }
        success: function(res) {
            resolve(res);
        } 
    })
})
.then(
    res.data => return new Promise((resolve) => {
         $.ajax({
                url:'/user/admin',
                type:'GET', 
                success: function(res) {
                    consolo.log(res);
                    return res;
                }
    })
    }
))
.then(res.data => .... )
.then(res.data => .... )
.then(res.data => .... )

```
虽然使用了promise对象，但是一路通过 ,但这种链式调用方法进行下去，代码的可读性较差。

所以ES7推出了async/await来解决了这个问题
```
async getHttp() {
   let ret1 = await queryData('data1');
   let ret2 = await queryData('data2');
   let ret3 = await queryData('data3');
   return res;
}
```
在async函数中使用await，那么await这里的代码就会变成同步的了，意思就是说只有等await后面的Promise执行完成得到结果才会继续下去，await就是等待，这样虽然避免了异步，但是它也会阻塞代码，所以使用的时候要考虑周全。


## 使用async/await注意事项
 1. awit必须得在有async修饰的函数中使用, await只能在async函数内部使用,用在普通函数里就会报错
 ```
async function a() {
     let res = awit p1();
 }

 ```
2. async语法会将自动将函数转化成一个promise,返回结果也是一个promise

```
let example = async function b() {
    let res = awit p1();
    return 'ok!';
}
console.log(example())
// Promise{<resolve>:'ok!'} 结果返回了一个resolve的promise
```


3. 只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数

```
let p1 = new Promise((reslove,reject) => [
    resolve(1);
]) 

let p2 =async function a() {
    let res = awit p1();
    console.log(2);
}
p2().then(
    res => console.log(3)
)

///打印结果为1 2 3
```

## 关于错误信息处理
因为错误信息有可能是发生在async函数内部，而通过链式.then().catch()获取不到错误信息 所以需要在内部使用try{}catch{}来捕捉awit函数中出现错误
```
function timeout(ms) {

  return new Promise((resolve, reject) => {

    setTimeout(() => {reject('error')}, ms);  //reject模拟出错，返回error

  });

}
async function asyncPrint(ms) {

  try {

     console.log('start');

     await timeout(ms);  //这里返回了错误

     console.log('end');  //所以这句代码不会被执行了

  } catch(err) {

     console.log(err); //这里捕捉到错误error

  }

}
asyncPrint(1000);
```



