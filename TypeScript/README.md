# 数据类

## any 任意  类型

- 声明为 any 的变量可以赋予任意类型的值。

```TS
let num;  //等价于 let num:any;
```

## number 数据类型

可以用来表示

- 整数
- 浮点数
-  进制数（二进制，八进制，十进制，十六进制）

```TS
let num:number = 25;  //等价于 let num = 25;
```

## string 字符串类型

- 一个字符系列，使用单引号（'）或双引号（"）来表示字符串类型。
- 反引号（`）来定义多行文本和内嵌表达式。

```TS
let str:string = "你好;"; // let str = "你好！";
```

## boolean 布尔类型

- 表示逻辑值：true 和 false。
- 只能用 true 和 false 来表示，不能用 0 和 1 表示
- 在 if 判断中还是能用 1 和 0 来进行

```TS
let isLogin:boolean = false; // let isLogin = false;
```

## Array 数组类型

声明变量为数组。

```TS
// 在元素类型后面加[];
let arr:number[] = [1,2];
// 使用数字泛型
let arr:Array<number> = [1,2];
```

##  元组类型

- 元组类型用来 **_表示已知元素数量_** 和 **_类型的数组_**
- 各元素的类型不必相同，对应位置的类型需要相同。

```TS
let x:[string,number,number]=["name",1,1];
```

## enum 枚举

- 枚举类型用于定义数值集合。
-

```ts
let x: [string, number, number] = ["name", 1, 1]
enum Color {
  red,
  blue,
  yellow
}
console.log(Color)
let c: Color = Color.blue
console.log(c)

// blue: 100 red: 0 yellow: 101
let x: [string, number, number] = ["name", 1, 1]
enum Color {
  red,
  blue = 100,
  yellow
}
console.log(Color)
let c: Color = Color.blue
console.log(c) // 100
```

## function 函数

```ts
// 规定返回值数据类型 void 则没有返回值
function returnValue(): number {
  return 1
}
function returnValue(): void {
  // ...
}

//  参数类型
function returnValue(value: number, value2: number): number {
  return value + value2
}

// 规定函数类型
let myFun: (a: number, b: number) => number

myFun = returnValue
let num: number = myFun(2, 6)
```

# 对象 & type

## 对象

- 有点像元组类型

```ts
let dataObj: { name: string; num: number } = {
  name: "henny",
  num: 12
}
// 上面声明了，后面都要根据这个格式
dataObj = {
  name: "red",
  num: 134
}

let dataObj: { name: string; num: number }
// 上面声明了，后面都要根据这个格式
dataObj = {
  name: "henny",
  num: 12
}
dataObj = {
  name: "red",
  num: 134
}
```

- 复杂对象

```ts
let complex: { data: number[]; myFunc: (item: number) => void }
complex = {
  data: [100, 1, 44],
  myFunc: function(item: number): void {
    console.log(this.data)
  }
}
complex.myFunc(100)
```

## type 类型

```ts
//
type myType = { data: number[]; myFunc: (item: number) => void }
let complex: myType
complex = {
  data: [100, 1, 44],
  myFunc: function(item: number): void {
    console.log(this.data)
  }
}
complex.myFunc(100)
```

# null、undefined 和 never

```ts
let unionType: number | string | boolean = 13
unionType = "hello"
unionType = false

// 检查类型
let unionType: number | string | boolean = 13
unionType = "hello"
unionType = false
if (typeof unionType == "boolean") {
  console.log("boolean")
}

// null undefined

let myNull = null
myNull = undefined

// never
// never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。
// 这意味着声明为 never 类型的变量只能被 never 类型所赋值，
// 在函数中它通常表现为抛出异常或无法执行到终止点（例如无线循环）
```

# 类

```ts
class Person {
  // public 公开，外部可以调用
  public name: string
  // protected 当前类或被继承类使用
  protected gender: string
  // private 只能在当前类使用
  private age: number = 27

  // 构造函数
  constructor(name: string, gender: string) {
    this.name = name
    this.gender = gender
  }

  setAge(age: number) {
    this.age = age
  }
  getAge(): number {
    return this.age
  }
}
const person = new Person("张三", "男")
console.log(person)
console.log(person.name)
person.setAge(26)
console.log(person.getAge())
```

```ts
// 继承
class Student extends Person {
  studentId: number
  constructor(name: string, gender: string, studentId: number) {
    super(name, gender)
    this.studentId = studentId
  }
}
const student = new Student("张", "男", 1214)
console.log(student)
```

## get set 修饰词

- 用于隔离私有属性和公开属性

```ts
class Person {
  private name: string = ""
  private age: number = 0

