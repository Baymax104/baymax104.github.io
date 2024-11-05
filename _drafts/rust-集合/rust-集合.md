---
layout: post
title: Rust基础——常用集合
categories:
- Rust
tags:
- Rust
---

## 开始

本章节介绍rust中的三个常用集合

- vector：长度可变的数组
- String
- HashMap

## vector

Vector表示一个长度可变的数组，是一个类型为`Vec<T>`的结构体

**创建vector**

使用`new()`关联函数创建一个空的vector

```rust
let v: Vec<i32> = Vec::new();
```

使用初始值创建vector时，使用`vec!`宏，支持类型推断

```rust
let v = vec![1, 2, 3];
```

> `vec`的内存分配与String类似，在栈上保存一个固定大小的结构体，其中包含一个指向存储元素的堆内存的指针，具有所有权
{: .prompt-info }

**访问vector**

有两种方式可以访问vector元素

- 索引访问：索引越界时会抛出panic

  ```rust
  let v = vec![1, 2, 3];
  let a: i32 = v[0];  // 发生数据移动或拷贝
  let b: &i32 = &v[0];  // 获取元素的不可变引用，遵循借用期规则
  let s: &[i32] = &v[0..2];  // slice引用
  
  let mut v1 = vec![1, 2, 3];
  let c: &mut i32 = &mut v1[0];  // 获取元素的可变引用，遵循借用期规则
  ```

  注意，访问时应该使用引用，**直接访问会发生数据的移动或拷贝**
- `get`方法：返回`Option<T>`，数组越界时返回`None`

  ```rust
  let v = vec![1, 2, 3];
  let a = v.get(0);
  match a {
      Some(n) => {}
      None => {}
  }
  ```

**更新vector**

使用更新vector的方法时，会隐式传入vector的可变引用，因此需要声明变量为可变的

- 添加元素

  ```rust
  let mut v = vec![1, 2, 3];
  v.push(1);  // 添加元素
  ```

- 修改元素

  ```rust
  let mut v = vec![1, 2, 3];
  v[0] = 1;
  
  // 通过引用修改
  let a: &mut i32 = &mut v[0];  // 获取可变引用
  *a = 2;  // 解引用后获取引用的值，可访问或修改
  ```

- 删除元素

  ```rust
  let mut v = vec![1, 2, 3];
  let removed = v.remove(0); // 移除第一个元素并返回它
  ```

**遍历vector**

不可变引用遍历

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{i}");
}
```

可变引用遍历

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

## String

**创建字符串**

```rust
let empty = String::new();  // 创建一个空字符串
let from = String::from("hello");  // 使用字面量创建字符串对象
```

**添加字符串**

```rust
let s = String::new();
let s1 = String::from("world");

// push_str接收一个&str类型参数
s.push_str("hello");  // 添加字符串字面量
s.push_str(&s1);  // 附加其他字符串，不移动所有权

// push接受一个char类型参数
s.push('a');
```

**拼接字符串**

```rust
// 使用+运算符拼接两个字符串
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意s1被移动了，不能继续使用

// 使用format!宏拼接字符串
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{s1}-{s2}-{s3}");  // 传入引用，不移动所有权
```

> 注意，使用`+`运算符实际是调用了`add`方法，它的函数签名为`fn add(self, s: &str) -> String`，左操作数传入值，会**发生所有权移动**，右操作数传入`&str`引用，可以接收String引用和字符串字面量

**遍历字符串**

由于rust内对于存储的字符串字节支持多种方式来解释（字节、Unicode标量值等），因此字符串**不支持索引访问**

遍历字符串需要使用字符数组或字节数组的迭代器访问

```rust
// 字符数组迭代器
for c in "hello".chars() {
    println!("{c}");
}

// 字节数组迭代器
for c in "hello".chars() {
    println!("{c}");
}
```

## HashMap



