# Operators：skip,takeLast,last,concat,startWith,merge

>* ### `skip`:忽略前几个送出的元素
>
>  ```js
>  import { interval } from "rxjs";
>  import { skip } from "rxjs/operators";
>  interval(1000)
>    .pipe(skip(5))
>    .subscribe(console.log);
>  ```
>
>* ### `takeLast`: 取最后几个
>
>  * 注意：必须等到整个observable 完成 (complete)，才能知道最后的元素有哪些，并且同步**同步送出**
>
>  ```js
>  import { interval } from "rxjs";
>  import { take, takeLast } from "rxjs/operators";
>  
>  interval(1000)
>    .pipe(
>      take(5),
>      takeLast(2)
>    )
>    .subscribe(console.log);
>  ```
>
>* ### `last`：最后一个 
>
>* ### `concat`:多个observable 合并成一个
>
>  * 6.0后被移入到 rxjs包中
>  * 必须先等前一个observable完成 complete 才会继续下一个
>
>  ```js
>  import { interval, of, concat } from "rxjs";
>  import { take } from "rxjs/operators";
>  var s1 = interval(1000).pipe(take(5));
>  var sou = of(3);
>  var sou1 = of(4, 5, 6);
>  concat(s1, sou, sou1).subscribe({
>    next: value => {
>      console.log(value);
>    },
>    error: err => {
>      console.log("Error: " + err);
>    },
>    complete: () => {
>      console.log("complete");
>    }
>  });
>  
>  ```
>
>* ### `startWith`： 在observable 的一开始塞入发送的元素，同步发出
>
>  * 经常用来保存状态
>
>  ```js
>  var source = Rx.Observable.interval(1000);
>  var example = source.startWith(0);
>  ```
>
>* ### `merge`
>
>  ```js
>  import { interval, merge } from "rxjs";
>  import { take } from "rxjs/operators";
>  // merge 跟 concat 一样都是用来合并 oberservable;
>  // merge 多个 observable 同时处理
>  // merge 之后的 example 在时间上同时在跑 s1，s2 ，
>  // 当两件事同时发生时，会同步送出数据，，当两个observable 都结束时才会真都结束
>  // morge的逻辑有点像OR(||) 就是当两个observable其中一个被触发时都可以被处理
>  // 常用在一个以上当按钮具有部分相同行为
>  var s1 = interval(1000).pipe(take(3));
>  var s2 = interval(1000).pipe(take(6));
>  
>  merge(s1, s2).subscribe({
>    next: value => {
>      console.log(value);
>    },
>    error: err => {
>      console.log("Error: " + err);
>    },
>    complete: () => {
>      console.log("complete");
>    }
>  });
>  ```
>



# 四，Operators

