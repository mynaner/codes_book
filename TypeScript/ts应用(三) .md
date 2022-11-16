# ts 应用(三)

参考地址 https://zhuanlan.zhihu.com/p/131393833

* 类型别名

```typescript
// TS 为我们提供了类型别名，它允许你为类型创建一个名字，这个名字就是类型的别名，进而你可以在多处使用这个别名，并且有必要的时候，你可以更改别名的值（类型），以达到一次替换，多处应用的效果

// JavaScript 
function getName(n) {
  if (typeof n === 'string') {
    return n;
  } else {
    return n();
  }
}

// typescript
type Name = string;
type NameResolver = () => string;
type NameParams = Name | NameResolver;
function getName(n: NameParams): Name {
  return "";
}
```

* 类型别名与接口

```typescript
Interface(接口) 和 Type alias(类型别名)
* 相同点
// 描述一个对象
interface UserOP {
  name: string;
  email: string;
  isBig: boolean;
  age: number;
}
type UserTy = {
  name: string;
  age: number;
};
// 描述一个函数
interface setUser {
  (name: string, email: string, isBig: boolean, age: number): UserOP;
}
type setUserTy = (name: string, age: number) => void;
// 创建一个变量是 UserOP的类型
let pikaqiu: UserOP;
pikaqiu = { name: "zynn", email: "re", isBig: false, age: 24 };
// 
let pikaqiu1: UserTy;
pikaqiu1 = {
  name: "cs",
  age: 12,
};
// 用来描述一个方法
let mySearchXX: setUser;
mySearchXX = function (
  name: string,
  email: string,
  isBig: boolean,
  age: number
): UserOP {
  return pikaqiu;
};
//
let mySearchXXTy: setUserTy;
mySearchXXTy = (name: string, age: number) => {};


// interface extends interface (接口继承接口)
interface UserName {
  name: string;
}
interface User extends UserName {
  age: number;
} 
// type extends type (类型继承类型)
type UserName = {
  name: string;
};
type User = UserName & { // 通过交叉类型实现继承
  age: number;
}; 
// interface extends type (接口继承类型)
type UserName = {
  name: string;
};
interface User extends UserName {
  age: number;
} 
// type extends interface (类型继承接口)
interface UserName {
  name: string;
}
type User = UserName & {
  age: number;
};


* 不同点

// type 可以声明基本类型别名，联合类型，元组等类型，interface 不行

// interface 能够声明合并，type 不行
```

* 类

```ts
// ES6 类的组成
*--> 构造函数
*--> 属性
*--> 实例属性
*--> 静态属性
*--> 方法
*--> 实例方法
*--> 静态方法
// TS 注解
*--> 注解构造函数
*--> 注解属性：
*--> 访问限定符： public/protected/private
*--> 修饰符：readonly
*--> 注解方法
*--> 访问限定符：public/protected/private
```

```ts

```

