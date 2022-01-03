[[算法-困难]]
[[算法-序列]]
[[算法-第一遍]]
[[算法-dp]]

![[Pasted image 20211230010522.png]]

```c++
class Solution {
public:
    int numDistinct(string s, string t) {
        int n = s.size(), m = t.size();
        if(n==m) return s==t ? 1 : 0;
        if(n<m) return 0;
        vector<vector<long>> dp(m+1, vector<long>(n+1));
        for(int j = 0; j <= n ; ++j) dp[0][j] = 1;
        for(int i = 0; i < m ; ++i){
            for(int j = 0; j < n; ++j){
                if(s.at(j)==t.at(i)) dp[i+1][j+1] = dp[i][j] + dp[i+1][j];
                else dp[i+1][j+1] = dp[i+1][j];
            }
        }
        return dp[m][n];
    }
};
```

与 [[115. 不同的子序列]] 同一题