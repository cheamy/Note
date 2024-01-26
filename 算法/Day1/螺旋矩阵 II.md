### 思路

1. 可以把外围四条边遍历完看做一个循环，那么n方就需要n/2个循环，n为奇数时，独立处理最中间的值。这里要注意每次大循环里面的小循环的边界条件，最方便的方法是采用一样的规则，这里使用左闭右开。

```c++
int loop = n/2;
int startx = 0, starty = 0;
int offset = 1;
int i, j;
while(loop--){
	i = startx;
	j = starty;
	for(j = starty; j < n - offset; j++){
		res[i][j] = count++;
	}
	for(i = startx; i < n-offset; i++){
		res[i][j] = count++;
	}
	for(; j > starty; j--){
		res[i][j] = count++;
	}
	for(; i > startx; i--){
		res[i][j] = count++;
	}
	startx++;
	starty++;
	offset++;
}
if(n%2 == 1) res[n/2][n/2] = count;
```



2. 也可以用另一种思路，就是一次小循环处理完一条边，维护四个边界值：l, r, t, b; 每次小循环更新对应的边界值：

```c++
		int l = 0, r = n-1, t = 0, b = n-1;
        int count = 1;
        while(l<=r && t<=b){
            //l->r
            for(int i = l; i <= r; i++){
                res[t][i] = count++;
            }
            t++;
            //t->b
            for(int i = t; i <= b; i++){
                res[i][r] = count++;
            }
            r--;
            //r->l
            for(int i = r; i >= l; i--){
                res[b][i] = count++;
            }
            b--;
            //b->t
            for(int i = b; i >= t; i--){
                res[i][l] = count++;
            }
            l++;
        }
```

