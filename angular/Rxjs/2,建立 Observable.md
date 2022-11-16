建立 Observable （Rxjs 6+）

* Observable可以同时处理**同步**跟**非同步**行为
* Observer是一个物件，这个物件具有三个方法，分别是**next** , **error** , **complete**
* 订阅一个Observable 就像在执行一个function

```js
// observer 观察者的三个方法(method)：
// 	next：每当Observable 发送出新的值，next 方法就会被呼叫。
// 	complete：在Observable 没有其他的资料可以取得时，complete 方法就会被呼叫，在complete 被呼叫之后，next 方法就不会再起作用。
//	error：每当Observable 内发生错误时，error 方法就会被呼叫。

import { Observable } from "rxjs";
var observable = new Observable(observer => {
  observer.next("Jerry");
  observer.next("Anna");
  observer.complete();
  observer.next("not work");
});
// 订阅这个 observable
observable.subscribe({
  next: value => console.log("next", value),
  error: error => console.log("error", error),
  complete: () => console.log("complete")
});
```



### Observable 创建实例的方法

>常用创建实力方法:
>
>* ### `new` 关键字 : 如上所示
>
>* #### `of`
>
>```js
>import { of } from "rxjs";
>var observable = of("Jerry","Anna");
>// 订阅这个 observable
>observable.subscribe({...});
>```
>
>* ### `from`  接收任何可列举的参数
>
>```js
>import { from } from "rxjs";
>var list = ["Jerry","Anna"];
>var observable = from(list); // "Jerry"  "Anna"
>
>// 能接收字符串
>// var observable = from("list"); // l i s t
>
>// 接收 Promise 也可以使用 FromPromise 实现相同的效果
>// var observable = from(
>//   new Promise((resolve, raject) => {
>//     setTimeout(() => {
>//       resolve("11232");
>//     }, 5000);
>//     raject(12);
>//   })
>// );
>
>// 订阅这个 observable
>observable.subscribe({...});
>```
>
>* ### `fromEvent`
>
>```js
>// fromEvent的第一个参数要传入DOM 对象，
>// 第二个参数传入要监听的事件名称。
>// 针对p标签的click 事件做监听，每当点击p标签 就会印出event。
>import { fromEvent } from "rxjs";
>var observable = fromEvent(document.getElementsByTagName("p"), "click");
>// 订阅这个 observable
>
>observable.subscribe({...});
>```
>
>* ### `fromEventPattern`
>
>  * 要用Event来建立Observable实例还有另一个方法`fromEventPattern`，这个方法是给类事件使用
>
>  * 所谓的类事件就是指其行为跟事件相像，同时具有注册监听及移除监听两种行为，就像DOM Event有`addEventListener`及`removeEventListener`一样！
>
> ```js
> import { fromEventPattern } from "rxjs";
> class Producer {
> 	constructor() {
> 		this.listeners = [];
> 	}
> 	addListener(listener) {
> 		if(typeof listener === 'function') {
> 			this.listeners.push(listener)
> 		} else {
> 			throw new Error('listener 必須是 function')
> 		}
> 	}
> 	removeListener(listener) {
> 		this.listeners.splice(this.listeners.indexOf(listener), 1)
> 	}
> 	notify(message) {
> 		this.listeners.forEach(listener => {
> 			listener(message);
> 		})
> 	}
> } 
> var egghead = new Producer(); 
> // egghead 同時具有 组册监听者移除监听者 两种方法
> var source =  fromEventPattern(
>         (handler) => egghead.addListener(handler), 
>         (handler) => egghead.removeListener(handler)
>     );
> observable.subscribe({...});
> egghead.notify('Hello! Can you hear me?');
> // Hello! Can you hear me?
> // egghead是Producer的实例，同时具有注册监听及移除监听两种方法
> // 我们可以将这两个方法依序传入fromEventPattern来建立Observable的物件实例！
> // var source =fromEventPattern(
> //        egghead.addListener.bind(egghead), 
> //        egghead.removeListener.bind(egghead)
> //    )
> //    .subscribe(console.log)
> ```
>
>* ### `EMPTY`, `NEVER`, `throwError`
>
>```js
>import { EMPTY, NEVER, throwError } from "rxjs";
>// 常量 EMPTY 得到一个空的 observable
>var observable = EMPTY;
>// 常量 NEVER 得到一个一直存在但却什么都不做的 observable
>var observable = NEVER;
>// throwError 只做一件事抛出错误
>var observable = throwError("123123");
>// 订阅 observable
>observable.subscribe({
>  next: value => console.log("next", value),
>  error: error => console.log("error", error),
>  complete: () => console.log("complete")
>});
>```
>
>* ### `interval`  `timer`
>
>```js
>// 原始实现：每隔一秒送出一个从0开始递增的整数
>// 每隔一秒送出一個從零開始遞增的整數，在 Observable 的世界也有一個 operator 可以更方便地做到這件事，就是 interval
>import { Observable, interval, timer } from "rxjs";
>const observable = new Observable(function(observer) {
>  var i = 0;
>  setInterval(() => {
>    observer.next(i++);
>  }, 1000);
>});
>// interval
>// Observable operator 的interval可以方便的实现   
>// 参数为 每间隔时间开始递增
>const observable = interval(1000);
>// timer
>// 第一个参数为 第一个值的等待时间
>// 第二个参数为 第一次之后的送值间隔时间
>const observable = timer(1000, 5000);
>// 只有一个值的时候 就会等一秒送出1同时通知结束
>const observable = timer(1000);
>// 订阅这个 observable
>observable.subscribe({...});
>```
>
>* ### `Subscription`
>
>  * 在无限的 observable 持续性的发送资源的情况下，需要在某个行为后不需要这个资源了
>  * 在订阅 observable 后 会返回一个 subscription 对象，这个对象具有释放资源 unsubscribe() 方法
>
>  ```js
>  import { timer } from "rxjs";
>  const observable = timer(1000, 1000);
>  
>  // 订阅这个 observable
>  const subscription = observable.subscribe({
>    next: value => console.log("next", value),
>    error: error => console.log("error", error),
>    complete: () => console.log("complete")
>  });
>  setTimeout(() => {
>    subscription.unsubscribe();
>  }, 5000);
>  
>  ```
>
>  

 
