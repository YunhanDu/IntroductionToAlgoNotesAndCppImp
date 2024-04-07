# 算法导论Ch8线性时间排序笔记与相应算法c++实现

作者： Claude Du

归并排序与堆排最差情形的运行时间上界为$O(nlgn)$ ，快排的平均运行时间上界为$O(nlgn)$。

并且以上三种算法的共同性质：

1. 运行时间上界也都为$\Omega(nlgn)$。
2. 排序顺序由输入元素之间的比较来决定。

我们把这种由性质2决定的算法称为比较排序，当前所算导中所引入的所有排序算法都是比较排序（归并排序，堆排，快排，插入排序，冒泡排序）。比较排序的最差情形运行时间为 $\Omega(nlgn)$ , 8.1节将证明该结论。

8.2节、8.3节和8.4节将讨论三种非比较排序且线性时间复杂度的排序算法：计数排序、基数排序和桶排序。

## 8.1 排序的下界

为了讨论比较算法的下界，我们不失一般性的假设**输入序列 $\left \{a_{1}, a_{2}, ...,a_{n}\right \}$ 的所有元素都是互异的** 。在该假设前提下， 比较操作 $a_{i} \leq a_{j}$ ，   $a_{i} \geq a_{j}$  ， $a_{i} \lt a_{j}$  和  $a_{i} \gt a_{j}$ 都是等价的，因为通过以上4种比较操作得到 的$a_{i}$ 和 $a_{j}$ 相对次序信息是相同的。因此我们进一步假设所有的比较操作都采用   $a_{i} \leq a_{j}$ 的形式。

### 决策树模型（The decision-tree model）

比较排序可以被抽象为一棵决策树。决策树是一棵完全二叉树（每个节点要么是叶子，要么有两个孩子），它可以表示给定输入规模情况下，某一特定排序算法对所有元素的比较操作。其中，控制，数据移动等其他操作都被忽略了。下图呈现了2.1节插入排序作用于3元素的输入序列的决策时情况：

![](./figure8_1.PNG)



最坏情况的下界





## 8.2 计数排序

计数排序假设输入规模为n的数组中的每一个元素都为小于等于k的非负整数，它的运行时间为 $\Theta(n+k)$ ，因此当 $k= O(n)$ ，计数排序的运行时间为 $\Theta(n)$。

其c++实现如下：

```c++
#include <string>
#include <iostream>
#include <vector>
#include <algorithm>
using std::vector;

class Solution {
private:
public:
    vector<int> CountingSort(vector<int>& arr) {
        int k = 0;
        for (int i = 0; i < arr.size(); ++i) {
            if (arr[i] > k) k = arr[i];
        }
        
        vector<int> bArr(arr.size(), 0);
        vector<int> cArr(k + 1, 0);
        for (int j = 0; j < arr.size(); ++j) {
            cArr[arr[j]] = cArr[arr[j]] + 1;
        }
        for (int i = 1; i <= k; ++i) {
            cArr[i] = cArr[i] + cArr[i - 1];
        }        
        // cArr[i] now contains the numner of elements less than or equal to i.
        // copy A to B, starting from the end of A
        // it takes \Theta(n)
        for (int j = arr.size() - 1; j >= 0; --j) {
            bArr[cArr[arr[j]]-1] = arr[j];
            cArr[arr[j]] = cArr[arr[j]] - 1; // to handle duplicate values
        }
        return bArr;
    }
};
```

总运行时长为$\Theta(n)$ 。计数排序另一大优点是它是稳定的。我们接下来简要证明下其稳定性：

计数排序是稳定的另一种数学表述：如果$arr[n]= arr[m]$ 且 $n \lt m$ ，经过计数排序后，$arr[n]$ 的新位置 $n^{'}$ 和$arr[m]$ 的新位置 $m^{'}$ 一定有 $n^{'}\lt m^{'}$ 。

证明该表述正确：首先在18行for循环中，从左往右进行同元素值计数统计时，刚刚扫描完数组小标$n$ 时的$count_n = cArr[arr[n]]$ 一定小于刚刚扫描完数组小标$m$ 时的$count_m = cArr[arr[m]]$,   经过 第21行的for循环后，这一关系依然得到保持 , 两个计数$count_n, count_m$ 都增加了相同的数值 $cArr[arr[m]-1]$，在第26行的for循环中，我们从右往左扫描$arr$，当$ cArr[arr[m]] = count_m$ 时，$arr[count_m]$ 被赋值 $arr[m]$, 即 $m' = count_m$ 。同理， $n' = count_n$ ， 前面已知 $count_n \lt count_m$ ，于是 $n^{'}\lt m^{'}$ ，稳定性得证 。

计数排序的稳定性之所以重要的另一个原因是：计数排序常常会被用作基数排序（Radix Sort）的子程序(subroutine)。

## 8.3 基数排序（Radix Sort）

讲真，书中讲解卡片排序机的例子我没看懂（澳门皇家赌场。。。在线发牌的核心技术？），还是看看d位10进制数字情形下的基数排序吧。

 如下图所示【原书第四版图8.3】：

