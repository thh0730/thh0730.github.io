---
layout: post
title: 关于Promise
---


* Promise的基本概念

  ```
  	Promise是ES6中新增的异步编程解决方案,在代码中的表现是一个对象;
  ```

  * ES6之前代码示例

    ```js
    //需求:从网络上加载3个资源,要求加载完资源1之后才能加载资源2,加载完资源2之后才能加载资源3
        //     前面任何一个资源加载失败,后续资源都不加载;
    
        function request(fn) {
            setTimeout(function () {
                fn("拿到的数据")
            }, 1000);
        }
    
        //ES6之前的回调函数嵌套方式,阅读性比较差
        request(function (data) {
            console.log(data, 1);	//拿到的数据 1
            request(function (data) {
                console.log(data, 2); //拿到的数据 2
                request(function (data) {
                    console.log(data, 3); //拿到的数据 3
                });
            });
        });
    ```

* Promise 的作用

  ```
  	在企业开发中为了保存异步代码执行的顺序,那么就会出现回调函数层层嵌套的现象,如果回调函数嵌套的层数太多,就会导致代码的阅读性与可维护性大大降低,在ES6中推出了Promise对象,可以将异步操作以同步流程来表示,避免了回调函数层层嵌套的现象(回调地狱);
  ```

  * 使用Promise改造上述代码

    ```js
    //需求:从网络上加载3个资源,要求加载完资源1之后才能加载资源2,加载完资源2之后才能加载资源3
        //     前面任何一个资源加载失败,后续资源都不加载;
    
    	function request(){
            return new Promise(function (resolve, reject){
                setTimeout(function (){
                    resolve("拿到的数据"); 
                }, 1000);
            });
        }
    
    	request().then(function (data){
            console.log(data, 1);	//拿到的数据 1
            return request();
        }).then(function (data){
            console.log(data, 2);	//拿到的数据 2
            return request();
        }).then(function (data){
            console.log(data, 3);	//拿到的数据 3
            return request();
        });
    ```



* 什么是Promise?

  ```
  	上述讲到Promise是ES6中新增的异步编程解决方案,在代码中的表现是一个对象,通过Promise就可以实现用同步的流程来表示异步的操作,通过Promise就可以避免回调函数层层嵌套(回调地狱)的问题;
  ```

* Promise的实现原理

  ```
  	1.如何创建Promise?
  	new Promise( function(resolve, reject) {} );
  	Promise对象不是异步的,只要创建Promise对象就立即执行存放的代码;
  	
  	2.实现原理
  	Promise对象是通过状态的改变来实现的,只要状态发生改变就会自动触发对应的函数
  	
  	3.Promise的三种状态
  	pending: 初始状态，既不是成功，也不是失败状态。
  	fulfilled(resolved): 意味着操作成功完成,只要调用Promise内传入的resolve()函数,就表示成功。
  	rejected: 意味着操作失败,只要调用Promise内传入的reject()函数,就表示失败。
  	
  	* 注意:状态一旦改变就不可逆,既从pending变为fulfilled(resolved),那就永远是fulfilled(resolved),既从pending变为rejected,那就永远是rejected;
  	
  	4.监听Promise状态改变
  	我们还可以通过函数来监听状态的变化;
  	resolved --> then();	状态为成功,就执行then();
  	rejected --> catch();	状态为失败,就执行catch();
  	
  	* 状态为pending时不会调用成功或者失败的函数;
  ```

