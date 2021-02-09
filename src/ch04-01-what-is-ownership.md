## 什么是 Ownership？

Rust’s central feature is _ownership_. Although the feature is straightforward
to explain, it has deep implications for the rest of the language.

**Ownership** 是 Rust 的重要特性。尽管这个特性解释起来不难，但是它深远地影响着这个语言的每一部分。

All programs have to manage the way they use a computer’s memory while running.
Some languages have garbage collection that constantly looks for no longer used
memory as the program runs; in other languages, the programmer must explicitly
allocate and free the memory. Rust uses a third approach: memory is managed
through a system of ownership with a set of rules that the compiler checks at
compile time. None of the ownership features slow down your program while it’s
running.

所有程序都需要解决他们运行时如何使用计算机内存的问题。
一些语言有 GC，GC 会在程序运行时自动寻找可以释放的内存，
在另一些语言，程序员需要显式分配和释放内存。
而 Rust 使用第三种方式：在编译时，ownership 系统借助编译器的一系列检查规则，管理内存。
任何 ownership 特性都不会让你的程序运行变慢。

Because ownership is a new concept for many programmers, it does take some time
to get used to. The good news is that the more experienced you become with Rust
and the rules of the ownership system, the more you’ll be able to naturally
develop code that is safe and efficient. Keep at it!

因为 ownership 对很多程序员来说都是一个新概念，这需要花点时间去习惯它。
好消息是，你对 Rust 和 ownership 系统的规则越熟悉，
越能写自然地写出安全高效的代码。加油！

When you understand ownership, you’ll have a solid foundation for understanding
the features that make Rust unique. In this chapter, you’ll learn ownership by
working through some examples that focus on a very common data structure:
strings.

当你理解 ownership，你就有了理解让 Rust 独一无二特性的基础。
在这一章，你将通过一些十分常见的数据结构——字符串，来学习 ownership。

