## 什么是 ownership？

**Ownership** 是 Rust 的重要特性。尽管这个特性直接解释起来不难，但是它深远地影响着这个语言的每一部分。

所有程序都需要解决他们在运行时如何使用计算机内存的问题。
一些语言有垃圾收集系统（GC），GC 会在程序运行时自动寻找可以释放的内存，
在另一些语言，程序员需要显式分配和释放内存。
而 Rust 使用第三种方式：在编译时，ownership 系统借助**编译器**的一系列检查规则管理内存。
任何 ownership 特性都不会让拖慢程序的运行速度。

因为 ownership 对很多程序员来说都是全新的概念，需要花点时间习惯它。
好消息是，你对 Rust 和 ownership 系统的规则越熟悉，
越能自然地写出安全高效的代码。加油！

当你理解 ownership，你就有了理解其他 Rust 特性的巩固基础。
在这一章，你将通过十分常见的数据结构——字符串，来学习 ownership。

> ### 栈（stack）和堆（heap）
>
> 在很多语言中，你基本不需要钻研栈和堆。但在 Rust 这样的语言，
> 一个值存在栈或堆会对语言的行为产生影响，所以你需要在此作出正确的选择。
> 这章后面的 ownership 部分会描述它与堆和栈的关系，这里仅是提前做些简单的解释。
>
> 堆和栈都是代码在运行时可以调用的一部分内存，但是它们的结构有所不同。
> 栈的储存顺序与获取顺序相同，与删除顺序相反，这就是所谓**后进先出**。
> 就像一叠盘子，放盘子的时候你会放在一叠盘子的最上方，而取盘子的时候你会拿走最上面的一个。
> 你想在中间和底部加盘子都很不方便！添加数据我们称为**推入栈中**，取出数据称为**弹出栈**。
>
> 储存在栈的数据必须有一个已知且固定的长度。而在编译时未知长度，或长度会改变的数据，则储存在堆中。
> 堆相对栈没有那么强的组织性：数据放入堆时，你申请了一部分空间。
内存分配器找到一块足够你使用的空位，将其标记为已使用，
> 然后返回一个指向该地址的**指针**。这个过程称为**在堆中分配内存（allocating on the heap）**，有时缩写为**分配（allocating）**。
> 推入栈的操作不属于分配，因为那个指针是已知的，固定长度的，你可以在栈中储存指针，但你想获取实际数据，就必须跟随指针寻找。
>
> 举一个你去餐厅的例子。进餐厅时，你告诉服务员有多少人用餐，然后服务员找到合适的桌子并带你到那里。
> 如果你的伙伴们晚了来，他们就可以问出你在哪并找到你。
>
> 推入栈比堆分配更快，因为数据总会存在栈顶，分配器不需要寻找存放新数据的位置，而堆在分配空间时需要做更多的工作，
> 因为分配器必须找到一块足够大的地方存放数据，并进行记录以备下次再使用。
>
> 访问堆里的数据比访问栈数据要慢，因为你要跟随指针获取数据。如果不在内存中反复横跳，近代处理器会更快一些。
继续刚才的比喻，餐厅服务员给很多桌客人下单，最快的方法就是一张桌子全都下完单，再到下一张桌子，
如果先给 A 下单一个菜，然后给 B 下单，又再回到 A，就会非常缓慢。同样的，处理器可以更快地当前数据临近的数据
（就像栈这样的结构），而远的就会变慢（堆）。同时，在堆中分配空间也消耗时间。
>
> 当你的代码调用函数，值（也有可能是指向堆数据的指针）会被传入函数，
然后函数的本地变量会被推入栈中，函数运行结束，这些变量会被弹出。
>
> Ownership 要做的就是保持跟踪不同的代码使用堆中的不同数据，
最小化堆中的重复数据、清除无用数据，保证你的内存不会爆炸。
> 一旦你理解 ownership，你就不用整天关注堆和栈，不过知道管理堆数据是 ownership 的存在意义，
可以帮助你理解它为什么要如此设计。

