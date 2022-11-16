# ts 应用

参考地址 https://zhuanlan.zhihu.com/p/126342731

* ts 原始数据类型

```typescript
/* 
number
string
boolean
null
undefined
void
symbol
bigint 
*/
// 提示 其中前六种是 ES5 中就有的，symbol 从 ES6 开始引入，bigint 是 ES2020 新引进的。
const tutureSlogan:string = "haha"
const tutureSlogan:string = 5201314 // 报错 Type '5201314' is not assignable to Type 'string'
```

* 非原始类型(也称为object类型)

```typescript
// array 数组
const arr:string[] = ["1","2","3"]
// tuple 
const arr:[string,number,string] =["1",2,"3"] 
// enum 枚举
```

* 特殊类型

```typescript
// any 任何类型
// 主要用于在编码的时候不知道一个变量的类型，所以先给它加一个 any 类型定义，表示它可以是任何类型，一般留待后续确认此变量类型之后再将 any 改为具体的类型。
let demand:any;

// unknown 类型
// unknown 类型和 any 都可以表示任何类型，应用场景也和上面类型，但是它更安全
// any 类型的变量是可以进行任意进行赋值、实例化、函数执行等操作，
// 但是 unknown 只允许赋值，不允许实例化、函数执行等操作
let demandOne: any;
let demandTwo: unknown;
demandOne = 'Hello, Tuture'; // 可以的
demandTwo = 'Hello, Ant Design'; // 可以的
demandOne.foo.bar() // 可以的
demandTwo.foo.bar() // 报错
// unknown 类型只允许赋值操作，不允许对象取值（Getter) 、函数执行等操作，所以它更安全。




// never 在TS中代表不存在的值类型，一般用于给函数进行类型声明，函数绝不会有返回值的时候使用，比如函数内抛出错误
// 给不会有返回值的函数 
function responseError(message: string): never {
  // ... 具体操作，接收信息，抛出错误
}
```

* 枚举和接口

```typescript
// interface
interface Todo {
  content: string;
  readonly user: string; //只读属性
  time?: string; // 可选属性
  isCompleted: boolean;
  // 多余属性检查
  [propName:string]:any;
}
const todo:Todo={
  //...
}


// Enum 枚举 
enum UserId {
  tuture, // 默认 0
  mRcfps, // 1
  crxk,   // 2
  pftom,  // 3
  holy    // 4
}
interface Todo {
  content: string;
  user: UserId; // 枚举应用
  time: string;
  isCompleted: boolean;
}

// 数字枚举
enum UserId {
  tuture, // 0
  mRcfps = 6,
  crxk, // 7
  pftom,// 8
  holy,// 9
}

// 字符串枚举
enum UserId {
  tuture = '66666666',
  mRcfps = '23410977',
  crxk = '25455350',
  pftom = '23410976',
  holy = '58352313',
}

// 异构枚举
enum UserId {
  tuture = '66666666',
  mRcfps = 6,
}
```

