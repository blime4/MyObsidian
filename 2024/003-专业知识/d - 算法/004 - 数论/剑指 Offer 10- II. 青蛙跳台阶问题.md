```c++

class Solution {
public:
    int numWays(int n) {
        long a = 1, b = 1, c = 1;
        for(int i=0;i<n-1;i++){
            a = b;
            b = c;
            c = a + b;
            c = c %1000000007;
        }
        return c;
    }
};
```