### Ownership 规则

首先，我们看看 ownership 的几条规则，记住这些规则，再看看下面详细介绍的例子：

- Rust 里的每个值都有一个叫 **owner** 的变量。
- 一个值同时只能有一个 owner
- owner 离开作用域时，值将被抛弃（drop）

### 变量作用域

第二章中已经学习过几个 Rust 程序的实例，现在我们不关心基础的语法问题，
暂时去掉 `fn main() {`，如果你想跟着做，需要手动加上 `main` 函数。
这样，我们的例子会更简洁，让我们更关注我们需要关注的细节。

来看看 ownership 的第一个例子，变量的**作用域**。
作用域是在程序里的一个范围，在这个范围内，变量是有效的。
举个例子，我们有一个变量 `s`：

```rust
let s = "hello";
```

变量 `s` 指向一个字符串字面量（string literal），这个值是写死在程序中的。
这个变量从声明到当前**作用域**结束，是可用的。例 4-1 的注释说明了 `s` 的使用范围。

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

<span class="caption">例 4-1：变量与它的有效作用域</span>

换言之，有两个重点：

- `s` **进入作用域**，那么它是有效的
- 直到**脱离当前作用域**，一直有效

暂时为止，作用域和变量有效性的关系与其他语言基本一样。现在我们在此基础上，介绍 `String` 类型。

### `String` 类型

为了说明 ownership 的规则，我们需要一种比第三章提到的更复杂的[数据格式][data-types]<!-- ignore -->。
之前提到的类型都储存于栈中，在离开作用域时，弹出栈，但我们现在要研究储存于堆中的数据，
并搞清楚 Rust 如何清理这些数据。

例子中我们使用 `String`，并关注它与 ownership 的关系。
它的表现与其他复杂数据类型（不管是标准库提供还是你自己创建的）类似。我们将会在第八章深入讨论 `String`。

上面已经使用过字符串字面量，也就是字符串的值写死到程序中。字符串字面量很方便，但不能适应各种情况，
原因就是他们是不可变的（immutable）。不是所有字符串的值都是确定的，例如我们想获取用户输入然后储存起来要怎么办？
要应付这种情况，Rust 提供第二种字符串类型，`String`。这个类型分配堆空间，可以存放一些我们在编译时未确定的文本。
你可以通过 `from` 函数由字符串字面量创建 `String`：

```rust
let s = String::from("hello");
```

双冒号（`::`）操作符让你在 `String` 命名空间下使用 `from` 函数，而不是通过 `string_from` 之类的方法。
我们会在第五章 [“Method Syntax”][method-syntax]<!-- ignore --> 部分讨论更多语法，
在第七章 [“Paths for Referring to an Item in the Module Tree”][paths-module-tree]<!-- ignore -->
讨论命名空间和模块化。

这种字符串是**可以**被修改的：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

那么这里面有什么不同？为什么 `String` 可以被修改但是字面量不可以呢？
其中缘由就是他们在内存中的两种不同处理方式。

### 内存与分配（Allocation）

在字符串字面量的例子中，我们在编译时就确定了内容，所以文本直接写死到最后的可执行文件。
这就是文本字面量快速高效的原因。但这些好处来源于字符串字面量的不变性。
我们不能把大小未知或在运行时会遭到修改的文本编译为二进制文件。

使用 `String` 支持可变的文本，需要在堆中分配内存来放置这些内容，其分配大小在编译时是未知的，意味着：

- 分配器需要在运行时请求内存
- 我们需要在使用完 `String` 之后归还那部分内存

That first part is done by us: when we call `String::from`, its implementation
requests the memory it needs. This is pretty much universal in programming
languages.

第一部分已经在调用 `String::from` 的时候完成了，执行后它获得了所需的内存。
这在编程语言中很常见。

