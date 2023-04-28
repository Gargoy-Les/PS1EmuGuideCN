# 2.3 程序计数器

[寄存器](https://zh.wikipedia.org/wiki/%E5%AF%84%E5%AD%98%E5%99%A8)是内建于CPU的极小而极快的特定用途的存储器。大多数CPU指令通过加法、乘法、**屏蔽\[1]** 将其内容存储至内存或取回以操作这些寄存器。

[程序计数器](https://zh.wikipedia.org/wiki/%E7%A8%8B%E5%BC%8F%E8%A8%88%E6%95%B8%E5%99%A8)(PC)是最基本的寄存器之一，它存在于几乎所有的计算机体系结构中，保存下一条指令的地址。

PS1使用32位地址，所以程序计数器寄存器是32位宽的(就像所有其他CPU寄存器一样)。

一个典型的CPU执行周期大致如此:

1. 取得程序计数器所指向的指令
2. 增加程序计数器的值
3. 执行指令
4. 重复

得知一条指令的大小，才能得知要取多少字节，和PC增加的量(以指向下一条指令)。一些架构有可变长度的指令(例如：x86及其派生)，这意味着我们必须解码指令才能知道它需要多少个字节。幸运的是，PlayStation使用固定长度的指令集([MIPS指令集](https://zh.wikipedia.org/wiki/MIPS%E6%9E%B6%E6%A7%8B))，所有的指令都是32位长。

现在开始写代码!

此时CPU的状态：

```rust
/// CPU的状态
pub struct Cpu {
    /// 程序计数器
    pc : u32,
     }
```

这是上述的 CPU 周期的实现：\\

```rust
impl Cpu {
    pub fn run_next_instruction(&mut self) {
        let pc = self.pc;
        // 取得程序计数器所指向的指令
        let instruction = self.load32(pc);
        // 增加程序计数器的值以指向下一条指令
        self.pc = pc.wrapping_add(4);
        self.decode_and_execute(instruction);
    }
}
```

在 Rust 中 wrapping\_add的作用是程序计数器在溢出的情况下返回 0（即 0xfffffffc + 4 => 0x00000000）。 我们将看到大多数 CPU 操作都在溢出时结束（尽管有些指令会捕获这些溢出并生成异常，我们稍后会看到）。

C语言在这种情况下无需担心是否使用 uint32\_t，因为 C 标准要求无符号溢出以wrapping\_add的方式返回。 然而，Rust 规定溢出是未定义的，如果检测到未经检查的溢出，将在debug builds中产生错误，这就是为什么我需要编写 pc.wrapping\_add(4) 而不是 pc + 4。

代码太少还不能Build，运行代码前还有三个难题：

* 启动时程序计数器的初始值是多少？
* 如何实现 fetch32 函数？
* 如何实现 decode\_and\_execute 函数？

注解：

**屏蔽\[1]** ：一种运算操作，用于将寄存器中的数据或位进行掩码处理，只保留需要的位或数据，屏蔽掉不需要的部分。
