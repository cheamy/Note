### 思路

秒

``` go
var(
    res [][]int
    path []int
)

func subsetsWithDup(nums []int) [][]int {
    sort.Ints(nums)
    res = [][]int{}
    path = []int{}
    backtracking(nums, 0)
    return res
}

func backtracking(nums []int, start int){
    tmp := make([]int, len(path))
    copy(tmp, path)
    res = append(res, tmp)

    for i := start; i < len(nums); i++ {
        if i > start && nums[i] == nums[i-1]{
            continue
        }
        path = append(path, nums[i])
        backtracking(nums, i+1)
        path = path[:len(path)-1]
    }
}
```