* then()方法

  ```js
  	1.在then()方法中可以接收两个参数,第一个参数是状态切换为成功时的回调,第二个参数是状态切换为失败时的回调;
  
  示例:
  			//创建一个Promise对象
              let promise = new Promise(function (resolve, reject) {
                  // resolve(); //成功
                  reject(); //失败
              });
              promise.then(function () {
                  console.log("成功");
              }, function () {
                  console.log("失败");
              });
  
  	2.同一个Promise对象可以多次调用then()方法,当该Promise对象的状态改变时,所有相同状态的then()方法都会被执行;
  
  示例:
              let promise = new Promise(function (resolve, reject) {
              resolve(); //成功	当状态为成功时,控制台打印成功1 成功2
              // reject(); //失败 当状态为失败时,控制台打印失败1 失败2
              });
              promise.then(function () {
                  console.log("成功1");
              }, function () {
                  console.log("失败1");
              });
              promise.then(function () {
                  console.log("成功2");
              }, function () {
                  console.log("失败2");
              });
  
  	3.then()方法每次执行完毕后会返回一个新的Promise对象,而且新的Promise对象会继承前一个Promise对象的状态;
  
  示例:
  
              let promise = new Promise(function (resolve, reject) {
                  // resolve(); //成功
                  reject(); //失败
              });
              let p2 = promise.then(function () {
                  console.log("成功1");
              }, function () {
                  console.log("失败1");
              });
              console.log(p2);	//返回了一个Promise对象,状态为rejected
              console.log(promise === p2); //返回了false
  
  	4.可以通过上一个Promise对象的then()方法给下一个Promise对象的then()方法传递参数,无论在上一个Promise对象成功的回调还是失败的回调传递的参数都会传递给下一个Promise对象成功的回调;
  
  示例:
  
              let promise = new Promise(function (resolve, reject) {
                  // resolve("111"); //成功
                  reject("aaa"); //失败 
              });
              //第一个then()
              let p2 = promise.then(function (data) {
                  console.log("成功1", data);   //控制台输出成功1 111
                  return "222";  //返回的值会给下一个then()的成功做传参
              }, function (data) {
                  console.log("失败1", data);
                  return "bbb"; //返回的值会给下一个then()的*成功*做传参
              });
  
              //第二个then()
              p2.then(function (data) {
                  console.log("成功2", data);   //当状态为成功时,控制台输出成功2 222,当状态为失败时,控制台输出成功2 bbb;
              }, function (data) {
                  console.log("失败2", data);
              });
  
  	5.如果then()方法返回的是一个Promise对象,那么会将返回的Promise对象的执行结果中的值传递给下一个then()方法;
  
  示例:
  
              let promise = new Promise(function (resolve, reject) {
                  resolve("111"); //成功
                  // reject("aaa"); //失败
              });
              let ppp = new Promise(function (resolve, reject) {
                  // resolve("222"); //成功
                  reject("bbb"); //失败
              });
              //第一个then()
              let p2 = promise.then(function (data) {
                  console.log("成功1", data);   //控制台输出成功1 111
                  return ppp;   //返回一个Promise对象
              }, function (data) {
                  console.log("失败1", data);
                  return "bbb";
              });
  
              //第二个then()
              p2.then(function (data) {
                  console.log("成功2", data);
              }, function (data) {
                  console.log("失败2", data);
              });
  
  	* 下一个Promise执行成功的回调函数还是失败的回调函数取决于返回的ppp的Promise对象的状态是成功还是失败;
  ```

* catch()

  ```js
  	1.catch()方法其实是then(undefined, () => {})的语法糖,意思就是then()方法中没有成功,只有失败状态的回调函数;
  	注意:在企业开发中,如果需要分开监听Promise的成功和失败状态的回调函数,只需要链式编写就可以;在监听Promise状态时,如果不监听失败状态,会报错误:Uncaught (in promise) undefined;
  
  示例:
  
              let promise = new Promise((resolve, reject) => {
                  // resolve();
                  reject();
              });
  
              //链式编写
               promise.then( () => { //es6的function(){}写法
                   console.log("成功");
               } ).catch( () => {
                   console.log("失败");
               } );
  
              //分开编写(错误示范)
              promise.then( () => {
                  console.log("成功");
              } );
              promise.catch( () => {
                  console.log("失败");
              } );
  
  	2.同一个Promise对象可以多次调用catch()方法,当该Promise对象的状态改变时,所有相同状态的catch()方法都会被执行;
  
  示例:
              let promise = new Promise(function (resolve, reject) {
              // resolve(); //成功	当状态为成功时,控制台打印成功1 成功2
               reject(); //失败 当状态为失败时,控制台打印失败1 失败2
              });
              promise.catch(function () {
                  console.log("失败1");
              });
              promise.catch(function () {
                  console.log("失败2");
              });
  
  	3.catch()方法每次执行完毕后会返回一个新的Promise对象;
  
  示例:
  
              let promise = new Promise(function (resolve, reject) {
                  // resolve(); //成功
                  reject(); //失败
              });
              let p2 = promise.catch(function () {
                  console.log("失败1");
              });
              console.log(p2);	//返回了一个Promise对象,状态为rejected
              console.log(promise === p2); //返回了false
  
  	4.无论在上一个Promise对象成功的回调还是失败的回调传递的参数都会传递给下一个Promise对象成功的回调;
  
  示例:
  
              let promise = new Promise(function (resolve, reject) {
                  // resolve("111"); //成功
                  reject("aaa"); //失败 
              });
              let p2 = promise.catch(function (data) {
                  console.log("成功1", data);   //控制台输出成功1 111
                  return "222";  //返回的值会给下一个then()的成功做传参
              });
  
              p2.then(function (data) {
                  console.log("成功2", data);   //当状态为成功时,控制台输出成功2 222,当状态为失败时,控制台输出成功2 bbb;
              }, function (data) {
                  console.log("失败2", data);
              });
  
  	5.如果catch()方法返回的是一个Promise对象,那么会将返回的Promise对象的执行结果中的值传递给下一个catch()方法;
  
  示例:
  
              let promise = new Promise(function (resolve, reject) {
                  // resolve("111"); //成功
                   reject("aaa"); //失败
              });
              let ppp = new Promise(function (resolve, reject) {
                  // resolve("222"); //成功
                  reject("bbb"); //失败
              });
  
              let p2 = promise.catch(function (data) {
                  console.log("失败1", data);   //控制台输出成功1 111
                  return ppp;   //返回一个Promise对象
              });
  
              p2.then(function (data) {
                  console.log("成功2", data);
              }, function (data) {
                  console.log("失败2", data);
              });
  
  	* 下一个Promise执行成功的回调函数还是失败的回调函数取决于返回的ppp的Promise对象的状态是成功还是失败;
  
  	6.和then()方法第二个参数的区别在于,catch方法可以捕获上一个Promise对象then()方法中的异常;
  
  示例:
  
              let promise = new Promise((resolve, reject) => {
                  resolve();
                  // reject();
              });
              promise.then( () => {
                  console.log("成功");
                  xxx     // xxx is not defined
              } ).catch( (error) => {	//传入参数可以自定义名称
                  console.log("失败", error);   
              });
  	* then()方法里的代码异常(错误)会被catch()捕获,并作为参数传入catch()方法
  ```