  set setName(name: string) {
    this.name = name
  }
  get getName(): string {
    return this.name
  }
  set setAge(age: number) {
    this.age = age
  }
  get getAge(): number {
    return this.age
  }
}
const person = new Person()
console.log(person)
person.setName = "张"
console.log(person.getName)
```

## static 静态属性，静态方法

- 可以不用实例化就调用(实例化后也能调用)

```ts
class Person {
  static PI = 3.1415926
  static fn(num: number): number {
    return this.PI * num
  }
}
console.log(Person.PI)
console.log(Person.fn(5))
```

# namespace 命名空间

- 只能访问 export 的变量或方法

```js
const PI = 3.1415926;
namespace MyMath{
  const PI = 3.14;
  export function fn(num:number):number{
    return PI*num;
  }
  export function fn1():void{
    console.log("fn1 方法被调用了")
  }
}
console.log(PI);
console.log(MyMath.fn(10));
```

## 文件  拆分

> tsc --outfile app.js fn.ts fn1.ts app.ts

> tsc --outfile 编译成文件 被编译文件 被编译文件 被编译文件

```js
// fn.ts
namespace MyMath{
  const PI = 3.14;
  export function fn(num:number):number{
    return PI*num;
  }
}

// fn1.ts
namespace MyMath{
  export function fn1():void{
    console.log("fn1 方法被调用了")
  }
}

// app.ts
MyMath.fn1()
console.log(MyMath.fn(10));
```

## 多重命名空间及文件引入

> tsc app.ts --outFile app.js

```js
// fn.ts
namespace MyMath{
  export namespace Circle{
    const PI = 3.14;
    export function fn(num:number):number{
      return PI*num;
    }
  }
}
// fn1.ts
namespace MyMath{
  export function fn1():void{
    console.log("fn1 方法被调用了")
  }
}
// app.ts
// 引入文件
/// <reference path="fn.ts" />
/// <reference path="fn1.ts" />
MyMath.fn1();
console.log(MyMath.Circle.fn(10));
```

# module 模块

```ts
// fn.ts
export const PI = 3.14
export function fn(num: number): number {
  return PI * num
}

// app.ts
import { PI, fn } from "./stuff/fn"
console.log(PI)
```

编译

> tsc --module commonjs app.ts

 浏览器还是不能直接识别

需要引入 systemjs；

模块

```js
expert
// 需要 {} 或者 * as name
import {PI,fn} from "./stuff/fn";
import  * as name from "./stuff/fn";



export default
//
import fn from "./stuff/fn";
```

# 接口

- 和 type 很相似
- interface 可以继承，type 不行
- ?: 可以不传

```ts
interface Person {
  name: string
  age: number
  sex?: string // 可以不传
  readonly salary: number // 只读，不能修改,
  [propName: string]: any // 任意名字 任意类型(任意key，任意val)
  greet(): void
}

type Person1 = { name: string; age: number }

let person: Person = {
  name: "张",
  age: 12,
  greet(): void {
    console.log()
  }
}
```

# 接口继承及类的实现

```TS
interface PersonInterface{
    name:string;
    age:number;
    sex?:string;
    readonly salary:number;
    [propName:string]:any;
    greet():void;
}
interface StudentInterface{
    id:number;
    course:string;
}
class People implements PersonInterface,StudentInterface{
    name:string = "张";
    age:number = 24;
    salary:number = 7000;
    id:number = 1;
    course:string = "it";
    greet():void{
        console.log(this.name);

    }
}
```

## 接口继承

```ts
interface PersonInterface {
  name: string
  age: number
  sex?: string
  readonly salary: number
  [propName: string]: any
  greet(): void
}
interface StudentInterface extends PersonInterface {
  id: number
  course: string
}
class People implements StudentInterface {
  name: string = "张"
  age: number = 24
  salary: number = 7000
  id: number = 1
  course: string = "it"
  greet(): void {
    console.log(this.name)
  }
}
```

# Generic 函数 泛型

- 函数中使用泛型

```ts
function fn<T>(arg: T): T {
  // t代表泛型
  return arg
}
// 推断类型
console.log(fn(1))
// 明确类型
console.log(fn<string>("zhang"))
```

- 接口中使用泛型

```ts
interface GenericInterface {
  <T>(tar: T): T
}
function fn<T>(arg: T): T {
  return arg
}
let myFn: GenericInterface = fn
// 推断类型
console.log(myFn(1))
// 明确类型
console.log(myFn<string>("zhang"))
// 泛型提升
interface GenericInterface<T> {
  (tar: T): T
}
function fn<T>(arg: T): T {
  return arg
}
let myFn: GenericInterface<number> = fn
// 推断类型
console.log(myFn(1))
```

- 泛型添加约束

```ts
function fn<T extends { length: number }>(arg: T): number {
  return arg.length
}
let obj = {
  name: "wang",
  length: 14
}
fn(obj)
```

## 泛型运用到类

```ts
class CountNumber<T> {
  // class CountNumber<T extends number>{
  number1: T
  number2: T
  constructor(num1: T, num2: T) {
    this.number1 = num1
    this.number2 = num2
  }
  count(): number {
    return +this.number1 + +this.number2
  }
}
const count = new CountNumber<string>("100", "20") // const count = new CountNumber<number>(100,20);
console.log(count.count())
```