然而第二部分就有点不一样了。在有**垃圾回收（GC）**的语言中，GC 会跟踪清理那些不再被占用的内存，我们无需为此操心。
如果没有 GC，归还内存就是我们程序员的责任了，就像声明时获取内存一样，释放内存时也要通过代码操作。
做好这一步是编程的历史性难题。如果忘记释放，就会浪费内存，如果过早释放，变量就无效了，
要是我们释放了两次，同样是个 bug。`allocate` 和 `free` 必须一对一配对。

Rust 另辟蹊径：变量离开作用域时，内存会被自动释放。请看例 4-1 的 `String` 版本：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

很自然地，在 `s` 离开作用域时，`String` 的内存会被归还到分配器。
在变量离开作用域时，Rust 自动为我们调用了一个特殊函数——[`drop`]，
`String` 的作者已经将回收内存的代码写到里面了。
Rust 会在遇到右花括号时自动调用 `drop`。

> 注意：在 C++ 中，这种在变量生命周期结束时释放资源的方式称为 **Resource Acquisition Is Initialization (RAII)**。
如果你曾经接触过 RAII 模式，那么 Rust `drop` 函数会让你觉得很类似。

这个模式对 Rust 代码的编写方式有深远的影响。虽然到现在为止看起来还很简单，
但是多个变量使用堆中的同一数据时，代码可能会出现一些意想不到的问题。接下来我们看看这些情况吧。

#### 变量和数据的交互方式：Move

Multiple variables can interact with the same data in different ways in Rust.
Let’s look at an example using an integer in Listing 4-2.

在 Rust 中，多个的变量可能以不同方式处理同一个数据，
我们来看看例 4-2，使用整形。

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

<span class="caption">例 4-2：把整形的 `x` 赋值到 `y`</span>

我们可以猜测其中发生的事情：`5` 绑定到 `x`，然后复制一份 `x` 的值，然后绑定到 `y`。
现在就有了 `x` 和 `y` 两个变量，都等于 `5`。事实上，确实是这样，因为整形是确定的简单值，而且确定大小。
这两个 `5` 都被推入栈中。

现在看看 `String` 的情况：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

这与前面一段代码很像，我们推测两者的工作方式也类似：
第二行获取 `s1` 的值的拷贝，然后绑定到 `s2`。但事实上不是这样的。

图 4-1 展示了 `String` 在底层的储存方式，它由三部分组成（图的左边）：一个指向储存着字符串内容的内存的指针、
长度还有容量。左边的数据储存在栈中，右边则是在堆中，里面储存着字符串的内容。

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-1：在内存中，`String` 储存 `"hello"` 的值并绑定到 `s1`</span>

长度是 `String` 当前使用的内存，单位为 byte。容量是分配器给 `String` 分配的总内存，单位为 byte。
长度和容量是有区别的，但是我们暂时不用纠结这个问题，总之现在先忽略容量吧。

当将 `s1` 赋值给 `s2` 时，`String` 数据被复制了一份，也就是说，复制了指针、长度和容量这三个储存在栈中的值。
而指针指向的堆中的数据则不会复制。换言之，数据在内存中如图 4-2 所示。

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-2: 内存中，`s2` 复制了 `s1` 的指针、长度和容量</span>

而**不会**是图 4-3 这样，把堆的数据也复制一遍。如果 Rust 这么做，在堆中数据十分庞大的情况下，
`s2 = s1` 这个操作就会花费很多时间，影响运行时表现。

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-3：如果 `s2 = s1` Rust 复制了堆里的数据</span>

之前，我们提到当变量离开作用域，Rust 会自动调用 `drop` 函数清理堆中内存。
但图 4-2 所示，有两个数据的指针都指向同一个位置，这就成问题了：`s2` 和 `s1` 同时离开作用域，
它们会同时释放同一块内存，这被称为**double free**错误，是一个我们之前提到的内存安全问题。
释放内存两次会导致 memory corruption，造成潜在的安全漏洞。

为了保证内存安全，在这种情况下 Rust 有一个细节操作。与其复制一份已经分配的内存，
Rust 直接将 `s1` 视为不可用，因此 Rust 在`s1`离开作用域时就不需要释放内存了。
试试在 `s2` 创建后再使用 `s1` 会怎么样？结果是它不能用了：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

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

