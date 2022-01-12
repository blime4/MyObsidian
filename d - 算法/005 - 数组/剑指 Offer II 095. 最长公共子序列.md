[[算法-第二遍]]
[[算法-序列]]
[[算法-dp]]

```c++
class Solution {
public:
    int longestCommonSubsequence(string s, string t) {
        int n = s.size(), m = t.size();
        vector<vector<int>> dp(n+1, vector<int>(m+1));
        for(int i=n-1;i>=0;--i){
            for(int j=m-1; j>=0; --j){
                if(s.at(i)==t.at(j)) dp[i][j] = dp[i+1][j+1] +1;
                else dp[i][j] = max(dp[i][j+1], dp[i+1][j]);
            }
        }
        return dp[0][0];
    }
};
```

