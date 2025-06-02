---
layout: post
title: TypeScript基础
categories:
- 前端
tags:
- TypeScript
- 前端
typora-root-url: ./
image:
  path: /assets/img/typescript/10b88c68-typescript-logo.png
date: 2024-04-14 23:18 +0800
---
## 开始

TypeScript是JavaScript的超集，扩展了JavaScript的语法，在JavaScript的基础上增加了编译期的类型检查

全局安装TypeScript命令：`npm i -g typescript`

TypeScript需要先编译为JavaScript才能工作，编译命令为`tsc xxx.ts`

## 类型

### 类型声明

TS中声明变量，在变量名后加上类型

```ts
let a: number = 10;
```

若变量在声明时初始化，则进行类型推断，若未初始化且未声明类型，则默认为Any类型

```ts
// 未初始化且未声明类型，默认类型为Any
let a;
a = 10;
a = "Hello";
```

TS支持字面量类型声明

```ts
let a: 10;  // a的值只能是10
```

类型声明支持同时声明多个类型（联合类型）

```ts
let a: number | string;  // a只能是number类型或string类型
let b: "a" | "b";  // 配合字面量类型声明实现枚举
```

函数类型声明

-   支持参数类型声明
-   支持返回值类型声明
-   支持限定参数个数

```ts
function func(a: number, b: number): number {
    return a + b;
}
```

### 基本类型

-   number

-   string

-   boolean

-   any：任意类型

-   unknown：类型转换安全的any，在赋值给其他类型时要显式类型检查

    ```ts
    let a: unknown;
    let s: string;
    if (typeof a === "string") {
        s = a;
    }
    
    // 强制类型转换
    s = a as string;
    ```

-   void：函数无返回值

-   never：函数永远不会返回，类似kotlin的nothing

-   object：对象类型

    一个对象类型是通过属性名和属性类型确定

    ```ts
    let obj: { name: string, age: number }
    ```

    支持可选属性

    ```ts
    let obj: { name: string, age?: number }
    ```

    动态任意属性

    ```ts
    // property为任意属性名，属性声明为any类型
    let obj: { name: string, [property: string]: any }
    ```

    不同对象的属性混合

    ```ts
    // obj是个既有name属性又有age属性的对象类型
    let obj: { name: string } & { age: number };
    ```

-   函数类型

    使用类似kotlin的语法声明一个函数类型

    ```ts
    let f: (a: number, b: number) => number;
    
    // 函数赋值
    function fun(a: number, b: number): number {
        return a + b;
    }
    f = fun;
    f = (a, b) => a + b;  // 使用lambda表达式
    ```

-   array

    ```ts
    let arr: number[] = [1, 2, 3, 4];
    let arr1: Array<number> = [1, 2, 3, 4];
    ```

-   tuple：看做固定长度数组

    ```ts
    let t: [string, number] = ["Hello", 123];
    ```

-   enum：枚举类型

    ```ts
    enum Gender {
        Male,
        Female
    }
    ```

类型别名

```ts
type MyType = string;
type MyType1 = 1 | 2 | 3;
```

## tsc编译

使用`tsc --init`命令创建tsconfig.json文件，用于配置tsc编译

默认配置如下

```json
{
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

根对象配置

| 配置名        | 描述               |
| ------------- | ------------------ |
| `include: []` | 选择被编译的文件   |
| `exclude: []` | 选择排除编译的文件 |

compilerOptions配置

| 配置名             | 描述                               |
| ------------------ | ---------------------------------- |
| `target`           | 指定编译的JS版本                   |
| `module`           | 指定模块化方案                     |
| `lib:[]`           | 指定项目使用的库，一般不自定义设置 |
| `outDir`           | 指定编译输出目录                   |
| `outFile`          | 将编译结果输出为一个文件           |
| `allowJs`          | 是否允许编译js文件                 |
| `checkJs`          | 是否检查js代码                     |
| `removeComments`   | 是否去除注释                       |
| `noEmit`           | 不输出编译文件                     |
| `noEmitOnError`    | 当编译错误时，不输出编译文件       |
| `alwaysStrict`     | 编译结果是否使用严格模式           |
| `noImplicitAny`    | 是否不允许隐式any类型              |
| `noImplicitThis`   | 是否不允许不明确this               |
| `strictNullChecks` | 是否严格空检查                     |
| `strict`           | 是否启用所有严格检查               |

## 面向对象

### 类

与js类相似，基本声明如下

```ts
class Person {
    
    // 实例字段
    name: string;
    age: number = 20;
    
    // 静态字段
    static count: number = 0;
    
    // 只读字段
    readonly hello: number = 10;
    
    // 构造器
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
    
    // 实例方法
    say() {
        // ...
    }
}
```

字段封装

-   字段支持访问级别设置：public、private、protected

-   getter与setter方法与js相同

    ```ts
    // 可提供getter和setter
    getName() {
        return this.name
    }
    setName(name) {
        this.name = name
    }
    // 简化语法糖，可简化为属性访问形式
    get name() {
        return this.name
    }
    set name(name) {
        this.name = name
    }
    ```

### 继承

```ts
class Animal {
    name: string = "";
    
    sayHello() {
        // ...
    }
}

class Dog extends Animal {
    // 字段子类重写
    name: string = "Hello";
    
    constructor() {
        // 父类构造
        super();
    }
    
    // 方法子类重写
    sayHello() {
        // ...
        super.sayHello();  // 通过super引用父类
    }
}
```

### 抽象类

```ts
abstract class Person {
    // 抽象字段
    abstract name: string;
    
    // 抽象方法
    abstract say(): void;
}
```

### 接口

```ts
interface MyInterface {
    // 接口字段
    name: string;
    
    // 接口方法
    method(): void;
}

// 实现接口
class Person implements MyInterface {
    name: string = "";

    method(): void {
    }
}
```

## 泛型

### 泛型函数

泛型函数声明

```ts
function func<T>(a: T): T {
    return a;
}
```

限定泛型参数上界

```ts
function func<T extends A>() {
    // ...
}
```

### 泛型类

```ts
class Person<T> {
    name: T;
    constructor(name: T) {
        this.name = name;
    }
}
```

### 索引签名

可以通过字符串来访问对象的属性

```typescript
const obj: any = {
    value: "test"
}
obj["value1"] = "test1"

console.log(obj["value"])
```

在typescript中，索引签名必须是`string`或`number`类型

**显式指定索引签名**

```typescript
const foo: {
  [index: string]: { message: string };
} = {};

// 储存的东西必须符合结构
// ok
foo['a'] = { message: 'some message' };

// Error, 必须包含 `message`
foo['a'] = { messages: 'some message' };

// 读取时，也会有类型检查
// ok
foo['a'].message;

// Error: messages 不存在
foo['a'].messages;
```

与字段声明结合使用

```typescript
// ok
interface Foo {
  [key: string]: number;
  x: number;
  y: number;
}

// Error
interface Bar {
  [key: string]: number;
  x: number;
  y: string; // Error: y 属性必须为 number 类型
}
```

