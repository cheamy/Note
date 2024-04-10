### 思路

回溯

``` go
func combinationSum3(k int, n int) [][]int {
    res := [][]int{}
    path := []int{}
    sum := 0
    var backtracking func(k int, n int, index int)
    backtracking = func(k int, n int, index int){
        // if sum > n {
        //     return
        // }
        if len(path) == k && sum == n{
            tmp := make([]int, k)
            copy(tmp, path)
            res = append(res, tmp)
            return 
        }
        for i:=index; i <= 9; i++{
            path = append(path, i)
            sum += i
            if sum <= n {
                backtracking(k, n, i+1)
            }
            
            path = path[:len(path)-1]
            sum -= i
        }
    }
    backtracking(k, n, 1)
    return res
}
```

