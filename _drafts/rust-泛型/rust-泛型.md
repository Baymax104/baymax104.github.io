---
layout: post
title: Rust基础——泛型和Trait
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

// 显式指定泛型参数
foo::<i32>(...);
```

**结构体泛型**

```rust
struct Point<T> {
    x: T,
    y: T
}

// 显式指定泛型参数
let point: Point<i32> = Point::<i32> { x: 0, y: 0 };
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

// 必须在impl之后加上<T>，表示这是泛型参数
impl <T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// 为指定泛型参数实现方法
// 只有Point<i32>类型的实例可以调用实现的方法
impl Point<i32> {
    fn foo(&self) {}
}

let point = Point { x: 0, y: 0 };  // 类型推断为Point<i32>
point.foo();
```

### const泛型

const泛型是使用**值**而不是类型作为泛型参数，需要指定值的类型

```rust
// 一个N维坐标点类型
// N是一个const泛型参数，用于指定index数组的类型
struct Point<T, const N: usize> {
    index: [T; N],
}

// 两个变量是不同的类型
let p1: Point<i32, 2> = Point::<i32, 2> { index: [1, 2] };
let p2: Point<i32, 3> = Point::<i32, 3> { index: [1, 2, 3] };
```

作为函数的泛型参数时，可以作为常量用于函数体中

```rust
// const泛型参数可以用于函数中
fn foo<const N: usize>(arr: &[i32; N]) {
    for i in 0..N {
        println!("{}", arr[i])
    }
}

fn foo1<const N: usize>(point: Point<i32, N>) {
    // ...
}

// 显式指定const泛型参数，实现限制数组参数的长度
let arr = [1, 2, 3];
foo::<3>(&arr);  // 必须传入长度为3的数组

let point = Point { index: [1, 2, 3] };  // 自动推断
foo1::<3>(&point);  // 必须传入Point<i32, 3>类型
```

const泛型参数可以传入一个常量表达式，使用大括号包围

```rust
let arr = [1, 2, 3];
foo::<{ 2 + 1 }>(&arr);  // 必须传入长度为3的数组
```

**常量函数**

使用`const`修饰一个函数，表示它是常量函数，常量函数会在编译期执行，将计算的常量值结果在调用处替换

```rust
fn foo<const N: usize>(arr: &[i32; N]) {
    for i in 0..N {
        println!("{}", arr[i])
    }
}

const fn calculate_size() -> usize { 2 + 1 }

let arr = [1, 2, 3];
foo::<{ calculate_size() }>(&arr);  // 在编译期得出foo的参数类型为&[i32; 3]
```

### 泛型的性能

rust的泛型与java不同，java仅在编译期检查类型，之后进行类型擦除，在运行时会丢失类型信息，而rust在编译时，会生成不同具体类型的代码，再将泛型定义替换为具体定义，避免了类型擦除问题，这称为**单态化**

例如，使用Option枚举，单态化会为每种具体类型生成具体代码

```rust
let integer = Some(5);
let float = Some(5.0);

// 单态化生成以下代码
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

## Trait

一个类型的行为由其可供调用的方法构成，如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了，Trait是一种包含多个方法的集合，类似于接口

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

为类型实现Trait

```rust
pub struct NewsArticle

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        // ...
    }
}

let news = NewsArticle;
news.summarize();
```

Trait可以有默认实现

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

### Trait Bound

可以将函数的参数指定为实现了某个Trait的类型

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

// 指定参数和返回值必须实现了Summary Trait
pub fn notify(item: &impl Summary) -> &impl Summary { item }
```

以上实际是Trait Bound的语法糖

```rust
// 使用Trait Bound可以更加灵活地指定impl Summary是同一个类型还是不同的类型

// 限制必须是实现Summary的同一个类型
fn notify<T: Summary>(item: &T) -> &T { item }
// 等价于fn notify1(item: &impl Summary) -> &impl Summary { item }
fn notify1<T: Summary, U: Summary>(item: &T) -> &U { item }
```

在实现方法时指定Trait Bound可以有条件地实现方法

```rust
struct Pair<T> {
    x: T,
    y: T,
}

// 为T类型实现了PartialOrd Trait的Pair<T>实现方法
impl<T: PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

**多个泛型参数的语法糖**

使用`+`指定多个Trait

