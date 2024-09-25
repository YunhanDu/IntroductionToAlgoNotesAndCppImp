```c++
#include <iostream>
#include <vector>
#include <unordered_set>
#include <stack>

using std::vector;
using std::unordered_set;
using std::stack;
class Solution {
private:
vector<int> path;
unordered_set<int> uset;
public:

    // iterative DFS
    bool DFSIterative(vector<vector<int>>& choices) {
        // stk用来存下标
        stack<int> indStk;
        int curIndex = 0;
        int size = choices.size();
        while (true) {
            // 处理curIndex越界
            if (path.size() < size && curIndex >= choices[path.size()].size()) {
                if (!indStk.empty()) {
                    curIndex = indStk.top() + 1;
                    uset.erase(choices[path.size()-1][indStk.top()]);
                    indStk.pop();
                    path.pop_back();
                    continue;
                } else return false;
            }
            // 当前元素和上一次的选择有冲突，换下一个节点
            if (uset.find(choices[path.size()][curIndex]) != uset.end()) {
                ++curIndex;
                continue;
            }
            // 没有冲突
            indStk.push(curIndex);
            path.emplace_back(choices[path.size()][curIndex]);
            uset.insert(path.back());
            if (path.size()== size) {
                // 我们只需要一个答案；
                break;
            } else curIndex = 0; 
        }
        return true;

    }
    vector<int> GetResult(vector<vector<int>>& choices) {
        if (DFSIterative(choices)) {
            return path;
        } else return {};
    }
    
};

int main()
{
    vector<int> resource = {5, 6, 9, 12, 16, 14, 18};
    vector<vector<int>> choices = {{5, 9, 16, 18}, {6, 12, 14, 18}, {6, 9, 16}, {5, 6, 14}};
    // display the final array, can be deleted 
    Solution sol;
    vector<int> result = sol.GetResult(choices);
    if (result.empty()) {
        std::cout << "no answer" << "\n";
        return 0;
    }
    for (auto& ele : result) {
        std::cout << ele << "";
    }
}
```

