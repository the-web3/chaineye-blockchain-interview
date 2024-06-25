# rust


## 1.详细说明 async-std，tokio，actix，native-tls，rustls 等的区别与联系

- async-std 和 tokio
async-std 和 tokio 都是 Rust 中用于异步编程的库。它们的主要区别在于，async-std 更注重提供类似于 Rust 标准库的 API 和使用习惯，而 tokio 则提供了更为灵活的 API 和自定义能力。在实际使用中，如果需要快速上手，可以选择 async-std；如果需要更高的灵活性和自定义能力，可以选择 tokio。

- tokio 和 actix
tokio 是一个异步编程框架，提供了用于异步 I/O 操作的基础设施，例如事件循环和异步任务的执行器。而 actix 是一个基于 tokio 的异步 Web 框架，提供了 Web 开发的相关功能和工具，例如 HTTP 服务器和路由等。

- native-tls 和 rustls
native-tls 和 rustls 都是 Rust 中用于 TLS 加密和解密的库。它们的主要区别在于，native-tls 使用操作系统的本地 TLS 库，例如 OpenSSL，而 rustls 是一个纯 Rust 实现的 TLS 库。在实际使用中，如果需要更高的性能和对操作系统 TLS 库的依赖度较低，可以选择 rustls；如果需要更广泛的兼容性和对操作系统 TLS 库的支持，可以选择 native-tls。

总的来说，async-std、tokio、actix、native-tls、rustls 这些库和框架都是 Rust 中非常常用的工具，用于异步编程和网络编程。它们之间的区别和联系需要根据具体的使用场景和需求来选择。


