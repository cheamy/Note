### 思路

同I, 只是加了一点限制

``` go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    m, n := len(obstacleGrid), len(obstacleGrid[0])
    dp := make([][]int, m)
    for i := range dp {
        dp[i] = make([]int, n)
    }
    if obstacleGrid[0][0] == 1 {
        dp[0][0] = 0
    }else {
        dp[0][0] = 1
    }
    for i := 1; i < m; i++ {
        if obstacleGrid[i][0] == 1 {
            dp[i][0] = 0
        }else {           
            dp[i][0] = dp[i-1][0]
        }
    }
    for i := 1; i < n; i++ {
        if obstacleGrid[0][i] == 1 {
            dp[0][i] = 0
        }else {            
            dp[0][i] = dp[0][i-1]
        }
    }
    for i := 1; i < m; i++ {
        for j := 1; j < n; j++ {
            if obstacleGrid[i][j] == 1 {
                dp[i][j] = 0
            }else {
                dp[i][j] = dp[i-1][j] + dp[i][j-1]
            }
        }
    }
    fmt.Println(dp)
    return dp[m-1][n-1]
}
```