* Promise.all()方法

  ```js
  	Promise的all静态方法:
  						1.all方法接收一个数组;
  						2.如果数组中多个Promise对象,只有都成功了才会执行then的成功方法,并且会按							 照添加的顺序,将所有成功的结果重新打包到一个数组中返回给我们;
  						3.如果数组中不是Promise对象,那么就会指直接执行then方法;
  
  示例:
  
          <script>
                  /*
                      应用场景: 要么一起成功,要么一起失败
                  */
                  let p1 = new Promise(function (resolve, reject) {
                       resolve("111");
                      // reject("aaa");
                  });
                  let p2 = new Promise(function (resolve, reject) {
                      setTimeout(function () {
                          resolve("222");
                      }, 5000);
                  });
                  let p3 = new Promise(function (resolve, reject) {
                      setTimeout(function () {
                          resolve("333");
                      }, 3000);
                  });
  				//all方法会在成功执行后将resolve()传入的参数组成一个新的数组作为参数传入成功的
  				//	then()方法中,当p1,p2(5s),p3(3s)都成功执行之后,才会执行成功的then()方法,
  				//	只要有一个失败,那就会直接执行失败的then()方法;
                  Promise.all([p1, p2, p3]).then(function (result) {
                      console.log("成功", result);
                      //result = ["111","222","333"]
                  }, function (err) {
                      console.log("失败", err)
                      //err = "aaa"
                })
              </script>
  ```
  
* Promise.race()方法

  ```js
  	Promise的race静态方法:
  						1.race方法接收一个数组;
  						2.如果数组中有多个Promise对象,谁先返回状态就听谁的,后返回的会被抛弃;
  						3.如果数组不是Promise对象,那么久会执行then()方法;
  						
  示例:
  
              <script>
                  /*
                      应用场景: 接口调试,超时处理
                  */
                  let p1 = new Promise(function (resolve, reject) {
                      resolve("111");
                      // reject("aaa");
                  });
                  let p2 = new Promise(function (resolve, reject) {
                      setTimeout(function () {
                          // resolve("222");
                          reject("bbb");
                      }, 5000);
                  });
                  let p3 = new Promise(function (resolve, reject) {
                      setTimeout(function () {
                          // resolve("333");
                          reject("ccc");
                      }, 3000);
                  });
  				
  				//谁先返回状态就听谁的
                  Promise.race([p1, p2, p3]).then(function (value) {
                      console.log("成功", value); //接收一个状态传递的参数
                  }, function (err) {
                      console.log("失败",err);
                  })
              </script>
  ```

* JavaScript异常处理机制

  ```
   	1.什么是JS中的异常?
    	简单粗暴就是有错误出现。由于JS是单线程的,编写的代码都是串行的,所以一旦前面的代码出现错误,程序就会被中断,后续代码就不会执行;
    
    示例:
    
                console.log(1);
                say();
                console.log(2);
                
    	* 上述代码输出的是1,因为say()未被定义的错误中断了程序执行所以2并未被打印;
    	
    	2.JS中异常的处理
    	* 自身编写的代码问题 --> 手动修复bug;
    	* 外界原因问题,如用户输入错误 --> try{}catch(){};
    	对于一些可预见的异常,我们可以使用try{}catch(){}来处理;
    	
    	3.JS中如何进行异常处理?
    	* 利用try{}catch(){}来处理异常可以保证程序不被中断,也可以记录错误原因以便于后续优化迭代更新,相关的语法:
                try{ 可能会遇到意外的代码 }catch(e){ 捕获错误的代码块 };
    
    	* try{}catch(){}可以让我们捕获到报错的信息,并且还可以让程序继续执行下去;
              
    示例:
    
                console.log("1");
                try {
                    say();
                }catch (e) {
                    console.log(e);
                }
                console.log("2");
    
    	* 上述代码输出的是:
  ```

  

  ​								  ![try{}catch(){}示意图](E:\学习笔记\JavaScript\try{}catch(){}示意图.png)