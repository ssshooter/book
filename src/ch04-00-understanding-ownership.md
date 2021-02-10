# 理解 ownership

所有权（ownership）是 Rust 最独特的特性，有了它 Rust 可以在不需要 GC 的情况下保证内存安全。
因此理解 ownership 如何工作是学习 Rust 的重要一环。
在这章中，我们会提及 ownership 和一些相关特性：borrowing、slice，并理解 Rust 如何在内存中储存数据。
