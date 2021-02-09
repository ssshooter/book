# 理解所有权

Ownership is Rust’s most unique feature, and it enables Rust to make memory
safety guarantees without needing a garbage collector. Therefore, it’s
important to understand how ownership works in Rust. In this chapter, we’ll
talk about ownership as well as several related features: borrowing, slices,
and how Rust lays data out in memory.

所有权是 Rust 最独特的特性，他让 Rust 再不需要 GC 的情况下保证内存安全。
因此理解所有权如何运行是学习 Rust 的重要一环。
在这章中，我们会提及所有权和一些相关特性：borrowing、slices 以及 Rust 如何在内存中储存数据。