> ### The Stack and the Heap 栈和堆
>
> In many programming languages, you don’t have to think about the stack and
> the heap very often. But in a systems programming language like Rust, whether
> a value is on the stack or the heap has more of an effect on how the language
> behaves and why you have to make certain decisions. Parts of ownership will
> be described in relation to the stack and the heap later in this chapter, so
> here is a brief explanation in preparation.
>
> 在很多语言中，大多数时候你都不需要钻研栈和堆。但在 Rust 这样的语言，
> 一个值存在栈或堆会对语言的行为产生影响，所以你不需要在此作出选择。
> 这章后面的 ownership 部分会描述它与堆和栈的关系，这里仅是提前做些简单的解释。
>
> Both the stack and the heap are parts of memory that are available to your
> code to use at runtime, but they are structured in different ways. The stack
> stores values in the order it gets them and removes the values in the
> opposite order. This is referred to as _last in, first out_. Think of a stack
> of plates: when you add more plates, you put them on top of the pile, and
> when you need a plate, you take one off the top. Adding or removing plates
> from the middle or bottom wouldn’t work as well! Adding data is called
> _pushing onto the stack_, and removing data is called _popping off the stack_.
>
> 堆和栈都是你的代码在运行时可以调用的一部分内存。但是他们的结构有所不同。
> 栈的储存顺序与获取顺序相同，与删除顺序相反，这就是所谓**后进先出**。
> 就像一叠盘子，放盘子的时候你会放在那叠盘子的最上方，而取盘子的时候你会拿走最上面的一个。
> 你想在中间和底部加盘子都很不方便！添加数据我们称为**推入栈中**，取出数据称为**弹出栈**。
>
> All data stored on the stack must have a known, fixed size. Data with an
> unknown size at compile time or a size that might change must be stored on
> the heap instead. The heap is less organized: when you put data on the heap,
> you request a certain amount of space. The memory allocator finds an empty
> spot in the heap that is big enough, marks it as being in use, and returns a
> _pointer_, which is the address of that location. This process is called
> _allocating on the heap_ and is sometimes abbreviated as just _allocating_.
> Pushing values onto the stack is not considered allocating. Because the
> pointer is a known, fixed size, you can store the pointer on the stack, but
> when you want the actual data, you must follow the pointer.
>
> 储存在栈的数据必须有一个已知且固定的长度，而在编译时未知长度，或长度会改变的数据，储存在堆中。
> 堆相对栈没有那么强的组织性：数据放入堆时，你申请了一些空间。内存分配器找到一些空位，足够你使用，将其标记为已使用，
> 然后返回一个指向该地址的**指针**。这个过程称为**在堆中分配内存**，有时缩写为**分配（allocating）**。
> 推入栈的操作不属于分配，因为那个指针是已知的，固定长度的，你可以在栈中储存指针，但你想获取实际数据，就必须跟随指针寻找。
>
> Think of being seated at a restaurant. When you enter, you state the number of
> people in your group, and the staff finds an empty table that fits everyone
> and leads you there. If someone in your group comes late, they can ask where
> you’ve been seated to find you.
>
> 想想你去餐厅的情况。当你进入餐厅时，你说明了一桌多少人，然后服务员找到合适的桌子并带你到那里。
> 如果你的伙伴们晚了来，他们就可以问出你在哪并找到你。
>
> Pushing to the stack is faster than allocating on the heap because the
> allocator never has to search for a place to store new data; that
> location is always at the top of the stack. Comparatively, allocating space
> on the heap requires more work, because the allocator must first find
> a big enough space to hold the data and then perform bookkeeping to prepare
> for the next allocation.
>
> 推入栈比堆分配更快，因为总会存在栈顶，分配器不需要寻找存放新数据的位置，而在堆分配空间就需要更多的工作，
> 因为分配器必须找到一块足够大的地方存放数据，并进行记账以备下次分配。
>
> Accessing data in the heap is slower than accessing data on the stack because
> you have to follow a pointer to get there. Contemporary processors are faster
> if they jump around less in memory. Continuing the analogy, consider a server
> at a restaurant taking orders from many tables. It’s most efficient to get
> all the orders at one table before moving on to the next table. Taking an
> order from table A, then an order from table B, then one from A again, and
> then one from B again would be a much slower process. By the same token, a
> processor can do its job better if it works on data that’s close to other
> data (as it is on the stack) rather than farther away (as it can be on the
> heap). Allocating a large amount of space on the heap can also take time.
>
> 访问堆里的数据比访问栈数据要慢，因为你要跟随指针获取数据。如果不在内存中反复横跳，近代处理器会更快一些。
继续刚才的比喻，餐厅服务员给很多桌客人下单，最快的方法就是一张桌子全都下完单，再到下一张桌子，
如果先给 A 下单一个菜，然后给 B 下单，又再回到 A，就会非常缓慢。同样的，处理器可以更快地当前数据临近的数据
（就像栈这样的结构），而远的就会变慢（堆）。在堆中分配一大块空间同样消耗时间。
>
> When your code calls a function, the values passed into the function
> (including, potentially, pointers to data on the heap) and the function’s
> local variables get pushed onto the stack. When the function is over, those
> values get popped off the stack.
>
> 当你的代码调用函数，值会被传入函数（也有可能时指向堆数据的指针），
然后函数的本地变量会被推入栈中，函数运行结束，这些变量会被弹出。
>
> Keeping track of what parts of code are using what data on the heap,
> minimizing the amount of duplicate data on the heap, and cleaning up unused
> data on the heap so you don’t run out of space are all problems that ownership
> addresses. Once you understand ownership, you won’t need to think about the
> stack and the heap very often, but knowing that managing heap data is why
> ownership exists can help explain why it works the way it does.
> 
> 

### Ownership 规则

First, let’s take a look at the ownership rules. Keep these rules in mind as we
work through the examples that illustrate them:

首先，我们看看 Ownership 的几条规则，记住这些规则，再看看下面详细介绍的例子：

- Each value in Rust has a variable that’s called its _owner_.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

- Rust 里的每个值都有一个叫**owner**的变量。
- 一个值同时只能有一个 owner
- owner 离开作用域时，值将被抛弃

### Variable Scope 变量作用域

We’ve walked through an example of a Rust program already in Chapter 2. Now
that we’re past basic syntax, we won’t include all the `fn main() {` code in
examples, so if you’re following along, you’ll have to put the following
examples inside a `main` function manually. As a result, our examples will be a
bit more concise, letting us focus on the actual details rather than
boilerplate code.



