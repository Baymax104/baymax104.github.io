---
layout: post
title: Rust基础——泛型、Trait和生命周期
categories:
- Rust
tags:
- Rust
image:
  path: assets/img/rust/rust.png
---

## 泛型

**函数泛型**

```rust
fn foo<T>(arg: &T) -> &T {
    // ...
}
```

**结构体泛型**

```rust
struct Point<T> {
    x: T,
    y: T
}
```

**枚举泛型**

```rust
enum Option<T> {
    Some(T),
    None
}
```

**方法泛型**

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl <T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

>   **泛型单态化**
>
>   rust的泛型与java不同，java仅在编译期检查类型，之后进行类型擦除，而rust在编译时，会生成不同具体类型的代码，再将泛型定义替换为具体定义，避免了类型擦除问题
{: .prompt-tip}

## Trait

一个类型的行为由其可供调用的方法构成，如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了，Trait是一种将方法签名组合起来的方法，目的是定义一个实现某些目的所必需的行为的集合

