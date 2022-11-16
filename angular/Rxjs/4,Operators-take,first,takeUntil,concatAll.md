# Operators： take，first，takeUntil，concatAll 

>* take: 取前几个就结束
>
>  ```js
>  import { interval } from "rxjs";
>  import { take } from "rxjs/operators";
>  var people = interval(1000);
>  var example = people.pipe(take(3));
>  example.subscribe({
>    next: value => console.log(value),
>    error: err => console.log("Error: " + err),
>    complete: () => console.log("complete")
>  });
>  ```
>
>* first: 送出的一个元素之后就结束 和 take(1) 一样
>
>  ```js
>  //...
>  var example = people.pipe(first());
>  //...
>  ```
>
>* takeUntil 可以在某个事件发生时，让一个observable 直接完成(complete)
>
>  ```js
>  import { interval, fromEvent } from "rxjs";
>  import { takeUntil } from "rxjs/operators";
>  var people = interval(1000);
>  var click = fromEvent(document.body, "click");
>  var example = people.pipe(takeUntil(click));
>  example.subscribe({
>    next: value => console.log(value),
>    error: err => console.log("Error: " + err),
>    complete: () => console.log("complete")
>  });
>  
>  ```
>
>* concatAll 
>
>  ```js
>  // Observable 每次发送的值也是 observable
>  // 这个时候我们可以用 concatAll 把 source 摊平成 example
>  // 也可以直接把 concatAll 想成把所有元素 concat 起来。
>  import { fromEvent, of } from "rxjs";
>  import { concatAll, map } from "rxjs/operators";
>  var click = fromEvent(document.body, "click");
>  var soure = click.pipe(map(e => of(1, 2, 3)));
>  var example = soure.pipe(concatAll());
>  example.subscribe({
>    next: value => console.log(value),
>    error: err => console.log("Error: " + err),
>    complete: () => console.log("complete")
>  });
>  
>  
>  
>  // 注意：concatAll 会处理 source 先发出来的 observable
>  // 而且必须等到这个observable结束，才会处理下一个 source 发出来的 observable
>  import { of, interval } from "rxjs";
>  import { concatAll, take } from "rxjs/operators";
>  var obs1 = interval(1000).pipe(take(5));
>  var obs2 = interval(500).pipe(take(2));
>  var obs3 = interval(2000).pipe(take(1));
>  var soure = of(obs1, obs2, obs3);
>  var example = soure.pipe(concatAll());
>  example.subscribe({
>    next: value => console.log(value),
>    error: err => console.log("Error: " + err),
>    complete: () => console.log("complete")
>  });
>  ```



## 实现简单的拖动效果

```js
import { fromEvent } from "rxjs";
import { concatAll, map, takeUntil } from "rxjs/operators";

const dragDOM = document.getElementById("drag");
const body = document.body;

// 按钮按下时触发
const mouseDown = fromEvent(dragDOM, "mousedown");
// 释放鼠标按钮时
const mouseUp = fromEvent(body, "mouseup");
// 在用户移动鼠标时发生。
const mouseMove = fromEvent(body, "mousemove");

mouseDown
  .pipe(
    // 通过点击dom对象触发开始
    // 通过map 得到 释放鼠标结束的 observable
    // 使用 concatAll 平铺 mouseMove 移动鼠标返回的一系列 observable
    // 在使用 map 重组  subscribe 得到的数据
    map(event => mouseMove.pipe(takeUntil(mouseUp))),
    concatAll(),
    map(event => ({ x: event.clientX, y: event.clientY }))
  )
  .subscribe({
    next: value => {
      console.log(value);
      dragDOM.style.left = value.x + "px";
      dragDOM.style.top = value.y + "px";
      // console.log(dragDOM);
    },
    error: err => console.log("Error: " + err),
    complete: () => console.log("complete")
  });
```
