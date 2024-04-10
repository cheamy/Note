### 思路

``` go
var (
    res     [][]int
    path    []int
    sum     int
)

func backtracking(candidates []int, target int, index int){
    if sum == target {
        tmp := make([]int, len(path))
        copy(tmp, path)
        res = append(res, tmp)
        return
    }

    for i := index; i < len(candidates) && sum + candidates[i] <= target; i++ {
        if i > index && candidates[i] == candidates[i-1] {
            continue
        }
        path = append(path, candidates[i])
        sum += candidates[i]
        backtracking(candidates, target, i+1)
        sum -= candidates[i]
        path = path[:len(path)-1]
    }
}

func combinationSum2(candidates []int, target int) [][]int {
    sort.Ints(candidates)
    res = [][]int{}
    path = []int{}
    sum = 0
    backtracking(candidates, target, 0)
    return res
}
```

新的剪枝方式, 排序后如果在当前的层的for循环内, 当前数字的值已经在当前层遍历过, 则可以剪枝