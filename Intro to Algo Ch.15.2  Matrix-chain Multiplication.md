Intro to Algo Ch.14.2  Matrix-chain Multiplication

I quite enjoy this topic. Here is my code of the bottom-up approach.

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using std::vector;

class Solution {
private:
    // dp[i][j]: the min operations of scalar multiplication of A_{i:j}
    vector<vector<int>> dp;
    // location[i]: the cut location of a rod of length i to get the max revenue
    vector<vector<int>> location;
public:
    int MatrixChainOrderBottomUp(vector<int>& dimensions) {
        dp.clear();
        location.clear();
        if (dimensions.size() <= 2) return 0;
        int len = dimensions.size();
        // initialization
        dp = vector<vector<int>>(len, vector<int>(len, INT32_MAX));
        location = vector<vector<int>>(len, vector<int>(len, INT32_MAX));   
        // initialization
        for (int i = 1; i <= len-1; ++i) dp[i][i] = 0;
        for (int length = 2; length <= len - 1; ++length) {
            for (int i = 1; i <= len - length; ++i) {
                int j = i + length -1;
                for (int k = i; k < j; ++k) {
                    int temp = dp[i][k] + dp[k+1][j] + dimensions[i-1]*dimensions[k]*dimensions[j];
                    if (temp < dp[i][j]) {
                        dp[i][j] = temp;
                        location[i][j] = k;
                    }
                }
            }
        }
        PrintOptimalParenthesis(1, len - 1);
        std::cout << "\n";
        return dp[1][len - 1];
    }
    void PrintOptimalParenthesis(int i, int j) {
        if (i==j) std::cout << "A" << i;
        else {
            std::cout << "(";
            PrintOptimalParenthesis(i, location[i][j]);
            PrintOptimalParenthesis(location[i][j] + 1, j);
            std::cout << ")";            
        }
    }
};
    
int main()
{
    Solution sol;
    // prices[i]: the revenue of each whole rod of length i
    vector<int> dimensions{30, 35, 15, 5, 10, 20, 25};
    int minCost = sol.MatrixChainOrderBottomUp(dimensions);
    std::cout <<  "The least computing cost of a matrix chain muliplication with a length " << dimensions.size() - 1 << ": "<< minCost << std::endl;
}
```

Here is the output result:

```shell
((A1(A2A3))((A4A5)A6))
The least computing cost of a matrix chain with a length 6: 15125
```

Here is my code of the memorized top-down approach.

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using std::vector;

class Solution {
private:
    // dp[i][j]: the min operations of scalar multiplication of A_{i:j}
    vector<vector<int>> dp;
    // location[i]: the cut location of a rod of length i to get the max revenue
    vector<vector<int>> location;
public:
    void PrintOptimalParenthesis(int i, int j) {
        if (i==j) std::cout << "A" << i;
        else {
            std::cout << "(";
            PrintOptimalParenthesis(i, location[i][j]);
            PrintOptimalParenthesis(location[i][j] + 1, j);
            std::cout << ")";            
        }
    }
    int MemorizedMaxtrixChainOrder(vector<int>& dimensions) {
        dp.clear();
        location.clear();
        if (dimensions.size() <= 2) return 0;
        int len = dimensions.size();
        // initialization
        dp = vector<vector<int>>(len, vector<int>(len, INT32_MAX));
        location = vector<vector<int>>(len, vector<int>(len, INT32_MAX)); 
        int ans = MemorizedMatrixChain(dimensions, 1, len - 1);
        PrintOptimalParenthesis(1, len - 1);
        std::cout << "\n";
        return ans;
    }
    int MemorizedMatrixChain(vector<int>& dimensions, int i, int j) {
        if (i == j) {
            dp[i][j] = 0;
            return dp[i][j];
        }
        if (dp[i][j] != INT32_MAX) return dp[i][j];
        for (int k = i; k < j; ++k) {
            int temp = MemorizedMatrixChain(dimensions, i, k) + MemorizedMatrixChain(dimensions, k + 1, j) + dimensions[i-1]*dimensions[k]*dimensions[j];
            if (temp < dp[i][j]) {
                dp[i][j] = temp;
                location[i][j] = k;
            }
        }
        return dp[i][j];
    }
};
    
int main()
{
    Solution sol;
    // prices[i]: the revenue of each whole rod of length i
    vector<int> dimensions{30, 35, 15, 5, 10, 20, 25};
    int minCost1 = sol.MemorizedMaxtrixChainOrder(dimensions);
    std::cout <<  "The least computing cost of a matrix chain muliplication with a length " << dimensions.size() - 1 << ": "<< minCost1 << std::endl;
}
```

Here is the output result:

```shell
((A1(A2A3))((A4A5)A6))
The least computing cost of a matrix chain muliplication with a length 6: 15125
```

