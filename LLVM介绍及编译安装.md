## LLVM介绍

LLVM的命名最早起源于底层语言虚拟机（Low Level Virtual Machine）的缩写 。它是一个用于建立编译器的基础框架，以C++编写。创建此工程的目的是对于任意汇编语言，利用该基础框架，构建一个包括编译时、链接时、执行时等的语言执行器。目前官方的LLVM只支持处理C/C++，Objective-C三种语言，当然的也有的一些非官方的扩展，使其支持ActionScript、Ada、D语言、Fortran、GLSL、HaskKell、Java bytecode、Objective-C、Python、Ruby、Rust、Scala以及C#。

传统的静态编译器分为三个阶段：前端、中端（优化）、后端，LLVM 也不例外，只是 LLVM 实现了一种**与源编程语言和目标机器架构无关的通用中间表示——LLVM IR**， 这样如果支持一种新的编程语言只需重新实现一个前端，支持一种新的目标架构只需重新实现一个后端，前端和后端的链接枢纽就是**LLVM IR**

