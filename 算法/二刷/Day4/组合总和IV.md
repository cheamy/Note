### 思路

完全背包, 最内层遍历顺序为从前到后.

对于遍历顺序, 对dp数组影响重大

1. 先遍历物品, 后遍历容量, 记住此时物品放入是有顺序的, 因此能求出排列数
2. 先遍历容量, 再遍历物品, 此时不管物品时如何放入的, 因此能求出组合数

``` go
func combinationSum4(nums []int, target int) int {
    dp := make([]int, target+1)
    dp[0] = 1
    
    for j := 0; j <= target; j++ {
        for i := 0; i < len(nums); i++ {
            if(j-nums[i] >= 0){
                dp[j] += dp[j-nums[i]]
            }
        }
    }
    return dp[target]
}
```

