### 思路

首先初始化res, 用map存储nums1位置, 单调栈联合map更新res

``` go
func nextGreaterElement(nums1 []int, nums2 []int) []int {
    n := len(nums1)
    res := make([]int, n)
    for i:=0; i < n; i++ {
        res[i] = -1
    }
    // map存储nums1位置
    m := map[int]int{}
    for i := 0; i < len(nums1); i++ {
        m[nums1[i]] = i
    }
    // 单调栈获得nums2下一个最大位置,并联合map更新res
    stack := []int{nums2[0]}
    for i := 1; i < len(nums2); i++ {
        top := stack[len(stack)-1]
        for len(stack) > 0 && nums2[i] > top {
            if index, ok := m[top]; ok { 
                res[index] = nums2[i]
            }
            stack = stack[:len(stack)-1]
            if len(stack) > 0 {
                top = stack[len(stack)-1]
            }

        }
        stack = append(stack, nums2[i])
    }
    return res
}
```

