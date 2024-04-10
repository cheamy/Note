### 思路

还是用回溯解决分割问题

``` go
var(
    res []string
    path []string
)

func restoreIpAddresses(s string) []string {
    res = []string{}
    path = []string{}
    backtracking(s, 0)
    return res
}

func backtracking(s string, startindex int){
    if len(path) == 4 {
        if startindex == len(s){
            str := strings.Join(path, ".")
            res = append(res, str)
        }
        return
    }
    for i := startindex; i < len(s); i++ {
        if i != startindex && s[startindex] == '0'{
            break
        }
        str := s[startindex : (i+1)]
        num, _ := strconv.Atoi(str)
        if num >= 0 && num <= 255 {
            path = append(path, str)
            backtracking(s, i+1)
            path = path[:len(path)-1]
        }else{
            break
        }
    }
}
```

