# Intro to Algo Ch.15 Dynamic Programming Exercises

14.1-1 is just a simple high-school level mathematical question

14.1-2 I have written an counter example on my ipad.

14.1-3

A pretty easy question.  The only thing which needs to be modified is the recurrence formula:



```c++
#include <iostream>
#include <vector>
#include <algorithm>
using std::vector;

class Solution {
private:
    // dp[i]: the max revenue of a rod of length i + 1 after evaluation
    vector<int> dp;
public:
    int BottomUpCutRodWithCost(vector<int> &prices, int cost) {
        
        int len = prices.size();
        if (len <= 1) return prices[len];
        dp.clear();
        
        dp = vector<int>(len , INT32_MIN);
        // initialization
        dp[0] = 0;
        for (int i = 1; i < len; ++i) {
            for (int j = 1; j <= i; ++j) {
                if ( j != i) {
                    dp[i] = std::max(dp[i], prices[j] + dp[i - j] - cost);
                } else dp[i] = std::max(dp[i], prices[j]);
            }

        }
        return dp[len - 1];
    }
    int getAnswer(int n) {
        if (n < 0|| n >= dp.size()) return INT32_MIN;
        return dp[n];
    }

};
    
int main()
{
    Solution sol;
    // prices[i]: the revenue of each whole rod of length i
    vector<int> prices{0, 1, 5, 8, 9, 10, 17, 17, 20, 24, 30};
    int ans = sol.BottomUpCutRod(prices, 1);
     std::cout <<  "The max revenue of a rod with a length " << prices.size() - 1 << ": "<< ans << std::endl;
    for (int i = 1; i < prices.size(); ++i) {
        std::cout << "The max revenue of a rod with a length " << i << ": "<< sol.getAnswer(i) << "\n";
    }
}
```

14.1-4

Here is my answer.

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using std::vector;

class Solution {
private:
    // dp[i]: the max revenue of a rod of length i after evaluation
    vector<int> dp;
public:
    // 14.1-4 MEMOIZED-CUT-ROD-AUX so that their for loops go up to only n/2
    int MemorizedCutRod(vector<int> &prices) {
        dp.clear();
        dp = vector<int>(prices.size(), INT32_MIN);
        // initialization
        dp[0] = 0;
        return MemoizedCutRodAux(prices, prices.size() - 1);
    }
    int MemorizedCutRodAux(vector<int> &prices, int n) {
        if (dp[n] >= 0) return dp[n];
        dp[n] = prices[n];
        for (int i = 1; i <= n/2; ++i) {
            dp[n] = std::max(dp[n], MemoizedCutRodAux(prices, i) + MemoizedCutRodAux(prices, n - i));
        }
        return dp[n];
    }
	// 14.1-4 CUT-ROD so that their for loops go up to only n/2
    int CutRod(vector<int> &prices, int n) {
        if (n == 0) return 0;
        int ans = prices[n];
        for (int i = 1; i <= n/2; ++i) {
            ans = std::max(ans, CutRod(prices, i) + CutRod(prices, n - i));
        }
        return ans; 
    }
    int getAnswer(int n) {
        if (n < 0|| n >= dp.size()) return INT32_MIN;
        return dp[n];
    }
};
    
int main()
{
    Solution sol;
    // prices[i]: the revenue of each whole rod of length i
    vector<int> prices{0, 1, 5, 8, 9, 10, 17, 17, 20, 24, 30};
    int ans = sol.CutRod(prices, 9);
    std::cout <<  "The max revenue of a rod with a length " << 9 << ": "<< ans << std::endl;
    int ansByMem = sol.MemorizedCutRod(prices);
    std::cout <<  "The max revenue of a rod with a length " << prices.size() - 1 << ": "<< ansByMem << std::endl;
    
    for (int i = 1; i < prices.size(); ++i) {
        std::cout << "The max revenue of a rod with a length " << i << ": "<< sol.getAnswer(i) << "\n";
    }
}
```

14.1-5

we  modify MEMOIZED-CUT-ROD based on the original memoized-cut-rod instead of modified version in 14.1-4

```C++
#include <iostream>
#include <vector>
#include <algorithm>
using std::vector;

class Solution {
private:
    // dp[i]: the max revenue of a rod of length i after evaluation
    vector<int> dp;
    // location[i]: the cut location of a rod of length i to get the max revenue
    vector<int> location;
public:
    int getAnswer(int n) {
        if (n < 0|| n >= dp.size()) return INT32_MIN;
        return dp[n];
    }
    // 14.1-5 Modify MEMOIZED-CUT-ROD-AUX D to return not only the value but the actual solution.
    int MemoizedCutRod(vector<int> &prices) {
        dp.clear();
        location.clear();
        dp = vector<int>(prices.size(), INT32_MIN);
        location = vector<int>(prices.size(), INT32_MAX);
        // initialization
        dp[0] = 0;
        int ans = MemoizedCutRodAux(prices, prices.size() - 1);
        PrintCutRodSol(prices.size() - 1);
        return ans;
    }
    void PrintCutRodSol(int n) {
        while (n > 0 && (n < location.size())) {
            std::cout << "The cut location is " << location[n] << std::endl;
            n = n - location[n];
        }
    }
    int MemorizedCutRodAux(vector<int> &prices, int n) {
        if (dp[n] >= 0) return dp[n];
        dp[n] = prices[n];
        int locate = n;
        for (int i = 1; i <= n; ++i) {
            int temp = MemoizedCutRodAux(prices, n - i);
            if (prices[i] + temp > dp[n]) {
                dp[n] =  prices[i] + temp;
                locate = i;
            }
        }
        location[n] = locate;
        return dp[n];
    }
};
    
int main()
{
    Solution sol;
    // prices[i]: the revenue of each whole rod of length i
    vector<int> prices{0, 1, 5, 8, 9, 10, 17, 17, 20, 24, 30};
    int ansByMem = sol.MemorizedCutRod(prices);
    sol.PrintCutRodSol(5);
    std::cout <<  "The max revenue of a rod with a length " << prices.size() - 1 << ": "<< ansByMem << std::endl;
    
    for (int i = 1; i < prices.size(); ++i) {
        std::cout << "The max revenue of a rod with a length " << i << ": "<< sol.getAnswer(i) << "\n";
    }
}
```

14.1-6

The code part seems too easy for me, the subproblem graph needs my attention.



I have come up with an interesting question. 