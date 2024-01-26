# map

map是一个指向hmap的指针

hmap结构体:

``` go
type hmap struct{
    count 		int 			// 元素数量
    flags		uint8			// 记录map的状态
    B			uint8			// 桶的数量为2^B
    noverflow	uint16			// 溢出桶数量的近似值
    hash0		uint32			// 哈希种子
    
    buckets		unsafe.Pointer	// 指向buskets数组([]bmap)的指针
    oldbuckets	unsafe.Pointer	// 指向buskets数组的指针, 用来扩容时保存旧桶, 非扩容时为空
    nevacuate	uintptr			// 表示扩容进度的计数器, 用于记录下一个需要迁移的旧桶
    
    extra		*mapextra		// 指向mapextra结构的指针, mapextra存储map中的溢出桶
}
```

``` go
type bmap struct{
    topbits 	[8]unint8		// 存储桶中存入的每个key hash值的高八位, 用于定位在桶中的位置
    // K/V未被使用时, 存的是K/V对应位置的状态(重要的有两个, 当前是否为空以及后面有没有元素)
    keys		[8]keytype		
    values		[8]valuetype
    overflow	uintptr			// 指向溢出桶的指针(拉链法解决hash冲突)
}
```

由于B长度为8, 而定位桶的操作是与运算:

` bucket := hash & bucketMask(h.B)`

所以其实是低八位确定桶的位置, 高八位确定元素在桶内的位置

``` go
type mapextra struct {
	overflow		*[]*bmap	// 溢出桶链表地址
	oldoverflow 	*[]*bmap	// 指向旧哈希表中溢出桶的数组的指针
	nextOverflow 	*bmap		// 下一个空闲溢出桶地址
}
```

