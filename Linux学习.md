内核空间于用户空间的数据交互：copy_from_user(void *to, const void __user *from, unsigned long n)

相对路径为**linux/include/linux/uaccess.h**,源码中对于这个操作进行了两次封装

```c
static __always_inline unsigned long
__copy_from_user_inatomic(void *to, const void __user *from, unsigned long n)
{
	kasan_check_write(to, n);//检查目标内存区域是否已经被映射到了用户空间
	check_object_size(to, n, false);//对拷贝长度进行安全性检查，避免了潜在的内存访问错误
	return raw_copy_from_user(to, from, n);
}

static __always_inline unsigned long
__copy_from_user(void *to, const void __user *from, unsigned long n)
{
	might_fault();
	kasan_check_write(to, n);//检查目标内存区域是否已经被映射到了用户空间
	check_object_size(to, n, false);//对拷贝长度进行安全性检查，避免了潜在的内存访问错误
	return raw_copy_from_user(to, from, n);
}
```



- 真正执行操作的**`raw_copy_from_user`**
- 一层壳**`__copy_from_user`** / **`__copy_from_user_inatomic`**

这两个函数都可以实现用户空间和内核空间的数据交互。区别在于：

- `__copy_from_user_inatomic`是不可重入版本，可以在原子上下文中使用，但是没有睡眠等待能力
- `__copy_from_user()`函数则不能在原子上下文中使用，但可以使用更多的内存交换功能来提高效率