As a first example of ownership, we’ll look at the _scope_ of some variables. A
scope is the range within a program for which an item is valid. Let’s say we
have a variable that looks like this:

来看看 ownership 的第一个例子，变量的**作用域**。
作用域是在程序里的一个范围，在这个范围内，变量是有效的。
举个例子，我们有一个变量：

```rust
let s = "hello";
```

The variable `s` refers to a string literal, where the value of the string is
hardcoded into the text of our program. The variable is valid from the point at
which it’s declared until the end of the current _scope_. Listing 4-1 has
comments annotating where the variable `s` is valid.

变量 `s` 指向一个字符串字面量，这个值是写死在程序中的。
这个变量从声明到当前**作用域**结束，是可用的。例 4-1 的注释说明了 `s` 的使用范围。

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

<span class="caption">例 4-1：变量与它的有效作用域</span>

In other words, there are two important points in time here:

换言之，有两个重点：

- When `s` comes _into scope_, it is valid.
- It remains valid until it goes _out of scope_.

- `s` 进入作用域，那么它是可用的
- 直到脱离当前作用域，一直可用

At this point, the relationship between scopes and when variables are valid is
similar to that in other programming languages. Now we’ll build on top of this
understanding by introducing the `String` type.

暂时为止，作用域和变量是否可用的关系与其他语言基本一样。现在我们在此基础上，介绍 `String` 类型。

### `String` 类型

To illustrate the rules of ownership, we need a data type that is more complex
than the ones we covered in the [“Data Types”][data-types]<!-- ignore -->
section of Chapter 3. The types covered previously are all stored on the stack
and popped off the stack when their scope is over, but we want to look at data
that is stored on the heap and explore how Rust knows when to clean up that
data.

说明 ownership 规则，我们需要一种更复杂的数据格式。
第三章中，那些类型都储存于栈中，在离开作用域时，弹出栈，但我们现在要研究储存于堆中的数据，
并搞清楚 Rust 如何清理这些数据。

We’ll use `String` as the example here and concentrate on the parts of `String`
that relate to ownership. These aspects also apply to other complex data types,
whether they are provided by the standard library or created by you. We’ll
discuss `String` in more depth in Chapter 8.

例子中我们使用 `String`，并关注 `String` 与 ownership 的关系。它的表现与其他复杂数据类型类似，
不管这些数据由标准库提供还是你自己创建的。我们将会在第八章深入讨论 `String`。

We’ve already seen string literals, where a string value is hardcoded into our
program. String literals are convenient, but they aren’t suitable for every
situation in which we may want to use text. One reason is that they’re
immutable. Another is that not every string value can be known when we write
our code: for example, what if we want to take user input and store it? For
these situations, Rust has a second string type, `String`. This type is
allocated on the heap and as such is able to store an amount of text that is
unknown to us at compile time. You can create a `String` from a string literal
using the `from` function, like so:

上面已经使用过字符串字面量，一个字符串值写死到程序中。字符串字面量是约定好的，他们不能适应各种情况。
原因就是他们是不变的（immutable）。不是所有字符串值都是确定的，例如我们想获取用户输入然后储存起来要怎么办？
要应付这种情况，Rust 有第二种字符串类型，`String`。这个类型分配堆空间，可以存放一些我们在编译时未确定的文本。
你可以通过 `from` 函数由字符串字面量创建 `String`，就像这样：

```rust
let s = String::from("hello");
```

The double colon (`::`) is an operator that allows us to namespace this
particular `from` function under the `String` type rather than using some sort
of name like `string_from`. We’ll discuss this syntax more in the [“Method
Syntax”][method-syntax]<!-- ignore --> section of Chapter 5 and when we talk
about namespacing with modules in [“Paths for Referring to an Item in the
Module Tree”][paths-module-tree]<!-- ignore --> in Chapter 7.

双冒号（`::`）操作符让你在 `String` 命名空间下使用 `from` 函数，而不是通过 `string_from` 之类的方法。
我们会在第五章 [“Method Syntax”][method-syntax]<!-- ignore --> 部分讨论更多语法，
在第七章 [“Paths for Referring to an Item in the Module Tree”][paths-module-tree]<!-- ignore -->
讨论命名空间和模块化。

This kind of string _can_ be mutated:

这种字符串是**可以**被修改的：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

