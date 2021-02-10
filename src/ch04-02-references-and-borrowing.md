## References and Borrowing 引用与借出

The issue with the tuple code in Listing 4-5 is that we have to return the
`String` to the calling function so we can still use the `String` after the
call to `calculate_length`, because the `String` was moved into
`calculate_length`.

例 4-5 的问题是我们必须返回 `String`，因为在调用 `calculate_length` 函数后还需要使用它，
但是 `String` 又被 move 到了 `calculate_length`。

Here is how you would define and use a `calculate_length` function that has a
reference to an object as a parameter instead of taking ownership of the
value:

以下你可以以“引用（reference）”作为参数使用 `calculate_length` 函数，而不是把参数的 ownership 交给函数：

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

First, notice that all the tuple code in the variable declaration and the
function return value is gone. Second, note that we pass `&s1` into
`calculate_length` and, in its definition, we take `&String` rather than
`String`.

经过修改，首先变量声明的元组没了，函数的 return 值没了，然后，注意我们传入 `&s1` 到 `calculate_length`，
在函数定义时，我们使用 `&String` 而不是 `String`。

These ampersands are *references*, and they allow you to refer to some value
without taking ownership of it. Figure 4-5 shows a diagram.

这些“与”符号就是**引用（reference）**，借助它你可以引用一个值而不用转移它的 ownership，如图 4-5 所示。

<img alt="&String s pointing at String s1" src="img/trpl04-05.svg" class="center" />

<span class="caption">Figure 4-5: A diagram of `&String s` pointing at `String
s1`</span>

> Note: The opposite of referencing by using `&` is *dereferencing*, which is
> accomplished with the dereference operator, `*`. We’ll see some uses of the
> dereference operator in Chapter 8 and discuss details of dereferencing in
> Chapter 15.
> 注意：`&` 引用的反面是通过 `*` 进行的**解引用（dereferencing）**。
我们将在第八章使用解引用，第十五章深入讨论解引用的细节。

Let’s take a closer look at the function call here:

我们关注一下函数的内容：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

The `&s1` syntax lets us create a reference that *refers* to the value of `s1`
but does not own it. Because it does not own it, the value it points to will
not be dropped when the reference goes out of scope.

`&s1` 允许我们创建一个对 `s1` 的引用，而不用转移所有权，
也就因为没有转移在所有权，在引用离开作用域时并不会被 drop。

Likewise, the signature of the function uses `&` to indicate that the type of
the parameter `s` is a reference. Let’s add some explanatory annotations:

同样的，函数签名的 `&` 表明参数 `s` 是一个引用。我们加上一些注释吧：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

The scope in which the variable `s` is valid is the same as any function
parameter’s scope, but we don’t drop what the reference points to when it goes
out of scope because we don’t have ownership. When functions have references as
parameters instead of the actual values, we won’t need to return the values in
order to give back ownership, because we never had ownership.

`s` 所在的作用域跟之前提到的作用域没什么区别，但是引用目标不会被 drop，因为根本没有 ownership。
当函数获取的是 reference 而不是实际的值，我们便不用在函数结束后把值返回回去，因为它从未拥有过这个变量。

We call having references as function parameters *borrowing*. As in real life,
if a person owns something, you can borrow it from them. When you’re done, you
have to give it back.

我们把 reference 做为函数参数称为**借出（borrowing）**。
就像在现实生活中别人拥有一件物品，你可以借过来，使用完了后，必须还回去。

So what happens if we try to modify something we’re borrowing? Try the code in
Listing 4-6. Spoiler alert: it doesn’t work!

那么我们想要修改已经 borrow 的东西会怎样？
试试例 4-6 的代码。（剧透一下，这必是不行的！）

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

<span class="caption">例 4-6：尝试修改一个借出了的值</span>

Here’s the error:

报错如下：

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Just as variables are immutable by default, so are references. We’re not
allowed to modify something we have a reference to.

默认情况下变量时不可修改的，引用也是。Rust 不允许我们修改被引用的东西。

### Mutable References 可变引用

We can fix the error in the code from Listing 4-6 with just a small tweak:

我们可以通过一些微小的调整，修复例 4-6 的错误：

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

First, we had to change `s` to be `mut`. Then we had to create a mutable
reference with `&mut s` and accept a mutable reference with `some_string: &mut
String`.

首先，为 `s` 添加 `mut`，然后通过 `&mut s` 创建可变引用，再修改函数签名为接收可变引用 `some_string: &mutString`。

