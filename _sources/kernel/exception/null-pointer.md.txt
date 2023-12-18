
exception
===========


## 内核空指针访问一定会panic 吗


不一定会 ， 在一般上下文中， 直接访问空指针，执行完oops_end 就会直接退出(虚拟机环境)：

async_page_fault —> __do_page_fault —> no_context  —> oops_end —> do_exit 让当前进程退出


如果在中断上下文中访问空指针，则会进入panic， 区别就在oops_end 代码中


```C
void oops_end(unsigned long flags, struct pt_regs *regs, int signr)
{
    if (regs && kexec_should_crash(current))
        crash_kexec(regs);  //kdump xxx

    if (!signr)
        return;
    if (in_interrupt())   // 如果是中断上下文，则触发panic
        panic("Fatal exception in interrupt");
    if (panic_on_oops)  // 如果在异常上下文也会
        panic("Fatal exception");

    kasan_unpoison_task_stack(current);
    rewind_stack_do_exit(signr);   // 准备调用do_exit
}
NOKPROBE_SYMBOL(oops_end);
```



