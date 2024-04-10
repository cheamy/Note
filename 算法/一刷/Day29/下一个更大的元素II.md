### 思路

使用单调栈两次遍历即可

``` go
func nextGreaterElements(nums []int) []int {
    n := len(nums)
    res := make([]int, n)
    for i := 0; i < n; i++ {
        res[i] = -1
    }
    stack := []int{0}
    
    for i := 1; i < 2*n; i++ {
        top := stack[len(stack)-1]
        for len(stack) > 0 && nums[i%n] > nums[top] {
            res[top] = nums[i%n]
            stack = stack[:len(stack)-1]
            if len(stack) > 0 {
                top = stack[len(stack)-1]
            }
        }
        stack = append(stack, i%n)
    }
    // for i := 1; i < len(stack); i++ {
    //     if nums[stack[i]] < nums[stack[0]]{
    //         res[stack[i]] = nums[stack[0]]
    //     }
        
    // }
    return res
}
```

