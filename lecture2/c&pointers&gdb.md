#### Memory in C

Static Memory

● Global variables, accessible throughout the whole program

● Defined with static keyword, as well as variables defined in global scope.

Stack Memory

● Local variables within functions. Destroyed after function exits.

Heap Memory

● You control creation and destruction of these variables: malloc(), free()

● Can lead to memory leaks, use-after-free issues.

#### Pointers in C

* 指针是一个 64 位整数，其值是内存中的一个地址。
* 每个变量都有一个地址，因此可以访问每个变量的指针，包括指向指针的指针。 和一个指向一个指针的指针。 依此类推。
* A 指针可以使用运算符++、--、+、- 处理算术运算。

#### Pointers Syntax