## 第一课：Rust语言基础知识部分
建议阅读的文档：
[九层之台，起于累土](https://mp.weixin.qq.com/s/irowOHVMUpY_u1kxJ5gZ6Q)

### 1. 简单说明可变变量以及不可变变量

*解答：* 
在Rust中，变量默认是不可变，如果想要一个变量可变必须要在声明变量的时候显式地加上`mut`标志。这也就是在Rust中声明变量被称之为`绑定`变量的原因。其主要目的是为了考虑安全，变化是安全的重要源头之一。
```Rust
let a = 100;
// a += 1; //放开这行代码，无法通过编译，因为变量`a`默认不可变。

let mut b = 100; // 用`mut`对变量`a` 作显式标浅
b = b + 1; // ok
```

### 2. 什么叫变量遮蔽（shadow）？请举例说明。
解答：
所谓的变量遮蔽（shadow），是指在同一上下文中，可以对已经声明或定义的变量进行重复的声明或定义，新的变量声明或定义后，原变量失效。新的变量与原变量的关系是，新变量仅仅在名称上与原变量相同，这就意味着新变量的类型可以不同于原变量。
```Rust
let s = String:from("abc");
println!("{s}");

let s = 100; //此处，对原变量s进行了遮蔽；
s += 20;
...
```

### 3. 什么是常量？它在定义时有什么要要求？常量与变量的区别？
解答：
1. 常量，是指程序整个的生命周期内都是不可变的量，用来表示全局范围内可被多处使用的不变量，比如一个星期有7天这种永恒的量。常量通常是在编译时就被包含在程序二进制文件中，程序加载内在后也会被单独存放在类型常量或者是静态的内存区域，该内存区域是只读的。
2. 常量在定义时，不适应类型的自动推断，必须显式地标注其类型。
   ```Rust
   //const DAYS_IN_WEEK = 7; //Error
   const DAYS_IN_WEEK: u8 = 7; // ok
   ```
3. 常量与变量的区别： 变量，通常是在函数内部的局部空间进行定义的，然后在运行的时候，实时地分配在栈空间里面。可见，变量的默认不可变性设计，主要是出于对安全的考究，其与常量的目的与内存管理机制都有着本质区别。

### 4. Rust中的基本（标量）数据类型都有哪些？
解答：
Rust中的基本数据类型，主要有`整数`、`浮点数`、`字符`、`布尔`、`元组`、`数组`等。
#### 1. 整型

分为有符号整数与无符号整数，有符号数用`u`开头，无符号数用`i`开头，同时有长度信息（8、16、32、64、128...），比如`i32`表示有符号且长度为32位的整数，`u32`表示无符号且长度为32的整数。
  
  * 在绑定变量的时候，如果不显式的指定整数的类型，编译器会自动推断为`i32`类型。
  * 还有一种特殊的整型:`usize` 与 `isize`，其长度与目标机器的字长长度一致。如果是32机器，其长度是32位，如果是64位机器，其长度是64位。
  * 不同长度的类型在进行转换时，要考虑到安全性问题。一般来说，把一个长度大的数转换成长度小的类型是不安全的，因为发生一部分二进制位被丢失的情况（即所谓的窄化），相反情况是安全的。

#### 2. 浮点数
分为32位与64位，分别为`f32`与`f64`。如果未显式指定类型，编译器会自动推断为`f64`类型。

#### 3.字符
字符的类型用`char`声明，当右值为字面量的时候，用单引号包裹，如果是双引号会被认为是字符串。
* 字符内部的数值编码是`UniCode`，它所表达的范围特别广，不但可以表达键盘上可打印的字符，还包括各种表情符号、中文、韩文等等。
* 一个字符占用4个字节

#### 4.布尔
布尔类型，用`bool`表示，用于描述逻辑上的真假值，有`true`和`false`两种值。
* 占用一个字节
* bool类型是一个独立的类型，与其他类型之间的转换关系

#### 5. 元组类型
元组是将多个类型的多个值组合到一个复合类型中的一种基本方式。元组的长度是固定的，声明后无法增长或缩小。
```Rust
let tup: (i32, f64, u8) = (500,6.4,1); // 定义
println!("1st number is {}", tup.0); // 访问其中的分量
let (x, y, z) = tup;
println!("The value of y is: {}", y);
```

没有任何值的元组是一种特殊的类型，写作：`()`，该类型称作为`单元类型`，如果一个表达式没有返回值或者函数没有没有返回值，都被隐式地返回单元类型。

#### 6. 数组
将多个类型相同的元素依次组合在一起，就是一个数组。
关于数组需要注意的点：
* 长度固定，一经定义就不可以改变
* 元素必须有相同的类型
* 元素之间，在物理上和逻辑上都是依次线性排列的
* 数组是存储在栈上
* 数组的长度也是类型一可或缺的部分，`[u8;3]` 和 `[u8;4]` 是不同的类型。

```Rust
let arr: [i32;5] = [1, 2, 3, 4, 5];
println!("the 3th element of arr is : {}", arr[2]);

let mut arr = [0, 0, 0, 0];
arr[0] = 100;
println!("{:?}", arr);
```

### 5. 编写一个函数，对一个i32类型的数组进行冒泡排序
要求：根据传入的布尔参数决定是升序还是降序。

解答：
```Rust
fn bubble_sort(arr: &mut [i32], ascending: bool)  {
    let len = arr.len();
    for i in 0..len - 1 {
        for j in 0..len - i - 1 {
            if (ascending && arr[j] > arr[j + 1]) || (!ascending && arr[j] < arr[j + 1]) {
                /*
                let tmp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = tmp;
                 */
                arr.swap(j, j + 1);
            }
        }
    }
}

#[test]
    fn test_bubble_sort() {
        let mut arr1 = [5, 3, 8, 6, 2];
        bubble_sort(&mut arr1, true);
        println!("升序排序: {:?}", arr1);

        let mut arr2 = [5, 3, 8, 6, 2];
        bubble_sort(&mut arr2, false);
        println!("降序排序: {:?}", arr2);
    }
```

## 第二课： 所有权与借用
### 6. 掌握所有权与借用的内涵，建议阅读以下文章：
[欲知山中事，须问打柴人](https://mp.weixin.qq.com/s/2aM6TQprVoZAjY0PY75g4A)


[不以规矩，不能成方圆](https://mp.weixin.qq.com/s/kUDc9xr_a-51RCmg_pg-FA)

### 7. 分析下面三段代码，回答能不能通过编译？并分别说明原因。
#### 1. 代码一
```Rust
fn main() {
  let i1 = 100;
  let i2 = i1;
  println!("{}",i1);
}
```
#### 2. 代码二
```Rust
fn main() {
  let s1 = String:from("abc");
  let s2 = s1;
  println!("{}",s1);
}
```

#### 3. 代码三
```Rust
fn f(s: String) {}
fn main() {
  let s1 = String:from("abc");
  f(s1);
  println!("{}",s1);
}
```

### 8. 下面代码会不会通过编译？并分别说明原因。
#### 1. 代码一
```Rust
fn main() {
  let ref_a: &i32;
  let b = *a;
  println!("{}", b);
}
```

#### 2. 代码二
```Rust
fn dangle() -> &i32 {
  let a = 100;
  &a
}

fn main() {
  let ref_some = dangle();
  println!("{}", *ref_some);
}
```

#### 3. 代码三
``` Rust
fn main() {
  let mut  a = 100;
  let ref_a1 = &a;
  println!("{}", *ref_a1);
  let ref_a2 = &a;
  println!("{}", *ref_a2);
  let ref_a3 = &a;
  println!("{}", *ref_a3);

  let ref_mut_a1 = &mut a;
  println!("{}", *ref_mut_a1);

  println!("{}", *ref_a2); //如果把这行代码注释了，又是什么情况？
}
```

## 第三课：结构体、枚举及常用集合类型
### 9. 结构体和枚举
#### 9.1 有一个名为 Person 的结构体，包含以下字段：

* name（字符串类型）
* age（整数类型）
* email（字符串类型）

#### 要求：
* 定义这个结构体
* 为该结构体实现关联函数`new`，用以构建实例
* 对其字段分别实现`get` 和 `set` 关联方法。

解答：
```Rust
// 1. 定义结构体 Person
#[derive(Debug, Clone)]
struct Person {
    name: String,
    age: u8,
    email: String,
}
// 实现相关的方法和函数
impl Person {
    fn new(name: String, age: u8, email: String) -> Self {
        Self{name, age, email}
    }

    fn set_name(&mut self,name: String) {self.name = name;}
    fn get_name(&self) -> &str {self.name.as_str()}

    fn set_age(&mut self, age: u8) {self.age = age;}
    fn get_age(&self) -> u8 {self.age}

    fn set_email(&mut self, email: String) {self.email = email}
    fn get_email(&self) -> &str {self.email.as_str()}
}


```

#### 9.2 有一个名为 Status 的枚举，表示一个任务的状态，包含以下变体：

* NotStarted
* InProgress（包含一个表示进度的浮点数）
* Completed


#### 要求：
* 定义该枚举类型
* 写一个关联方法`send_message`，返回类型为`String`,使用 match 根据三种不同的变体打印相关的信息。对于InProgress类型，打印的信息中需要包含进度信息。

解答：
```Rust
// 定义枚举 Status
enum Status {
    NotStarted,
    InProgress(f64),
    Completed,
}

impl Status {
    fn send_message(status: Status) ->String {
        match status {
            Status::NotStarted => "还没有开始".to_string(),
            Status::InProgress(progress) => format!("正在进行中，当前进度为：{}", progress),
            Status::Completed => "已经完成了".to_string(),
        }
    }
}
```
### 10. Option和match
#### 10.1 定义一个函数 find_person_by_email，接受一个 Vec<Person> 和一个 String 作为参数，返回一个 Option<Person>，表示在列表中找到的第一个匹配的Person。如果没有找到，返回 None。
解答：
```Rust
fn find_person_by_email(people: Vec<Person>, email: String) -> Option<Person> {
    for person in people {
        if person.email == email {
            return Some(person);
        }
    }
    None
}
```

#### 10.2 定义一个函数print_person_info，使用 match 表达式处理 Option<Person>，如果找到，打印出 Person 的信息，如果没有找到，打印出一条消息表示没有找到。
解答：

```Rust
fn print_person_info(person_option: Option<Person>) {
    match person_option {
        Some(person) => println!("Found person: {:?}", person),
        None => println!("Person not found"),
    }
}
```
### 11. String、Vec和HashMap
#### 11.1 定义一个函数 group_people_by_age，接受一个 Vec<Person> 作为参数，返回一个 HashMap<u32, Vec<Person>>，将 Person 按照年龄进行分组。
解答：
```Rust
// 定义函数 group_people_by_age
use std::collections::HashMap;
fn group_people_by_age(people: Vec<Person>) -> HashMap<u8, Vec<Person>> {
    let mut age_groups: HashMap<u8, Vec<Person>> = HashMap::new();
    for person in people {
        age_groups.entry(person.age).or_insert(Vec::new()).push(person);
    }
    age_groups
}
```
#### 11.2 使用 String 类型完成以下任务：

定义一个函数 to_uppercase_names，接受一个 Vec<Person> 作为参数，返回一个新的 Vec<String>，其中包含所有 Person 的大写名字。
解答：
```Rust
// 定义函数 to_uppercase_names
fn to_uppercase_names(people: Vec<Person>) -> Vec<String> {
    let mut uppercase_names = Vec::new();
    for person in people {
        uppercase_names.push(person.name.to_uppercase());
    }
    uppercase_names
}
```

## 第四课 泛型与特征
### 12. 实现一个简单的通知系统，能够发送不同类型的通知消息（如邮件通知、短信通知、推送通知），并且能够存储和处理这些不同类型的通知。要求如下：

* 定义一个 Notification 特征，包含一个 send 方法，用于发送通知。
* 实现三个结构体 EmailNotification、SmsNotification 和 PushNotification，它们分别代表邮件通知、短信通知和推送通知，并为这些结构体实现 Notification 特征。
* 创建一个 NotificationSender 结构体，该结构体可以存储多个不同类型的通知，并能够调用它们的 send 方法来发送所有的通知。
* 使用泛型和特征对象来实现 NotificationSender 的通知存储。


参考答案：
```Rust
// 定义 Notification 特征
trait Notification {
    fn send(&self);
}

// 实现 EmailNotification 结构体
struct EmailNotification {
    recipient: String,
    subject: String,
    body: String,
}

impl Notification for EmailNotification {
    fn send(&self) {
        println!("Sending email to {}: {}\n{}", self.recipient, self.subject, self.body);
    }
}

// 实现 SmsNotification 结构体
struct SmsNotification {
    phone_number: String,
    message: String,
}

impl Notification for SmsNotification {
    fn send(&self) {
        println!("Sending SMS to {}: {}", self.phone_number, self.message);
    }
}

// 实现 PushNotification 结构体
struct PushNotification {
    device_id: String,
    message: String,
}

impl Notification for PushNotification {
    fn send(&self) {
        println!("Sending push notification to {}: {}", self.device_id, self.message);
    }
}

// 定义 NotificationSender 结构体，使用泛型
struct NotificationSender<T: Notification> {
    notifications: Vec<T>,
}

impl<T: Notification> NotificationSender<T> {
    fn new() -> Self {
        NotificationSender {
            notifications: Vec::new(),
        }
    }

    fn add_notification(&mut self, notification: T) {
        self.notifications.push(notification);
    }

    fn send_all(&self) {
        for notification in &self.notifications {
            notification.send();
        }
    }
}

// 使用特征对象的版本
struct DynNotificationSender {
    notifications: Vec<Box<dyn Notification>>,
}

impl DynNotificationSender {
    fn new() -> Self {
        DynNotificationSender {
            notifications: Vec::new(),
        }
    }

    fn add_notification(&mut self, notification: Box<dyn Notification>) {
        self.notifications.push(notification);
    }

    fn send_all(&self) {
        for notification in &self.notifications {
            notification.send();
        }
    }
}

fn main() {
    // 使用泛型的 NotificationSender
    let mut email_sender = NotificationSender::new();
    email_sender.add_notification(EmailNotification {
        recipient: String::from("user@example.com"),
        subject: String::from("Welcome!"),
        body: String::from("Thank you for signing up!"),
    });

    let mut sms_sender = NotificationSender::new();
    sms_sender.add_notification(SmsNotification {
        phone_number: String::from("123-456-7890"),
        message: String::from("Your code is 1234."),
    });

    email_sender.send_all();
    sms_sender.send_all();

    // 使用特征对象的 DynNotificationSender
    let mut dyn_sender = DynNotificationSender::new();
    dyn_sender.add_notification(Box::new(EmailNotification {
        recipient: String::from("user@example.com"),
        subject: String::from("Welcome!"),
        body: String::from("Thank you for signing up!"),
    }));
    dyn_sender.add_notification(Box::new(SmsNotification {
        phone_number: String::from("123-456-7890"),
        message: String::from("Your code is 1234."),
    }));
    dyn_sender.add_notification(Box::new(PushNotification {
        device_id: String::from("device123"),
        message: String::from("You have a new message."),
    }));

    dyn_sender.send_all();
}

```

## 第五课 闭包与生命周期
### 13. 编写一个函数 apply_twice，它接受一个闭包和一个整数，并将该闭包应用两次。闭包接受并返回一个整数。你需要正确地定义 apply_twice 函数的类型签名，并实现该函数。
```Rust
fn apply_twice<F>(f: F, x: i32) -> i32
where
    F: Fn(i32) -> i32,
{
    // 在此实现
}

// 使用示例
fn main() {
    let add_two = |x| x + 2;
    let result = apply_twice(add_two, 5);
    println!("{}", result); // 应输出 9
}

```

### 14. 编写一个函数 generate_adder，它接受一个整数参数 x，并返回一个闭包。该闭包接受一个整数参数 y，并返回 x + y。注意，闭包必须捕获外部变量 x。
```Rust
fn generate_adder(x: i32) -> impl Fn(i32) -> i32 {
    // 在此实现
}

// 使用示例
fn main() {
    let adder = generate_adder(5);
    println!("{}", adder(3)); // 应输出 8
}

```

### 15. 编写一个函数 longest_with_an_announcement，它接受两个字符串切片和一个闭包作为参数。该闭包接受一个字符串切片，并返回一个字符串切片。函数返回两个字符串切片中较长的一个，并调用闭包来打印一个公告。
```Rust
fn longest_with_an_announcement<'a, F>(x: &'a str, y: &'a str, ann: F) -> &'a str
where
    F: Fn(&str) -> &str,
{
    // 在此实现
}

// 使用示例
fn main() {
    let x = "Hello, world!";
    let y = "Hi, Rust!";
    let announcement = |s: &str| {
        println!("Announcement: {}", s);
        s
    };
    let result = longest_with_an_announcement(x, y, announcement);
    println!("The longest string is {}", result); // 应输出 "Hello, world!"
}

```

## 第六课 项目管理与错误处理
### 15. 实现一个简单的计算器库
#### 问题：编写一个名为 calculator_workspace 的工作区项目，其中包含两个库包：

1. calculator_core：核心计算逻辑。
2. calculator_io：用于处理输入输出及错误管理。
calculator_core 包含基本的加减乘除功能，并在计算过程中可能返回自定义错误。

calculator_io 包含读取用户输入和输出结果的功能，并将 calculator_core 中的错误转换为用户友好的错误信息。

#### 项目结构
```css
calculator_workspace/
├── Cargo.toml
├── calculator_core/
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
├── calculator_io/
│   ├── Cargo.toml
│   └── src/
│       └── lib.rs
└── tests/
    └── integration_test.rs

```
