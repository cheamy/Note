# defer

## _defer内存分配

- 内联
- 堆上分配
- 栈上分配

优先使用内联方式, 内联不满足且没有内存逃逸, 使用栈, 都不符合使用堆

## 底层结构

![image-20240127094329590](F:\Note\实习冲刺\语言\Go\defer.assets\image-20240127094329590.png)

每个defer语句对应一个_defer实例, 组成单链表, 保存在goroutine数据结构中, 每次从头部插入 \_defer,实现后进先出