So, what’s the difference here? Why can `String` be mutated but literals
cannot? The difference is how these two types deal with memory.

那么这里面有什么不同？为什么 `String` 可以被修改但是字面量不可以呢？
其中缘由就是他们在内存中的两种不同处理方式。

### Memory and Allocation 内存与分配

In the case of a string literal, we know the contents at compile time, so the
text is hardcoded directly into the final executable. This is why string
literals are fast and efficient. But these properties only come from the string
literal’s immutability. 
Unfortunately, we can’t put a blob of memory into the
binary for each piece of text whose size is unknown at compile time and whose
size might change while running the program.

在字符串字面量的例子中，我们在编译时就确定了内容，所以文本直接写死到可执行文件。
这就是文本字面量快速高效的原因。但这些好处来源于字符串字面量的不变性。
我们不能把大小未知或在运行时会遭到修改的文本编译为二进制文件。

With the `String` type, in order to support a mutable, growable piece of text,
we need to allocate an amount of memory on the heap, unknown at compile time,
to hold the contents. This means:

使用 `String` 支持可变的文本，需要在堆中分配一部分内存来放置这些内容，其分配大小在编译时是未知的，也就是：

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we’re
  done with our `String`.

- 分配器需要在运行时请求内存
- 我们需要在使用完 `String` 归还那部分内存

That first part is done by us: when we call `String::from`, its implementation
requests the memory it needs. This is pretty much universal in programming
languages.

第一部分已经在调用 `String::from` 的时候完成了，执行后它获得了它需要的内存。
这在编程语言中很常见。

However, the second part is different. In languages with a _garbage collector
(GC)_, the GC keeps track and cleans up memory that isn’t being used anymore,
and we don’t need to think about it. Without a GC, it’s our responsibility to
identify when memory is no longer being used and call code to explicitly return
it, just as we did to request it. Doing this correctly has historically been a
difficult programming problem. If we forget, we’ll waste memory. If we do it
too early, we’ll have an invalid variable. If we do it twice, that’s a bug too.
We need to pair exactly one `allocate` with exactly one `free`.

然而第二部分就有点不一样了。在有**垃圾回收（GC）**的语言中，GC 会跟踪清理那些不再被占用的内存，我们无需为此操心。
如果没有 GC，归还内存就是我们程序员的责任了，就像声明时获取内存一样，释放内存时也要通过代码操作。
做好这一步是编程的历史性难题。如果我们忘记了，就会浪费内存，如果过早释放内存，我们的变量就无效了，
要是我们释放了两次，同样是个 bug。我们需要 `allocate` 和 `free` 的一对一配对。

Rust takes a different path: the memory is automatically returned once the
variable that owns it goes out of scope. Here’s a version of our scope example
from Listing 4-1 using a `String` instead of a string literal:

Rust 另辟蹊径：变量离开它的作用域时，内存会被自动释放。
这里有一个例 4-1 的 `String` 版本：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

There is a natural point at which we can return the memory our `String` needs
to the allocator: when `s` goes out of scope. When a variable goes out of
scope, Rust calls a special function for us. This function is called [`drop`],
and it’s where the author of `String` can put the code to return the memory.
Rust calls `drop` automatically at the closing curly bracket.

很自然地，在 `s` 离开作用域时，`String` 的内存会被归还到分配器。
在变量离开作用域时，Rust 自动为我们调用了一个特殊函数——[`drop`]，
`String` 的作者已经将回收内存的代码写到里面了。
Rust 会在遇到右花括号时自动调用 `drop`。

> Note: In C++, this pattern of deallocating resources at the end of an item’s
> lifetime is sometimes called _Resource Acquisition Is Initialization (RAII)_.
> The `drop` function in Rust will be familiar to you if you’ve used RAII
> patterns.
> 
> 注意：在 C++ 中，这种在变量生命周期结束时释放资源的方式称为 **Resource Acquisition Is Initialization (RAII)**。
如果你曾经接触过 RAII 模式，那么 Rust `drop` 函数会让你觉得很类似。

This pattern has a profound impact on the way Rust code is written. It may seem
simple right now, but the behavior of code can be unexpected in more
complicated situations when we want to have multiple variables use the data
we’ve allocated on the heap. Let’s explore some of those situations now.

