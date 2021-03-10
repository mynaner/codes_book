# ts 应用(二)

参考地址 https://zhuanlan.zhihu.com/p/127738397

### 函数

* 注解函数

```typescript
// javascript
function add(x, y) {
  return x + y;
}
// typescript
// 有时候返回值也可以不写，TS 可以根据参数类型和函数体计算返回值类型，也就是俗称的自动推断类型机制。
function add(x: number, y: number): number {
  return x + y;
}
```

* 函数类型

```typescript
const add = function(x,y){
  return x+y;
}

// ts
const add: (x: number, y: number) => number = function (x, y) {
  return x + y;
};

```
* 可选参数

```typescript
function buildName(fristName: string, lastName?: string) {}
// 
buildName('Tom', 'Huang');
buildName('mRcfps');
```

* 重载
```typescript
// 重载（Overloads）是 TS 独有的概念，在 JS 中没有，它主要为函数多返回类型服务，具体来说就是一个函数可能会在内部执行一个条件语句，根据不同的条件返回不同的值，这些值可能是不同类型的

let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: { suit: string; card: number }[]): number;
function pickCard(x: number): { suit: string; card: number };
function pickCard(x): any {
  // 如果 x 是 `object` 类型，那么我们返回 pickCard 从 myDeck 里面取出 pickCard1 数据
  if (typeof x == "object") {
    let pickedCard = Math.floor(Math.random() * x.length);
    return pickedCard;
  }
  // 如果 x 是 `number` 类型，那么直接返回一个可以取数据的 pickCard2
  else if (typeof x == "number") {
    let pickedSuit = Math.floor(x / 13);
    return { suit: suits[pickedSuit], card: x % 13 };
  }
}
let myDeck = [
  { suit: "diamonds", card: 2 },
  { suit: "spades", card: 10 },
  { suit: "hearts", card: 4 },
];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);
let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);

```



* 交叉类型

```typescript
// 交叉类型就是多个类型，通过 & 类型运算符，合并成一个类型，这个类型包含了多个类型中的所有类型成员
interface ErrorHandling {
  success: boolean;
  error?: {
    message: string;
  };
}
interface ArtistsData {
  artists: { name: string }[];
}
const handleArtistsPresponse = (requset: ArtistsData & ErrorHandling) => {
  if (requset.error) {
    console.log(requset.error.message);
    return;
  }
  console.log(requset.artists);
};
// 我们的艺术家信息接口（Interface）是 ArtistsData ，它是请求成功之后返回的具体数据之一
// 除了这个，我们的响应体一般还有标志响应是否成功的状态，以及错误的时候的打印信息，
// 所以我们还定义了一个 ErrorHandling ，它们两个进行一个交叉类型操作就组成了我们的艺术家响应体：ArtistsData & ErrorHandling ，
// 然后我们在函数参数里面标志 response 为这个交叉类型的结果，并在函数体之类根据请求是否成功的状态 reponse.error 判断来打印对应的信息。
```

* 联合类型

```typescript
// 联合类型实际上是通过操作符 | ，将多个类型进行联合，组成一个复合类型，
// 当用这个复合类型注解一个变量的时候，这个变量可以取这个复合类型中的任意一个类型，
// 这个有点类似枚举了，就是一个变量可能存在多个类型，但是最终只能取一个类型。

const value: string = "hello Tuture";
const padding: string = "   ";

function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number ,got '${padding}'.`);
}
padLeft(value, 12);
```

* 字面量类型

```typescript
// 数字字面量
let tuture: 520;
tuture = 520; // 正确
tuture = 521; // 错误 Type '521' is not assignable to type '520
// 字符串字面量
let tuture: '520';
tuture = '520'; // 正确
tuture = '521'; // 错误 Type '"521"' is not assignable to type '"520"'

// 应用场景 
// 实现枚举 or 实现类型守卫

// 字面量和联合类型实现枚举效果
function getUserInfo(osType: "Linux" | "mac" | "windows") {}

// 正常枚举
enum EnumOsType {
  Linux,
  Mac,
  Windows,
}
function getUserInfo1(osType: EnumOsType) {}

// 类型守卫
// 类型守卫是我们 联合类型+字面量类型 的又一个应用场景，它主要用于在进行 ”联合“ 的多个类型之间，存在相同的字段，也存在不同的字段，然后需要区分具体什么时候是使用哪个类型
interface Linux {
  type: "Linux";
  linuxUserInfo: "极客";
}
interface Mac {
  type: "Mac";
  macUserInfo: "极客1";
}
interface Windows {
  type: "Windows";
  WindowsUserInfo: "极客2";
}
function getUserInfo(os: Linux | Mac | Windows) {
  // console.log(os.linuxUserInfo); // error
  switch (os.type) {
    case "Linux":
      console.log(os.linuxUserInfo);
      break;
    case "Mac":
      console.log(os.macUserInfo);
      break;
    case "Windows":
      console.log(os.WindowsUserInfo);
      break;
  }
}

```