> * ### combineLatest:
>
>   * 取各个observable 最后送出的值，再组成一个值输出
>
>   ```js
>   import { interval, combineLatest } from "rxjs";
>   import { take } from "rxjs/operators";
>
>   var s1 = interval(1000).pipe(take(3));
>   var s2 = interval(1000).pipe(take(6));
>
>   // s1,s2 只要有一个输出 就执行输出(且有过输出)，
>   // 如果 s1，s2 同时满足输出，就分两次输出，
>   // s1 第1次 => 0          s1 未输出 => null   结果不输出    s1 的 1s
>   // s1 第1次 => 0          s1 第1次 => 0       [ 0 , 0 ]   s2 的 1s
>   // s1 第2次 => 1          s1 第1次 => 0       [ 1 , 0 ]   s1 的 2s
>   // s1 第2次 => 1          s1 第2次 => 1       [ 1 , 1 ]   s2 的 2s
>   // s1 第3次 => 2          s1 第2次 => 1       [ 2 , 1 ]   s1 的 3s
>   // s1 第3次(最高3次) => 2  s1 第3次 => 2       [ 2 , 2 ]   s2 的 3s
>   // s1 第3次(最高3次) => 2  s1 第4次 => 3       [ 2 , 3 ]   s2 的 4s
>   // s1 第3次(最高3次) => 2  s1 第5次 => 4       [ 2 , 4 ]   s2 的 5s
>   // s1 第3次(最高3次) => 2  s1 第6次 => 5       [ 2 , 5 ]   s2 的 6s
>
>   var ex = combineLatest([s1, s2]).subscribe({
>     next: value => {
>       const [x1, x2] = value;
>       console.log(value);
>       console.log(x1 + x2);
>     },
>     error: err => {
>       console.log("Error: " + err);
>     },
>     complete: () => {
>       console.log("complete");
>     }
>   });
>
>   ```
>
> 
>
> * ### zip : 
>
>   * 因为zip必须cache住还没处理的元素，当两个observable 一个很快一个很慢，就会cache非常多的元素，等待比较慢的那个observable，所以不能乱用
>
>   ```js
>   
>   // 实现间隔 100ms 送出 'h', 'e', 'l', 'l', 'o'
>   import { interval, zip, from } from "rxjs";
>   import { take } from "rxjs/operators";
>   var s1 = from("hello");
>   var s2 = interval(300).pipe(take(6));
>   // 必须 s1,s2 都送出一个元素的时候就执行输出(不同步就需等待)，
>   var ex = zip(s1, s2).subscribe({
>     next: value => {
>       const [x1, x2] = value;
>       console.log(value);
>       console.log(x1 + x2);
>     },
>     error: err => {
>       console.log("Error: " + err);
>     },
>     complete: () => {
>       console.log("complete");
>     }
>   });
>   ```
>
> * ### withLatestFrom
>
>   * 运作方式和 combineLatest 有点像，只是它有主从关系，只有在主 observable 送出新的值时,才会订阅，附 obserbalbe 只有在主observable下运作
>   * 主，附 observable 都会一直执行发送值，只时不会订阅
>
>   ```js
>   import { interval, zip, from } from "rxjs";
>   import { map, withLatestFrom } from "rxjs/operators";
>   var main = zip(from("hello"), interval(500));
>   var some = zip(from([0, 1, 0, 0, 0, 1]), interval(300));
>   main
>     .pipe(
>       withLatestFrom(some),
>       map(([first, second]) => {
>         return second[0] === 1 ? first[0].toUpperCase() : first[0];
>       })
>     )
>     .subscribe({
>       next: value => {
>         console.log(value);
>       },
>       error: err => {
>         console.log("Error: " + err);
>       },
>       complete: () => {
>         console.log("complete");
>       }
>     });
>   
>   ```

## 运用之前的知识实现视频播放的拖动效果

```js
import { fromEvent } from "rxjs";
import {
  concatAll,
  filter,
  map,
  takeUntil,
  withLatestFrom
} from "rxjs/operators";
import "./style.css";
var video = document.getElementById("video");
var anchar = document.getElementById("anchar");
const scroll = fromEvent(document, "scroll");
const mouseDown = fromEvent(document.getElementById("video"), "mousedown");
const mouseUp = fromEvent(document, "mouseup");
const mouseMove = fromEvent(document, "mousemove");
mouseDown
  .pipe(
    filter(e => video.classList.contains("video-fixed")),
    map(e => mouseMove.pipe(takeUntil(mouseUp))),
    concatAll(),
    withLatestFrom(mouseDown),
    map(([move, down]) => {
      return {
        x: vaildValue(
          move.clientX - down.clientX,
          window.innerWidth - video.getBoundingClientRect().width,
          0
        ),
        y: vaildValue(
          move.clientY - down.clientY,
          window.innerHeight - video.getBoundingClientRect().height,
          0
        )
      };
    })
  )
  .subscribe({
    next: value => {
      console.log(value);
      video.style.top = value.y + "px";
      video.style.left = value.x + "px";
    },
    error: err => {
      console.log("Error: " + err);
    },
    complete: () => {
      console.log("complete");
    }
  });

scroll
  .pipe(
    map(e => {
      return anchar.getBoundingClientRect().bottom < 0;
    })
  )
  .subscribe({
    next: value => {
      if (value) {
        video.classList.add("video-fixed");
      } else {
        video.classList.remove("video-fixed");
      }
      console.log(value);
    },
    error: err => {
      console.log("Error: " + err);
    },
    complete: () => {
      console.log("complete");
    }
  });

function vaildValue(value, max, min) {
  return Math.min(Math.max(value, min), max);
}
```

