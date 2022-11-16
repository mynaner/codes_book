# Operators-scan,buffer

### scan

>scan类似于原生js Array的reduce方法

```js
var arr = [1,2,3,4];
var result = arr.reduce((origin,next)=>{ 
  // orgin 为上次计算后的值(第一次为 reduce第二个参数的值，如果么有第二参数，则为0)，
  // next 为arr中的数据
  // result 为计算完成的结果
  console.log(origin);
  return origin+next;
},0)
console.log(result);
```

```js
import { from, interval, zip } from "rxjs";
import { scan } from "rxjs/operators";
var source = zip(from("hello"), interval(600)).pipe(
  scan((origin, next) => {
    return origin + next[0];
  }, "")
);
source.subscribe({
  next: value => {
    console.log(value);
  },
  error: err => {
    console.log("Error: " + err);
  },
  complete: () => {
    console.log("complete");
  }
});
```



### buffer 是一个家族

>* buffer
>
>  ```js
>  //  把 source 送出的元素缓存在数组中，等到 source2 送出元素的时候，就会把缓存的数组发送出来
>  import { interval } from "rxjs";
>  import { buffer } from "rxjs/operators";
>  var source = interval(300);
>  var source2 = interval(1000);
>  var example = source.pipe(buffer(source2));
>  example.subscribe({
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
>  
>
>* bufferCount
>
>  ```js
>  
>  ```
>
>  
>
>* bufferTime
>
>* bufferToggle
>
>* bufferWhen