这个模式对 Rust 代码的编写方式有深远的影响。虽然到现在为止看起来还很简单，但是随着越来越多的变量
储存到堆中，你的代码可能会出现一些意想不到的问题。接下来我们看看这些情况吧。

#### 变量和数据的交互方式：Move

Multiple variables can interact with the same data in different ways in Rust.
Let’s look at an example using an integer in Listing 4-2.

在 Rust 中，不同的变量可能以不同方式交互相同的数据，
我们来看看例 4-2，使用整形。

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

<span class="caption">例 4-2：把整形的 `x` 赋值到 `y`</span>

We can probably guess what this is doing: “bind the value `5` to `x`; then make
a copy of the value in `x` and bind it to `y`.” We now have two variables, `x`
and `y`, and both equal `5`. This is indeed what is happening, because integers
are simple values with a known, fixed size, and these two `5` values are pushed
onto the stack.

我们可以猜测其中发生的事情：`5` 绑定到 `x`，然后复制一份 `x` 的值，然后绑定到 `y`。
现在就有了 `x` 和 `y` 两个变量，都等于 `5`。事实上，确实是这样，因为整形是确定的简单值，而且确定大小。
这两个 `5` 都被推入栈中。

Now let’s look at the `String` version:

现在看看 `String` 的情况：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

This looks very similar to the previous code, so we might assume that the way
it works would be the same: that is, the second line would make a copy of the
value in `s1` and bind it to `s2`. But this isn’t quite what happens.

这与前面一段代码很像，所以我们推测它们的工作方式也类似：第二行获取 `s1` 的值的拷贝，然后绑定到 `s2`。但事实上不是这样的。

Take a look at Figure 4-1 to see what is happening to `String` under the
covers. A `String` is made up of three parts, shown on the left: a pointer to
the memory that holds the contents of the string, a length, and a capacity.
This group of data is stored on the stack. On the right is the memory on the
heap that holds the contents.

图 4-1 展示了 `String` 在底层的储存方式，它由三部分组成（图的左边）：一个指向储存着字符串内容的内存的指针、
长度还有容量。这组数据储存在栈中，右边则是堆内存，里面储存着字符串的内容。

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-1：在内存中，`String` 储存 `"hello"` 的值并绑定到 `s1`</span>

The length is how much memory, in bytes, the contents of the `String` is
currently using. The capacity is the total amount of memory, in bytes, that the
`String` has received from the allocator. The difference between length
and capacity matters, but not in this context, so for now, it’s fine to ignore
the capacity.

长度是 `String` 当前使用的内存，单位为 byte。容量是分配器给`String`分配的总内存，单位为 byte。
长度和容量是有区别的，但是我们暂时不用纠结这个问题，总之现在先忽略容量吧。

When we assign `s1` to `s2`, the `String` data is copied, meaning we copy the
pointer, the length, and the capacity that are on the stack. We do not copy the
data on the heap that the pointer refers to. In other words, the data
representation in memory looks like Figure 4-2.

当将 `s1` 赋值给 `s2` 时，`String` 数据被复制了一份，也就是说，复制了指针、长度和容量这三个储存在栈中的值。
而指针指向的堆中的数据则不会复制。换言之，数据在内存中如图 4-2 所示。

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-2: 内存中，`s2` 复制了 `s1` 的指针、长度和容量</span>

The representation does _not_ look like Figure 4-3, which is what memory would
look like if Rust instead copied the heap data as well. If Rust did this, the
operation `s2 = s1` could be very expensive in terms of runtime performance if
the data on the heap were large.

而**不会**是图 4-3 这样，把堆的数据也复制一遍。如果 Rust 这么做，在堆中数据十分庞大的情况下，
`s2 = s1` 这个操作就会花费很多时间，影响运行时表现。

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Another possibility for what `s2 = s1` might
do if Rust copied the heap data as well</span>

Earlier, we said that when a variable goes out of scope, Rust automatically
calls the `drop` function and cleans up the heap memory for that variable. But
Figure 4-2 shows both data pointers pointing to the same location. This is a
problem: when `s2` and `s1` go out of scope, they will both try to free the
same memory. This is known as a _double free_ error and is one of the memory
safety bugs we mentioned previously. Freeing memory twice can lead to memory
corruption, which can potentially lead to security vulnerabilities.

