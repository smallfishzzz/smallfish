# stack smashing detected 

## 这个代码会造成栈溢出
```c
#include <stdio.h>
#include <stdbool.h>

void set_val(int *val)
{
        *val = 1;
}

int main()
{
    bool i = 0;
    
    set_val(&i);
    return 0;
}
```

当编译代码的时候打开了栈保护的时候，-fstack-protector-strong 或者 -fstack-protector-all，编译器会在栈上放置一个随机数，当函数返回的时候会检查这个随机数是否被修改，如果被修改了，就会报错。

```asm
Dump of assembler code for function main:
   0x0000555555555162 <+0>:     endbr64
   0x0000555555555166 <+4>:     push   %rbp
   0x0000555555555167 <+5>:     mov    %rsp,%rbp
   0x000055555555516a <+8>:     sub    $0x10,%rsp
   0x000055555555516e <+12>:    mov    %fs:0x28,%rax
   0x0000555555555177 <+21>:    mov    %rax,-0x8(%rbp)
   0x000055555555517b <+25>:    xor    %eax,%eax
   0x000055555555517d <+27>:    movb   $0x0,-0x9(%rbp)
   0x0000555555555181 <+31>:    lea    -0x9(%rbp),%rax
   0x0000555555555185 <+35>:    mov    %rax,%rdi
   0x0000555555555188 <+38>:    call   0x555555555149 <set_val>
   0x000055555555518d <+43>:    mov    $0x0,%eax
   0x0000555555555192 <+48>:    mov    -0x8(%rbp),%rdx
   0x0000555555555196 <+52>:    sub    %fs:0x28,%rdx
   0x000055555555519f <+61>:    je     0x5555555551a6 <main+68>
   0x00005555555551a1 <+63>:    call   0x555555555050 <__stack_chk_fail@plt>
=> 0x00005555555551a6 <+68>:    leave
   0x00005555555551a7 <+69>:    ret
```