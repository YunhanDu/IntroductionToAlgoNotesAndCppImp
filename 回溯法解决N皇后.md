对称二叉树

递归法

判断二叉树是否对称，要比较的是根节点的左子树和右子树是不是互相翻转的

![image-20240118162428934](D:\离职计划准备\可信专业级准备\科目一算法联系总结\image-20240118162428934.png)

N皇后问题的自行想出来的较差算法

```c++
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
#include <map>
#include <stack>
#include <algorithm>
#include <stdio.h>  // puts
using std::vector;
using std::unordered_map;
using std::string;
using std::pair;
class Solution {
    // 棋盘中true的位置可以下棋，false的位置不能下棋
    vector<vector<pair<bool, int>>> board;
    vector<int> path;
    vector<vector<string>> resultSet;
    vector<string> result;
    void BlockPosition(int pos, const int n) {
        int size = path.size();
        for (int i = size; i < n; ++i) {
            board[i][pos].first = false;
            board[i][pos].second++;
        }
        for (int j = 0; j < pos; ++j) {
            if (size + pos - 1 - j > n -1) continue;
            board[size + pos - 1 - j][j].first = false;
            board[size + pos - 1 - j][j].second++;
        } 
        for (int k = pos + 1; k < n; ++k) {
            if (size - pos - 1 + k > n -1) break;
            board[size - pos - 1 + k][k].first = false;
            board[size - pos - 1 + k][k].second++;
        }
    }
    void DeleteBlock(int pos, const int n) {
        int size = path.size();
        for (int i = size; i < n; ++i) {
            board[i][pos].second--;
            if (board[i][pos].second == 0) board[i][pos].first = true;
        }
        for (int j = 0; j < pos; ++j) {
            if (size + pos - 1 - j > n -1) continue;
            board[size + pos - 1 - j][j].second--;
            if (board[size + pos - 1 - j][j].second == 0) board[size + pos - 1 - j][j].first = true;
        } 
        for (int k = pos + 1; k < n; ++k) {
            if (size - pos - 1 + k > n -1) break;
            board[size - pos - 1 + k][k].second--;
            if (board[size - pos - 1 + k][k].second == 0) board[size - pos - 1 + k][k].first = true;
        }
    }
    void DealWithResult() {
        for (int i = 0; i < path.size(); ++i) {
            result[i][path[i]] = 'Q';
        }
    }
    void RecoverResult() {
        for (int i = 0; i < path.size(); ++i) {
            result[i][path[i]] = '.';
        }
    }
    
    void BackTrack(const int n) {
        if (path.size() == n) {
            DealWithResult();
            resultSet.emplace_back(result);
            RecoverResult();
            return;
        }
        for (int i = 0; i < n; ++i) {
            if (board[path.size()][i].first == false) continue;
            path.emplace_back(i);
            BlockPosition(i, n);
            BackTrack(n);
            DeleteBlock(i, n);
            path.pop_back();
        }
        return;
    }
public:
    vector<vector<string>> solveNQueens(int n) {
        board = vector<vector<pair<bool,int>>>(n, vector<pair<bool, int>>(n, {true, 0}));
        path.clear();
        resultSet.clear();
        result = vector<string>(n, string(n, '.'));
        BackTrack(n);
        return resultSet;

    }
};
    
int main()
{
    Solution s;
    vector<vector<string>> solSet = s.solveNQueens(4);
    for (auto& sol : solSet) {
        for (auto& row : sol) {
            std::cout << row << std::endl;
        }
        std::cout << std::endl;

    }
}
```