之前，我们提到当变量离开作用域，Rust 会自动调用 `drop` 函数清理堆中内存。
但图 4-2 所示，有两个数据的指针都指向同一个位置，这就成问题了：`s2` 和 `s1` 离开作用域，
它们会同时释放同一块内存，这被称为**double free**错误，是一个我们之前提到的内存安全问题。
释放内存两次会导致 memory corruption，造成潜在的安全漏洞。

To ensure memory safety, there’s one more detail to what happens in this
situation in Rust. Instead of trying to copy the allocated memory, Rust
considers `s1` to no longer be valid and, therefore, Rust doesn’t need to free
anything when `s1` goes out of scope. Check out what happens when you try to
use `s1` after `s2` is created; it won’t work:

为了保证内存安全，在这种情况下 Rust 有一个细节操作。与其复制一份已经分配的内存，
Rust 直接将 `s1` 视为不可用，因此Rust 在`s1`离开作用域时就不需要释放内存了。
看看再`s2`创建后再使用`s1` 会怎么样？结果是它不能用了：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

You’ll get an error like this because Rust prevents you from using the
invalidated reference:

Rust 会报出一个这样的错误，防止你使用已经无效的引用：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

If you’ve heard the terms _shallow copy_ and _deep copy_ while working with
other languages, the concept of copying the pointer, length, and capacity
without copying the data probably sounds like making a shallow copy. But
because Rust also invalidates the first variable, instead of being called a
shallow copy, it’s known as a _move_. In this example, we would say that
`s1` was _moved_ into `s2`. So what actually happens is shown in Figure 4-4.

如果你之前听说过**浅复制**和**深复制**，只复制指针、长度和容量就像是浅复制。
但因为 Rust 会同时让之前的变量无效，所以就不称其为浅复制，而是**move**。
在这个例子里，我们可以说 `s1` **移动到了** `s2`，所以实际情况如图 4-4 所示。

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: Representation in memory after `s1` has been
invalidated</span>

That solves our problem! With only `s2` valid, when it goes out of scope, it
alone will free the memory, and we’re done.

这就解决了问题，只有 `s2` 有效的情况下，离开作用域时只有它被释放，搞定。

In addition, there’s a design choice that’s implied by this: Rust will never
automatically create “deep” copies of your data. Therefore, any _automatic_
copying can be assumed to be inexpensive in terms of runtime performance.

这是一个设计上的选择，Rust 永远不会自动深复制你的数据，因此任何
**默认的**复制为了保证运行时性能都会选择最快捷的方式。

#### 变量和数据的交互方式：Clone

If we _do_ want to deeply copy the heap data of the `String`, not just the
stack data, we can use a common method called `clone`. We’ll discuss method
syntax in Chapter 5, but because methods are a common feature in many
programming languages, you’ve probably seen them before.

如果我们**就是**需要深复制堆里的 `String` 数据，我们可以使用通用方法 `clone`。
我们将会在第五章讨论方法语法，但毕竟方法是编程语言的常见特性，大家应该在之前就接触过了。

Here’s an example of the `clone` method in action:

这是一个 `clone` 方法的例子：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

This works just fine and explicitly produces the behavior shown in Figure 4-3,
where the heap data _does_ get copied.

这样就产生了图 4-3 的效果，堆中的数据**也成功被复制**。

When you see a call to `clone`, you know that some arbitrary code is being
executed and that code may be expensive. It’s a visual indicator that something
different is going on.



#### 栈数据专用的：Copy

There’s another wrinkle we haven’t talked about yet. This code using integers –
part of which was shown in Listing 4-2 – works and is valid:

还有一个小问题我们还未谈到。例 4-2 中代码使用了整形数，它正常工作，完全合理。

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

But this code seems to contradict what we just learned: we don’t have a call to
`clone`, but `x` is still valid and wasn’t moved into `y`.

但这代码似乎和我们刚学到的有冲突：我们没有用 `clone`，但是 `x` 依然可用，没有被 move 到 `y`。

The reason is that types such as integers that have a known size at compile
time are stored entirely on the stack, so copies of the actual values are quick
to make. That means there’s no reason we would want to prevent `x` from being
valid after we create the variable `y`. In other words, there’s no difference
between deep and shallow copying here, so calling `clone` wouldn’t do anything
different from the usual shallow copying and we can leave it out.

