---
layout: post
title: Rust基础——结构体
categories:
- Rust
tags:
- Rust
---

## 结构体定义

`struct`是一个自定义数据类型，允许你包装和命名多个相关的值，从而形成一个有意义的组合

使用`struct`关键字来定义一个结构体

```rust
struct User {
    username: String,
    email: String,
    active: bool
}
```

创建结构体的三种方式

- 直接赋值初始化
- 初始化简化语法
- 结构体更新语法

```rust
// 创建结构体，在栈上分配内存
let user = User {
    username: String::from("username"),
    email: String::from("email"),
    active: true
};

// 使用字段初始化的简化语法
let username = String::from("username1");
let email = String::from("email");
let active = true;
let user1 = User { username, email, active};

// 结构体更新语法
let user2 = User {
    email: String::from("email"),  // 重新赋值字段
    ..user1  // 使用其他结构体的字段
},
```

注意，使用结构体更新语法时，等价于将其他结构体的字段赋值给新的结构体，赋值时会进行数据的移动和拷贝

例如以下代码，`user1.username`被移动到`user2.username`，`user1.active`被拷贝到`user2.active`，此时，`user1`失效

> 若在使用更新语法时，**发生了所有权移动**，则被用于更新的结构体在创建新的结构体后就不再有效，若没有发生所有权移动，则被用于更新的结构体依然有效
{: .prompt-info}

```rust
// 使用结构体更新语法等价于将其他结构体的字段赋值给新的结构体
let user2 = User {
    email: String::from("email"),
    username: user1.username,  // user1.username被移动到user2.username
    active: user1.active  // user1.active被拷贝到user2.active
};
```

## 元组结构体

在元组的基础上，可以创建特定元组类型的结构体，称为元组结构体

```rust
// 元组结构体即使元组内的类型相同，但依然是不同的类型，且与(i32, i32, i32)不兼容
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
```

元组结构体依然支持元组的解构和索引访问

```rust
let color = Color(0, 0, 0);
let red = color.0;  // 索引访问
// 解构赋值，注意(r, g, b)对应的类型是元组(_, _, _)，不是Color(i32, i32, i32)
let Color(r, g, b) = color;
```

## 类单元结构体

类单元结构体就是没有任何字段的结构体，通常用于需要在某个类型上实现`trait`，但不需要在类型中存储数据

```rust
struct Unit
let unit = Unit;  // 创建类单元结构体
```

## 方法语法

rust可以在结构体中实现方法，方法在`impl`块中实现，与相应的结构体关联

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 { self.width * self.height }
}
```

方法参数中的`&self`表示关联结构体的不可变引用，是`self: &Self`的简写，其中

- `self`是一个关键字，表示一个方法的接受者实例（receiver）
- `Self`表示关联结构体的类型，可以用在参数类型和返回值类型上

对于`self`和`Self`的用法，有以下三种情况

```rust
impl Rectangle {
    
    // 不使用引用，传入时会将关联结构体实例的所有权移动到方法内
    fn method1(self: Self) {
        // ...
    }
    
    // 使用不可变引用，可简写为&self
    fn method2(self: &Self) {
        // ....
    }
    
    // 使用可变引用，可简写为
    fn method3(self: &mut Self) {
        // ...
    }
}
```