如果你之前听说过**浅复制（shallow copy）**和**深复制（deep copy）**，
只复制指针、长度和容量就类似浅复制。
但因为 Rust 会同时让之前的变量无效，所以就不称其为浅复制，而称为 **move**。
在这个例子里，我们可以说 `s1` **移动到了** `s2`，所以实际情况如图 4-4 所示。

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-4：内存中 `s1` 已失效</span>

这就解决了问题，只有 `s2` 有效的情况下，离开作用域时只有它被释放，搞定。

这是一个设计上的选择，Rust 永远不会自动深复制你的数据，任何
**默认的**复制为了保证运行时性能都会选择最快捷的方式。

#### 变量和数据的交互方式：Clone

如果我们**就是**需要深复制堆里的 `String` 数据，我们可以使用通用方法 `clone`。
我们将会在第五章讨论方法语法，但毕竟方法是编程语言的常见特性，大家应该都已经接触过了。

使用 `clone` 方法的例子：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

这样就产生了图 4-3 的效果，堆中的数据**也成功被复制**。

看到 `clone` 被调用，你要立刻反应到代码的性能可能会可能不高，明白地告诉你这些代码会有点与众不同。

#### 栈数据专用的：Copy

还有一个小问题我们还未谈到。例 4-2 中代码使用了整形数，它正常工作，完全合理。

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

但这代码似乎和我们刚学到的有冲突：我们没有用 `clone`，但是 `x` 依然可用，没有被 move 到 `y`。

原因是整形等类型在编译时拥有确定的大小，并**可以完全储存在栈中**，所以复制实际的值也十分快捷，
也就没有必要在创建 `y` 后让 `x` 无效了。换言之，深浅复制在这里毫无区别，
用不用 `clone` 不会有什么不同。

Rust has a special annotation called the `Copy` trait that we can place on
types like integers that are stored on the stack (we’ll talk more about traits
in Chapter 10). If a type implements the `Copy` trait, an older variable is
still usable after assignment. 

Rust won’t let us annotate a type with the
`Copy` trait if the type, or any of its parts, has implemented the `Drop`
trait. If the type needs something special to happen when the value goes out of
scope and we add the `Copy` annotation to that type, we’ll get a compile-time
error. To learn about how to add the `Copy` annotation to your type to
implement the trait, see [“Derivable Traits”][derivable-traits]<!-- ignore -->
in Appendix C.

Rust 有一种特别的写法，称为 `Copy` 特性（trait）。
我们将在第十章了解更多关于 trait 的东西
如果一个类型实现了 `Copy` 特性，旧变量在赋值后依然可用。
Rust 

So what types implement the `Copy` trait? You can check the documentation for
the given type to be sure, but as a general rule, any group of simple scalar
values can implement `Copy`, and nothing that requires allocation or is some
form of resource can implement `Copy`. Here are some of the types that
implement `Copy`:



- 所有整形，如 `u32`
- 布尔值 `bool`，值为 `true` 和 `false`
- 所有浮点型，如 `f64`
- 字符型 `char`
- 元组，如果只包含指定类型，也可以实现 `Copy`，例如 `(i32, i32)` 可以 `Copy`，`(i32, String)` 不行

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

### 返回值与作用域

返回值也能转移 ownership，请看例 4-4。

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

<span class="caption">例 4-4：转移返回值的 ownership</span>

变量的 ownership 遵循相同的模式：一个变量赋值到另一个变量会 move 它。
当一个包含堆数据的变量离开作用域，它的值会被 `drop` 清理——
除非这些数据已经被 move 到了其他变量。

Taking ownership and then returning ownership with every function is a bit
tedious. What if we want to let a function use a value but not take ownership?
It’s quite annoying that anything we pass in also needs to be passed back if we
want to use it again, in addition to any data resulting from the body of the
function that we might want to return as well.

每个函数都获取 ownership 之后再返还 ownership 会十分麻烦，我们可不可以在不获取 ownership 的情况下使用这个值呢？


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