But mutable references have one big restriction: you can have only one mutable
reference to a particular piece of data in a particular scope. This code will
fail:

不过可变引用有一个限制：在一个作用域里，对某一个数据只能存在一个引用。否则，代码会出问题：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

Here’s the error:

报错如下：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

This restriction allows for mutation but in a very controlled fashion. It’s
something that new Rustaceans struggle with, because most languages let you
mutate whenever you’d like.



The benefit of having this restriction is that Rust can prevent data races at
compile time. A *data race* is similar to a race condition and happens when
these three behaviors occur:



* Two or more pointers access the same data at the same time.
* At least one of the pointers is being used to write to the data.
* There’s no mechanism being used to synchronize access to the data.



Data races cause undefined behavior and can be difficult to diagnose and fix
when you’re trying to track them down at runtime; Rust prevents this problem
from happening because it won’t even compile code with data races!



As always, we can use curly brackets to create a new scope, allowing for
multiple mutable references, just not *simultaneous* ones:



```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

A similar rule exists for combining mutable and immutable references. This code
results in an error:



```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Here’s the error:

报错如下：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Whew! We *also* cannot have a mutable reference while we have an immutable one.
Users of an immutable reference don’t expect the values to suddenly change out
from under them! However, multiple immutable references are okay because no one
who is just reading the data has the ability to affect anyone else’s reading of
the data.



Note that a reference’s scope starts from where it is introduced and continues
through the last time that reference is used. For instance, this code will
compile because the last usage of the immutable references occurs before the
mutable reference is introduced:

```rust,edition2018
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

The scopes of the immutable references `r1` and `r2` end after the `println!`
where they are last used, which is before the mutable reference `r3` is
created. These scopes don’t overlap, so this code is allowed.

Even though borrowing errors may be frustrating at times, remember that it’s
the Rust compiler pointing out a potential bug early (at compile time rather
than at runtime) and showing you exactly where the problem is. Then you don’t
have to track down why your data isn’t what you thought it was.

### Dangling References 悬空引用

In languages with pointers, it’s easy to erroneously create a *dangling
pointer*, a pointer that references a location in memory that may have been
given to someone else, by freeing some memory while preserving a pointer to
that memory. In Rust, by contrast, the compiler guarantees that references will
never be dangling references: if you have a reference to some data, the
compiler will ensure that the data will not go out of scope before the
reference to the data does.

在有指针的语言，很容易会错误地创建**悬空指针（dangling pointer）**。因为释放了内存而指针依然指向同样的地方，
悬空指针指向的内存可能已经给其他变量占用。在 Rust 中，编译器保证了 reference 绝不会成为悬空引用：
如果你引用了一些数据，编译器会保证这些数据在 reference 离开作用域前仍然可用。

Let’s try to create a dangling reference, which Rust will prevent with a
compile-time error:

试试“造”一个悬空引用，会发现 Rust 报编译错误：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

Here’s the error:

报错如下：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

This error message refers to a feature we haven’t covered yet: lifetimes. We’ll
discuss lifetimes in detail in Chapter 10. But, if you disregard the parts
about lifetimes, the message does contain the key to why this code is a problem:

报错信息指出的问题我们还没讨论过：生命周期，我们将在第十章讨论它。
现在，即使你忽略报错里生命周期的部分，也还是能看出一些问题：

```text
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.
```

Let’s take a closer look at exactly what’s happening at each stage of our
`dangle` code:

看看我们的“悬空”代码每一步都发生了什么：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

Because `s` is created inside `dangle`, when the code of `dangle` is finished,
`s` will be deallocated. But we tried to return a reference to it. That means
this reference would be pointing to an invalid `String`. That’s no good! Rust
won’t let us do this.

`s` 在 `dangle` 中创建，当 `dangle` 运行完，`s` 会被释放。但我们尝试去返回它的引用，
这意味着这个引用指向一个无效的 `String`，这可不好！Rust 不允许这种事情发生。

The solution here is to return the `String` directly:

解决方案是直接返回 `String`：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

This works without any problems. Ownership is moved out, and nothing is
deallocated.

完美，ownership 被转移到外层，什么都不会被释放。

### The Rules of References 引用的规则

Let’s recap what we’ve discussed about references:

来总结一下 reference 吧：

* At any given time, you can have *either* one mutable reference *or* any
  number of immutable references.
* References must always be valid.

* 你可以**选择** 1 个可变引用**或**多个不变引用
* Reference 必须总是有效的

Next, we’ll look at a different kind of reference: slices.

接着，我们学习另一种引用：slice