```rust
// 参数需要实现Summary和Display两个Trait
fn notify(item: &(impl Summary + Display)) {
    // ...
}

// Trait Bound
fn notify1<T: Summary + Display>(item: &T) {
    // ...
}
```

使用`where`关键字简化Trait Bound

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
    // ...
}

// 在where语句中统一设置Trait Bound
fn some_function1<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

### Trait Object

当在返回值类型上使用Trait Bound时，rust不允许返回不同类型的对象，即使它们满足Trait Bound

例如下面的代码，编译无法通过

```rust
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        // 返回Post类型
        Post {
           // ...
        }
    } else {
        // 返回Weibo类型
        Weibo {
            // ...
        }
    }
}
```

编译无法通过是因为rust无法在编译期确定返回值类型，函数可能返回Post类型，也可能返回Weibo类型，只能在运行时确定

当代码中引入多态时，需要一种机制确定实际调用的类型，这称为**分发**，分发分为静态分发和动态分发

- 静态分发：在编译期确定实际类型
- 动态分发：在运行时确定实际类型

rust默认使用单态化生成具体类型代码，在编译期就确定了实际类型，即实现了静态分发。但对于上述代码无法在编译期确定实际类型，只能在运行时确定实际类型的情况，就需要使用动态分发

从类型的角度看，一个Trait是包含了具有某种特性的类型的集合，**Trait本身也可以看做一个类型**，将Trait作为类型的特性在rust中通过Trait Object实现，同时rust通过Trait Object实现了动态分发

例如，通过Trait Object在Vec中保存不同的类型，定义以下Trait和类型

```rust
trait A {}

struct AImpl1;
struct AImpl2;
impl A for AImpl1 {}
impl A for AImpl2 {}
```

现在我们需要在一个Vec中同时保存`AImpl1`和`AImpl2`，如果我们将Trait当做java中的接口，我们会很自然地写出如下代码

```rust
let v: Vec<&A> = vec![&AImpl1, &AImpl2];
```

这段代码会被编译器报错，因为这里没有使用Trait Object，使用Trait Object需要加上`dyn`关键字，以下代码可以编译通过

```rust
let v: Vec<&dyn A> = vec![&AImpl1, &AImpl2];
```

这里指定Vec的泛型时，我们将`A`看做了一个类型，而Trait本身并不是类型，此时就需要Trait Object来实现Trait的类型特性（私以为可以将Trait Object理解为编译时类型，实现了Trait的类型为运行时类型）

**实现多态的效果**

通过Trait Object，可以实现多态中父类类型引用子类对象的效果

```rust
trait A {
    fn a(&self);
}

trait B {
    fn b(&self);
}

// 在B的Trait Object上实现A Trait
impl A for dyn B {
    fn a(&self) {
        println!("A");
    }
}

struct BImpl;

// 在BImpl上实现B Trait
impl B for BImpl {
    fn b(&self) {
        println!("B");
    }
}

fn main() {
    // 父类类型引用子类对象
    let b: &dyn B = &BImpl;
    b.a();
    b.b();
}
```

以上代码类似下面的java代码

```java
interface A {
    void a();
}

interface B extends A {
    default void a() {
        System.out.println("A");
    }
    void b();
}

class BImpl implements B {
    @Override
    public void b() {
        System.out.println("B");
    }
}

public static void main(String[] args) {
    B b = new BImpl();
    b.a();
    b.b();
}
```

当我们使用Trait Object时，我们也就使用了动态分发，编译器会在运行时才确定Trait的实际类型，下面的代码可以编译通过

```rust
struct BImpl1;

impl B for BImpl1 {
    fn b(&self) {
        println!("B1");
    }
}

fn main() {
    let mut b: &dyn B = &BImpl;
    b.a();
    b.b();
    // b的实际类型在运行时确定，可以改变b的实际类型
    b = &BImpl1;
    b.b();
}
```

需要注意的是，Trait Object不能作为值类型出现，下面的代码无法编译通过

```rust
// 无法编译通过
fn foo(a: dyn A) {
    // ...
}

let b: dyn B = BImpl;  // 无法编译通过
```

这是因为Trait Object的实际类型可以是任意实现了该Trait的类型，**rust无法在编译期确定实际类型的大小**，无法在栈上分配内存，因此需要引用或者智能指针

**Trait Object的限制**





### 动态分发