![](./figure8_3.PNG)

基数排序是先按照最低有效位（the least significant  digit）来进行一轮稳定排序，再按照次最低有效位再进行一轮稳定排序，直到对所有的d位数字都完成了排序，此时便已排序完成。

伪代码实现如下：

```pseudocode
RADIX-SORT.A; n; d /
1 for i D 1 to d
2 	use a stable sort to sort array A[1:n] on digit i 
```

在基数排序中使用会另一个稳定的排序子程序，通常计数排序会被选为该子程序。

10进制情形下的Radix排序c++实现如下，其中稳定排序使用了计数排序：

```c++
// author: Claude Du
#include <string>
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
using std::vector;

class Solution {
private:
    // get digit i
    int dthDigit(int val, int d) {
        int remainder = pow(10, d);
        int denominator = pow(10, (d-1));
        return ((val % remainder) / denominator);
    }
    // countingsort arr on digit d
    vector<int> CountingSort(vector<int>&arr, int d) {
        int k = 9;
        vector<int> bArr(arr.size(), 0);
        vector<int> cArr(k + 1, 0);
        for (int j = 0; j < arr.size(); ++j) {
            int cIndex = dthDigit(arr[j], d);
            cArr[cIndex] = cArr[cIndex] + 1;
        }
        // cArr[i] now contains the number of elements equal to i.
        for (int i = 1; i <= k; ++i) {
            cArr[i] = cArr[i] + cArr[i - 1];
        }
        // cArr[i] now contains the numner of elements less than or equal to i.
        // copy A to B, starting from the end of A
        for (int j = arr.size() - 1; j >= 0; --j) {
            int cIndex = dthDigit(arr[j], d);
            bArr[cArr[cIndex] - 1] = arr[j]; // to handle duplicate values
            --(cArr[cIndex]); 
        }
        return bArr;
    }
public:
    void RadixSort(vector<int>& arr, int d) {
        for (int i = 1; i <= d; ++i) {
            // use a stable sort to sort arr on digit i
            arr = CountingSort(arr, i);
        }

    }
};
// here is a case to test RadixSort
int main()
{
    Solution sol;
    vector<int> arr = {3, 9, 6, 8, 4, 7, 3, 10, 2, 160, 212, 192, 43, 169, 320, 71};
    sol.RadixSort(arr, 3);
    for (auto& ele : arr) {
        std::cout << ele << " ";
    }
    std::cout << "\n";
}
```

### 引理8.3

给定n个d位数，其中每一个数位有k个可能的取值。如果基数排序使用的稳定排序耗时$\Theta(n + k)$ ，那么它就可以在 $\Theta(d(n + k))$ 时间内将这些数排好序。

### 引理8.4

给定n个b位数和任何正整数 $r \leq b$ , 如果 RadixSort 使用的稳定排序算法对数据取值间是 0 到 k 的输入进行排序耗时 $\Theta(n+k)$ ，那么它就可以在 $\Theta((b/r)(n + 2^r))$ 时间内将这些数排好序



## 8.4 桶排序

桶排序的假设前提：输入数据均匀分布，即输入是由一个随机过程产生的，该过程将元素均匀、独立地分布在[0, 1)区间上。

桶排序的c++实现与单个用例简单校验如下：

```c++
// author: Claude Du
#include <iostream>
#include <vector>
#include <list>
#include <cmath>
using std::vector;
using std::list;
class Solution {
private:
    void InsertSort(list<float>& bucket) {
        if (bucket.empty()) return;
        list<float>::iterator it =bucket.begin();
        ++it;
        for (; it != bucket.end(); ++it) {
            float val = *it;
            for (list<float>::iterator it2 = bucket.begin(); it2 != it; ++it2) {
                if ((*it2) > val) {
                    bucket.erase(it);
                    bucket.insert(it2, val);    
                    break;
                }
            }
        }
    }
public:
    void BucketSort(vector<float>& arr) {
        int n = arr.size();
        vector<list<float>> bArr(n, list<float>{});
        for (int i = 0; i < n; ++i) {
            bArr[floorf(n*arr[i])].emplace_back(arr[i]);           
        }
        for (int i = 0; i < n; ++i) {
            InsertSort(bArr[i]);
        }
        // concatenate the lists bArr[0], ..., B[n-1] together in order
        int index = 0;
        for (int i = 0; i < n; ++i) {
            if (bArr[i].empty()) continue;
            for (auto it = bArr[i].begin(); it != bArr[i].end(); ++it) {
                arr[index] = *it;
                ++index;
            }
        }
    }
};

int main()
{
    Solution sol;
    vector<float> arr = {0.78, 0.17, 0.39, 0.26, 0.72, 0.94, 0.21, 0.12, 0.23, 0.68};

    sol.BucketSort(arr);
    for (auto& ele : arr) {
        std::cout << ele << " ";
    }
    std::cout << "\n";
}
```