原因是整形等类型在编译时拥有确定的大小，并可以完全储存在栈中，所以复制实际的值也十分快捷。
所以也就没有必要在创建 `y` 后让 `x` 无效了。换言之，深浅复制在这里毫无区别，
用不用 `clone` 也不会有什么不同，这种情况下可以忽略它的存在。

Rust has a special annotation called the `Copy` trait that we can place on
types like integers that are stored on the stack (we’ll talk more about traits
in Chapter 10). If a type implements the `Copy` trait, an older variable is
still usable after assignment. Rust won’t let us annotate a type with the
`Copy` trait if the type, or any of its parts, has implemented the `Drop`
trait. If the type needs something special to happen when the value goes out of
scope and we add the `Copy` annotation to that type, we’ll get a compile-time
error. To learn about how to add the `Copy` annotation to your type to
implement the trait, see [“Derivable Traits”][derivable-traits]<!-- ignore -->
in Appendix C.



So what types implement the `Copy` trait? You can check the documentation for
the given type to be sure, but as a general rule, any group of simple scalar
values can implement `Copy`, and nothing that requires allocation or is some
form of resource can implement `Copy`. Here are some of the types that
implement `Copy`:



- All the integer types, such as `u32`.
- The Boolean type, `bool`, with values `true` and `false`.
- All the floating point types, such as `f64`.
- The character type, `char`.
- Tuples, if they only contain types that also implement `Copy`. For example,
  `(i32, i32)` implements `Copy`, but `(i32, String)` does not.

### Ownership 和函数

The semantics for passing a value to a function are similar to those for
assigning a value to a variable. Passing a variable to a function will move or
copy, just as assignment does. Listing 4-3 has an example with some annotations
showing where variables go into and out of scope.

向函数传入一个值与赋值一个变量是一样的，所以向函数传入变量就像赋值一样产生 move 或者 copy。
例 4-3 展示了变量进入与离开作用域。

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

<span class="caption">例 4-3：函数 ownership 和作用域</span>

If we tried to use `s` after the call to `takes_ownership`, Rust would throw a
compile-time error. These static checks protect us from mistakes. Try adding
code to `main` that uses `s` and `x` to see where you can use them and where
the ownership rules prevent you from doing so.

如果在 `takes_ownership` 后访问 `s`，Rust 会抛出编译错误。这种静态检测提早提醒我们修复错误。
你可以尝试在 `main` 添加一些代码，看看 `s` 和 `x` 的使用范围。

### Return Values and Scope 返回值与作用域

Returning values can also transfer ownership. Listing 4-4 is an example with
similar annotations to those in Listing 4-3.

返回值也能转移 ownership，请看例 4-4。

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

<span class="caption">例 4-4：转移返回值的 ownership</span>

The ownership of a variable follows the same pattern every time: assigning a
value to another variable moves it. When a variable that includes data on the
heap goes out of scope, the value will be cleaned up by `drop` unless the data
has been moved to be owned by another variable.

变量的 ownership 遵循相同的模式：赋值到另一个变量 move 它。
当一个包含堆数据的变量离开作用域，它的值会被 `drop` 清理——
除非这些数据已经被 move 到了其他变量。

Taking ownership and then returning ownership with every function is a bit
tedious. What if we want to let a function use a value but not take ownership?
It’s quite annoying that anything we pass in also needs to be passed back if we
want to use it again, in addition to any data resulting from the body of the
function that we might want to return as well.



It’s possible to return multiple values using a tuple, as shown in Listing 4-5.

可以像例 4-5 使用元组返回多个值。

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

<span class="caption">例 4-5：归还参数的 ownership</span>

But this is too much ceremony and a lot of work for a concept that should be
common. Luckily for us, Rust has a feature for this concept, called
_references_.

但每次都这么做就有点麻烦了，这本应是很普通的需求，幸运地，Rust 有一个解决这个问题的特性，
叫做 **引用（reference）**。

[data-types]: ch03-02-data-types.html#data-types
[derivable-traits]: appendix-03-derivable-traits.html
[method-syntax]: ch05-03-method-syntax.html#method-syntax
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[`drop`]: ../std/ops/trait.Drop.html#tymethod.drop
