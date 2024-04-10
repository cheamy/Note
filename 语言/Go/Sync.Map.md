## 介绍

Go语言map不支持并发操作, 会出现fatal error, 无法被defer+recover捕获, 所以提供了并发安全的Sync.Map, 本质上是用空间换时间, 来获得并发读写能力, 底层是由readmap 和dirtymap组成, readmap负责过滤掉对map的读、删、改操作，dirtymap负责消化写操作，保持全量数据，对readmap进行兜底

## 操作

``` go
// 增
m.Store("11")

// 读
m.Load("11")

// 删
m.Delete("11")

// 遍历
walk := func(key, value interface{}) bool {
	fmt.Println("Key =", key, "Value =", value)
	return true
}
m.Range(walk)
```

## 写

1. 看一下readmap里是否存在对应key-entry对(只要状态不为expunged), 存在则使用CAS操作更新, return
2. 加锁double check
3. 如果在readmap中存在, 直接更新(对应的是dirtymap将全量数据交给readmap后), 如果是expunged态, 则需要再dirtymap中补全对应的k-v对 返回
4. 如果在dirtymap存在, 直接更新, 返回
5. 直接在dirtymap中写入新的kv对

## 读

1. 先查readmap, 如果有, 返回
2. 加锁, double check
3. 再查一下readmap, 如果没有, 则访问dirtymap
4. 进入missLocked流程
   1. 增加miss次数
   2. 判断是否超过dirtymap长度, 是则将数据交给readmap,dirtymap置空,miss归零;
5. 解锁, 返回结果

## 删

1. 先看readmap, 如果有 将entry变为nil态
2. 如果readmap没有对应keyentry, 且amend为true, 数据有缺失, 加锁doublecheck, 再访问dirtymap,   尝试从dirtymap中删除并进入missLocked流程
3. 释放锁

## entry状态

entry里是一个unsafe.Pointer, 用来指向值. 使用三种状态来标识它

1. 正常状态

2. nil(软删), 此时readmap和dirtymap仍保有物理地址, 只是逻辑上不存在

   当删除操作在readmap中命中, 则将k对应的entry.p置为nil

3. expunged(硬删), 此时dirtymap不保有该entry的物理地址

   当readmap的读穿透(miss次数)达到上限, 则将dirtymap全部交于readmap, 此时当dirtymap有写请求, 则需要将readmap中逻辑存在的key-entry拷贝到dirtymap中, nil状态的则更新为expunged态

