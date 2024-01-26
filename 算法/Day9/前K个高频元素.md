

### 思路

看到前k个，第一想法是大根堆， 但是考虑一下，大根堆的话，就要建立一个长度为len(nums)的堆， 空间复杂度和时间复杂度都不好。那么有什么办法是维护容量为k的堆呢，因为要保留的是前k个最大的，那就是说插入的时候需要弹出的是最小的那部分，所以可以使用小顶堆

``` go
func topKFrequent(nums []int, k int) []int {
	map_num := map[int]int{}
	//记录每个元素出现的次数
	for _, item := range nums {
		map_num[item]++
	}
	h := &IHeap{} //这里返回的是指针，用于对底层数据的操作，而不是值传递
	heap.Init(h)
	for key, value := range map_num {
		heap.Push(h, [2]int{key, value})
		if h.Len() > k {
			heap.Pop(h)
		}
	}
	res := make([]int, k)
	for i := 0; i < k; i++ {
		res[k-i-1] = heap.Pop(h).([2]int)[0]
	}
	return res
}

type IHeap [][2]int

func (h IHeap) Len() int {
	return len(h)
}

func (h IHeap) Less(i, j int) bool {
	return h[i][1] < h[j][1]
}

func (h IHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h *IHeap) Push(x interface{}) {
	*h = append(*h, x.([2]int))
}

func (h *IHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[:n-1]
	return x
}
```

go的heap需要实现sort的所有接口，包括Len(), Less(), Swap(), 以及Push()和Pop()

实现之后就可以使用heap.Init()来保证插入和弹出行为能够满足堆的要求了。