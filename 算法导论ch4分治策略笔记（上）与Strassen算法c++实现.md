# 算法导论ch4分治策略笔记（上）与Strassen算法c++实现
作者：Claude Du
算导第四章的内容较多，无论是数学理论部分还是代码实现部分都太多了，还是得拆成上下两部分：

第一部分主要内容是分治策略的两个实例：

1. 用分治法处理最大子数组问题
2. Strassen算法

本篇笔记只覆盖第一部分，第二部分主要是讨论递归式的求解，本篇笔记不会涉及很多，分治策略的基本概念和一个经典实例已在算法导论ch2笔记中讨论过

https://3ms.huawei.com/km/blogs/details/15196581?l=zh-cn。本篇章便不重复叙述了

## 4.1 最大子数组问题（来自算导第三版，算导第四版英文版已删该小章节）

该股票问题具体描述请见第三版原书, 其中有个重要的概念，**最大子数组**（Maximum subarray）。

**最大子数组**（Maximum subarray）：数组A中的和最大的非空连续子数组, 被称为数组A的最大子数组

算导第三版的求解该问题时，先将该股票问题进行了问题变换，先将该问题转化成最大子数组和问题(LeetCode上一模一样的问题https://leetcode.cn/problems/maximum-subarray/description/)

对于变换后的问题，使用分治策略的求解方法, 先将子数组$arr[low:high]$划分为两个规模尽量相等的子数组 $arr[low:mid]$ 和 $arr[mid:high]$, 其中 $mid =(mid + high)/2$  。则 $arr[low:high]$ 的最大子数组 $arr[i:j]$ 必然是以下三种情况的其中一个：

1.arr[i:j]完全位于arr[low:mid]中， 其中 $low \leq i \leq j \leq mid$ 。

2.arr[i:j]完全位于arr[mid + 1:high]中， 其中 $mid + 1 \leq i \leq j \leq high$ 。

3.跨越了中点， 其中 $low \leq i \leq mid \lt j \leq high$ 。

我们在这三种情况中取子数组的和最大的即可。

按照分治思想的经典三步走一把：

1.**分解**：将未发现最大子数组的子数组 $arr[low:high] $ 分解成两个相邻,规模尽量相等的子数组，$arr[low:mid]$ 和 $arr[mid+1:high]$ , 其中 $mid =(mid + high)/2$ 。

2.**解决**：递归的求出两个子数组$arr[low:mid]$ 和 $arr[mid+1:high]$ 自己的最大子数组，保存两个最大子数组位置（数组两端的索引）和自身元素和的信息, c++中可以用tuple来存储这些信息。

3.**合并**：求出跨越中点mid的最大子数组，将其元素和与$arr[low:mid]$ 和 $arr[mid+1:high]$ 的最大子数组元素和进行比较，元素和最大的子数组为 $arr[low:high] $的最大子数组。

三步已走， 其代码实现也很清晰了，c++代码实现如下：

```c++
// author: Claude Du
class Solution
{
private:
    // tuple[0] refers to maximum subarray's startIndex
    // tuple[1] refers to maximum subarray's endIndex
    // tuple[2] refers to the sum of all of maximum subarray's elements
    tuple<int, int, int> FindMaximumSubarray(vector<int> &arr, int low, int high)
    {
        if (low == high) return {low, high, arr[low]}; //base case: only one element
        // 1.divide
        int mid = low + (high - low) / 2;
        // 2. conquer
        tuple<int, int, int> leftMax = FindMaximumSubarray(arr, low, mid);
        tuple<int, int, int> rightMax = FindMaximumSubarray(arr, mid + 1, high);
        // 3.combine
        tuple<int, int, int> crossMax = FindMaxCrossingSubarray(arr, low, mid, high);
        // compare the three cases to find the maximum subarray
        if (std::get<2>(leftMax) >= std::get<2>(rightMax) && std::get<2>(leftMax) >= std::get<2>(crossMax)) {
            return leftMax;
        } else if (std::get<2>(rightMax) >= std::get<2>(leftMax) && std::get<2>(rightMax) >= std::get<2>(crossMax)) {
            return rightMax;
        } else return crossMax;
    }
public:
    tuple<int, int, int> FindMaxiumSubarray(vector<int> &arr)
    {
        if (arr.size() == 0)
            return {};
        return FindMaximumSubarray(arr, 0, arr.size() - 1);
    }
};
```

这里有一步关键步骤求出跨越中点mid的最大子数组FindMaxCrossingSubarray也需要实现。

在线性时间内（对于数组arr[low:high]）求出跨越中点的最大子数组是非常容易的，c++实现如下:

```c++
// author: Claude Du
#include <iostream>
#include <vector>
#include <algorithm>
#include <tuple>

using std::tuple;
using std::vector;
tuple<int, int, int> FindMaxCrossingSubarray(vector<int> &arr, int low, int mid, int high)
{
    // Within arr[low: mid], find the maximum subarray with the tail, arr[mid]
    int leftSumMax = INT_MIN; 
    int leftMaxInd = mid;
    int leftSum = 0;
    for (int i = mid; i >= low; --i) {
        leftSum += arr[i];
        if (leftSum > leftSumMax) {
            leftSumMax = leftSum;
            leftMaxInd = i;
        }
    }
    // Within arr[mid + 1: high], find the maximum subarray with the head, arr[mid + 1]
    int rightSumMax = INT_MIN;
    int rightMaxInd = mid + 1;
    int rightSum = 0;
    for (int j = mid + 1; j <= high; ++j) {
        rightSum += arr[j];
        if (rightSum > rightSumMax) {
            rightSumMax = rightSum;
            rightMaxInd = j;
        }
    }
    return {leftMaxInd, rightMaxInd, leftSumMax + rightSumMax};
}
```

显然FindMaxCrossingSubarray的时间复杂度为$\Theta(n)$ , 其中 $n= high - low + 1$ 。FindMaximumSubarray的思路和并归排序有很强的相似性，其时间复杂度也和并归排序相似，毕竟其运行时间 $T(n)$ 的递归式都和并归排序都几乎一样：
$$
T(n) = \left\{\begin{array}{lcl}
\Theta(1) & { n =1}\\
2*T(n/2) + \Theta(n) & {n>1}
\end{array} \right.
$$
可计算得出$T(n)$为 $\Theta(nlgn)$。

整体实现如下

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <tuple>

using std::tuple;
using std::vector;
class Solution
{
private:
    tuple<int, int, int> FindMaxCrossingSubarray(vector<int> &arr, int low, int mid, int high)
    {
        int leftSumMax = INT_MIN; 
        int leftMaxInd = mid;
        int leftSum = 0;
        for (int i = mid; i >= low; --i) {
            leftSum += arr[i];
            if (leftSum > leftSumMax) {
                leftSumMax = leftSum;
                leftMaxInd = i;
            }
        }
        int rightSumMax = INT_MIN;
        int rightMaxInd = mid + 1;
        int rightSum = 0;
        for (int j = mid + 1; j <= high; ++j) {
            rightSum += arr[j];
            if (rightSum > rightSumMax) {
                rightSumMax = rightSum;
                rightMaxInd = j;
            }
        }
        return {leftMaxInd, rightMaxInd, leftSumMax + rightSumMax};
    }
    tuple<int, int, int> FindMaximumSubarray(vector<int> &arr, int low, int high)
    {
        if (low == high) return {low, high, arr[low]}; //base case: only one element
        int mid = low + (high - low) / 2;
        tuple<int, int, int> leftMax = FindMaximumSubarray(arr, low, mid);
        tuple<int, int, int> rightMax = FindMaximumSubarray(arr, mid + 1, high);
        tuple<int, int, int> crossMax = FindMaxCrossingSubarray(arr, low, mid, high);
        if (std::get<2>(leftMax) >= std::get<2>(rightMax) && std::get<2>(leftMax) >= std::get<2>(crossMax)) {
            return leftMax;
        } else if (std::get<2>(rightMax) >= std::get<2>(leftMax) && std::get<2>(rightMax) >= std::get<2>(crossMax)) {
            return rightMax;
        } else return crossMax;
    }
public:
    tuple<int, int, int> FindMaxiumSubarray(vector<int> &arr)
    {
        if (arr.size() == 0)
            return {};
        return FindMaximumSubarray(arr, 0, arr.size() - 1);
    }
};

int main()
{
    vector<int> nums = {13, -3, -25, 20, -3, -16, -23, 18, 20, -7, 12, -5, -22, 15, -4, 7};
    Solution sol;
    tuple<int,int,int> ans = sol.FindMaxiumSubarray(nums);
    std::cout << "MaxiumSubarray's low index: " << std::get<0>(ans) << "\n";
    std::cout << "MaxiumSubarray's high index: " << std::get<1>(ans) << "\n";
    std::cout << "MaxiumSubarray's summation: " << std::get<2>(ans) << "\n";
}
```

这里使用的分治法，个人感觉更多的是刻意的练习，加深对分治法的理解与运用，该分支法仅稍优于暴力求解的方法，像动态规划求解（时间复杂度为 $\Theta(n)$ ，练习4.1-5）或者线段树求解(也是一种分治法，时间复杂度为 $\Theta(lgn)$ )都是更优的求解方法，具体解法可以看LeetCode的题解。

## 4.2 矩阵乘法的Strassen算法（第四版中的Ch4.1和Ch4.2）

我们可以用分治法处理方阵的乘法，先回顾一下方阵乘法数学公式：

$A= (a_{ij})$  和 $B= (b_{ij})$  是 $n \times n$ 的方阵，则 $ C = A\cdot B$ 中的元素  $c_{ij}$ 为：
$$
c_{ij}=\sum_{k = 1}^n a_{ij} \cdot b_{ij}
$$
有 $n^2$ 个元素需要计算， 每个元素要有n个乘法和n-1个加法需要运算。

### 暴力求解

按照公式，我们可以很轻松的用c++实现暴力求解算法

```C++
// author: Claude Du
void SquareMatrixMulitplyBruteForce(vector<vector<int>>& A, vector<vector<int>>& B, vector<vector<int>>& C) {
    if (A.empty() || B.empty()) {
        std::cerr << " invalid arguments!" << std::endl;
        return;
    }

    if (A.size() != B.size() || A[0].size() != A.size() || B[0].size() != B.size()) {
        std::cerr << " invalid arguments " << std::endl;
        return;
    }
    if (C.empty() || C.size() != A.size() || C[0].size() != A.size())
        C = vector<vector<int>>(A.size(), vector<int>(A.size(), 0));
    for (int i = 0; i < C.size(); ++i) 
        for (int j = 0; j < C.size(); ++j)   
            for (int k = 0; k < C.size(); ++k) 
                C[i][j] += A[i][k] * B[k][j];     
}
```

### 一个简单的分治算法

无论是第四版还是第三版的伪代码都仅仅提供了大概的算法指导思路，都是假设$ n = 2^m$ 的 $ n \times n$ 方阵的乘法，无法通用于其他正整数n的情形，要实现任意正整数n的情形有两个方式：

1. 针对任何正整数n, 硬用分治法去分析，递归基要着重分析，我自己的C++实现就用了这个方式。
2. 我们把 $ n \times n$ 方阵扩大成 $2^{\lceil lgn\rceil} \times 2^{\lceil lgn\rceil}$ 方阵, 扩大的后多出来的部分直接填充0，可以直接套用伪代码的算法

我这里会使用方法1实现，方法2则会在Strassen算法中使用。

大致分治算法如下：

1. **分解**：将A, B, C均切成4个大致为 $ n/2 \times n/2$ 的子矩阵：
   $$
   A=\begin{bmatrix} A_{11} & A_{12} \\A_{21} & A_{22} \end{bmatrix} &
   B=\begin{bmatrix} B_{11} & B_{12} \\B_{21} & B_{22} \end{bmatrix} &
   C=\begin{bmatrix} C_{11} & C_{12} \\C_{21} & C_{22} \end{bmatrix}
   $$
   这里矩阵分解推荐使用通过下标计算完成分解，不要真的创建12个新的大致为 $ n/2 \times n/2$ 的子矩阵中间变量。

2. **解决**：则 $ C = A\cdot B$ 可以改写为：
   $$
   \begin{bmatrix} C_{11} & C_{12} \\C_{21} & C_{22} \end{bmatrix} = 
   \begin{bmatrix} A_{11} & A_{12} \\A_{21} & A_{22} \end{bmatrix} \cdot 
   \begin{bmatrix} B_{11} & B_{12} \\B_{21} & B_{22} \end{bmatrix}
   $$
   其中$C_{11}, C_{12}, C_{21}, C_{22}$ 的表达式为：
   $$
   C_{11} = A_{11}\cdot B_{11} + A_{12}\cdot B_{21} \\
   C_{12} = A_{11}\cdot B_{12} + A_{12}\cdot B_{22} \\
   C_{21} = A_{21}\cdot B_{11} + A_{22}\cdot B_{21} \\
   C_{22} = A_{21}\cdot B_{12} + A_{22}\cdot B_{22}
   $$
   我们要递归的求解出 $ A_{11}\cdot B_{11}$ ,  $ A_{12}\cdot B_{21}$ 等8个规模近似于 $ n/2 \times n/2$ 的子矩阵相乘的结果，此问题便得以解决。

   这里递归基分两种情形：

   1. 切出来的A只有1行（包括A只有1个元素）
   2. 切出来的A只有1列

   递归基两种情形的处理直接看C++实现的MatrixMultiplyRecursive函数中recursion basis部分。

3. **合并**：这里并没有任何实质性的步骤

按照这个算法，任意 $ n \times n$ 方阵的乘法的简单分治算法c++代码实现如下（实现难度还是比较大的，而且这个代码还需要优化，函数里的参数太多了）：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
// author: Claude Du
using std::vector;
class Solution
{
private:

    void MatrixMultiplyRecursive(vector<vector<int>>& A, int startAr, int startAc, int endAr, int endAc,
                                 vector<vector<int>>& B, int startBr, int startBc, int endBr, int endBc, 
                                 vector<vector<int>>& C, int startCr, int startCc, int endCr, int endCc) 
    {
        // recursion basis:
        //   1. a single row in A, which indicates a single row in C 
        //   The time complexity for this part is O(1), the highest cost is 2 addtions + 4 multiplications
        if (startAr == endAr) {
            for (int j = startCc; j <= endCc; ++j) {
                for (int k = startAc; k <= endAc; ++k) {
                    C[startCr][j] += A[startCr][k]*B[k - startAc + startBr][j];
                }
            }
            return;
        }
        //  2. a single column in A, for C[startCr:endCr][startCc:endCc]
        //   The time complexity for this part is O(1), the highest cost is 4 multiplications
        if (startAc == endAc) {
            for (int i = startCr; i <= endCr; ++i) {
                for (int j = startCc; j <= endCc; ++j)
                    C[i][j] += A[i][startAc] * B[startBr][j];
            }   
            return;
        }

        // Divide
        int midAr = startAr + (endAr - startAr) / 2, midAc = startAc + (endAc - startAc) / 2;
        int midBr = startBr + (endBr - startBr) / 2, midBc = startBc + (endBc - startBc) / 2;
        int midCr = startCr + (endCr - startCr) / 2, midCc = startCc + (endCc - startCc) / 2;
        // Conquer:
        //   1. C11 += A11*B11
        MatrixMultiplyRecursive(A, startAr, startAc, midAr, midAc,
                                B, startBr, startBc, midBr, midBc,
                                C, startCr, startCc, midCr, midCc);
        //   2. C11 += A12*B21
        MatrixMultiplyRecursive(A, startAr, midAc + 1, midAr, endAc,
                                B, midBr + 1, startBc, endBr, midBc,
                                C, startCr, startCc, midCr, midCc);
        //   3. C12 += A11*B12
        MatrixMultiplyRecursive(A, startAr, startAc, midAr, midAc,
                                B, startBr, midBc + 1, midBr, endBc,
                                C, startCr, midCc + 1, midCr, endCc);
        //   4. C12 += A12*B22
        MatrixMultiplyRecursive(A, startAr, midAc + 1, midAr, endAc,
                                B, midBr + 1, midBc + 1, endBr, endBc,
                                C, startCr, midCc + 1, midCr, endCc);
        //   5. C21 += A21*B11
        MatrixMultiplyRecursive(A, midAr + 1, startAc, endAr, midAc, 
                                B, startBr, startBc, midBr, midBc,   
                                C, midCr + 1, startCc, endCr, midCc); 
        //   6. C21 += A22*B21
        MatrixMultiplyRecursive(A, midAr + 1, midAc + 1, endAr, endAc,  
                                B, midBr + 1, startBc, endBr, midBc,    
                                C, midCr + 1, startCc, endCr, midCc);   
        //   7. C22 += A21*B12
        MatrixMultiplyRecursive(A, midAr + 1, startAc, endAr, midAc,    
                                B, startBr, midBc + 1, midBr, endBc,   
                                C, midCr + 1, midCc + 1, endCr, endCc);
        //   8. C22 += A22*B22
        MatrixMultiplyRecursive(A, midAr + 1, midAc + 1, endAr, endAc,  
                                B, midBr + 1, midBc + 1, endBr, endBc,  
                                C, midCr + 1, midCc + 1, endCr, endCc); 
    }
public:
    void SquareMatrixMultiplyRecursive(vector<vector<int>>& A, vector<vector<int>>& B, vector<vector<int>>& C) {
        if (A.empty() || B.empty()) {
            std::cerr << " invalid arguments!" << std::endl;
            return;
        }

        if (A.size() != B.size() || A[0].size() != A.size() || B[0].size() != B.size()) {
            std::cerr << " invalid arguments " << std::endl;
            return;
        }
        if (C.empty() || C.size() != A.size() || C[0].size() != A.size())
            C = vector<vector<int>>(A.size(), vector<int>(A.size(), 0));
        MatrixMultiplyRecursive(A, 0, 0, A.size()-1, A.size()-1, 
                                B, 0, 0, A.size()-1, A.size()-1,
                                C, 0, 0, A.size()-1, A.size()-1);
    }
    void SquareMatrixMulitplyBruteForce(vector<vector<int>>& A, vector<vector<int>>& B, vector<vector<int>>& C) {
        if (A.empty() || B.empty()) {
            std::cerr << " invalid arguments!" << std::endl;
            return;
        }

        if (A.size() != B.size() || A[0].size() != A.size() || B[0].size() != B.size()) {
            std::cerr << " invalid arguments " << std::endl;
            return;
        }
        if (C.empty() || C.size() != A.size() || C[0].size() != A.size())
            C = vector<vector<int>>(A.size(), vector<int>(A.size(), 0));
        for (int i = 0; i < C.size(); ++i) 
            for (int j = 0; j < C.size(); ++j)   
                for (int k = 0; k < C.size(); ++k) C[i][j] += A[i][k] * B[k][j];     
    }
};

int main()
{
    // Test for SquareMatrixMultiplyRecursive
    vector<vector<int>> A = {{1, 3, 1, 3, 1}, {7, 5, 1, 5, 7}, {1, 1, 1, 1, 1}, {7, 5, 1, 5, 7}, {1, 3, 1, 3, 1}};
    vector<vector<int>> B = {{6, 8, 1, 6, 8}, {4, 2, 1, 2, 4}, {1, 1, 1, 1, 1}, {4, 2, 1, 2, 4}, {6, 8, 1, 6, 8}};
    vector<vector<int>> C(A.size(), vector<int>(A.size(), 0));
    Solution sol;
    sol.SquareMatrixMultiplyRecursive(A, B, C);
    vector<vector<int>> D(A.size(), vector<int>(A.size(), 0));
    sol.SquareMatrixMulitplyBruteForce(A, B, D);
    for (int i = 0; i < A.size(); ++i) {
        for (int j = 0; j < A.size(); ++j) {
            if (C[i][j] != D[i][j]) std::cout << "SquareMatrixMultiplyRecursive is wrong!" << "\n";
        }
    }
}
```

分析其复杂度，给出简单分治SquareMatrixMultiplyRecursive的运行时间$T(n)$的递归式：
$$
T(n) = \left\{\begin{array}{lcl}
\Theta(1) & { n =1}\\
8*T(n/2) + \Theta(n^2) & {n>1}
\end{array} \right.
$$
可通过4.5节主定理计算得出$T(n)$为 $\Theta(n^3)$，简单分治算法在这里并不优于暴力求解法。

### Strassen算法

这里第四版的Strassen算法讲解就明显比第三版更加有趣，建议阅读第四版。

Strassen的算法核心思想：让之前的简单分治法矩阵乘法的递归树不要那么茂盛，我们会在每个递归中多添加常数次的子矩阵的加法运算（该算法的代价），从而让原来递归树的每个非叶节点有8个分支缩成7个分支， 从而达到缩短算法运行时间的效果， 其运行时间$T(n)$的递归式：
$$
T(n) = \left\{\begin{array}{lcl}
\Theta(1) & { n =1}\\
7*T(n/2) + \Theta(n^2) & {n>1}
\end{array} \right.
$$
可通过4.5节主定理计算得出$T(n)$为 $\Theta(n^{lg7})$，此算法明显优于暴力求解法， 真不愧是Strassen remarkable Algorithm呀。。。

Strassen算法整体流程：

1.预处理：方阵 $ n \times n$ 的A, B, C的n不为2的幂，则先把n扩成 $2^{\lceil lgn\rceil}$， 把方阵扩成$2^{\lceil lgn\rceil} \times 2^{\lceil lgn\rceil}$ ，多出来的index全部补零。

   1.1 计算 $2^{\lceil lgn\rceil}$的c++实现如下：

```c++
int RoundUpPower(uint32_t size) {
    if ((size & (size - 1)) == 0) return size;
    uint32_t ansVal = 0x80000000;
    while ((ansVal & size) == 0) {
        ansVal = (anVal >> 1);
    }

    return (ansVal << 1);
}
```

  1.2 把方阵扩成$2^{\lceil lgn\rceil} \times 2^{\lceil lgn\rceil}$ ，多出来的index全部补零, c++实现如下（vector的resize（）会自动补零）：

```c++
// Padding zeros for the matrix to be resized
void Resize(vector<vector<int>>& matrix, uint32_t minPow) {
    matrix.resize(minPow);
    for (int i = 0; i < matrix.size(); ++i) {
        matrix[i].resize(minPow);
    }
}
```

2 Strassen的核心算法流程 （此时经过预处理的矩阵 $ n \times n$ 的A, B, C中 的n已经 为 2的幂）：

   2.1 如果 n = 1, 即被切出来的A, B,C只有一个元素，那么 $C[i][j] = C[i][j] + A[i][j]\cdot B[i][j]$, c++实现如下：

```c++
// recursion basis:
//   1. There is single element for A, B, C
if (len == 1) {
	C[startCr][startCc] += A[startAr][startAc] * B[startBr][startBc];
	return;
}
```

​           否则将矩阵A, B, C切分成 $ n/2 \times n/2$ 的子矩阵：
$$
A=\begin{bmatrix} A_{11} & A_{12} \\A_{21} & A_{22} \end{bmatrix}, &
B=\begin{bmatrix} B_{11} & B_{12} \\B_{21} & B_{22} \end{bmatrix}, &
C=\begin{bmatrix} C_{11} & C_{12} \\C_{21} & C_{22} \end{bmatrix}
$$
​           这里矩阵分解推荐使用通过下标计算完成分解，复杂度为 $\Theta(1)$，不要真的创建12个新的 $ n/2 \times n/2$ 的子矩阵中间变量。

​    2.2 创建10个 $ n/2 \times n/2$ 的矩阵 $S_{1}, S_{2},...,S_{10}$ , 每个矩阵为2.1中创建的两个子矩阵的和或差，复杂度为 $\Theta(n^2)$ , (敲公式手麻了, 聪明的你一定能看懂)：

```pseudocode
    S[1] = B[1, 2] - B[2, 2]
    S[2] = A[1, 1] + A[1, 2]
    S[3] = A[2, 1] + A[2, 2]
    S[4] = B[2, 1] - B[1, 1]
    S[5] = A[1, 1] + A[2, 2]
    S[6] = B[1, 1] + B[2, 2]
    S[7] = A[1, 2] - A[2, 2]
    S[8] = B[2, 1] + B[2, 2]
    S[9] = A[1, 1] - A[2, 1]
    S[10] = B[1, 1] + B[1, 2]
```

​     2.3 用步骤2.1切分出来的子矩阵和步骤2.2创建的10个矩阵，递归地用Strassen算出7个矩阵积 $P_{1}, P_{2},...,P_{7}$ (敲公式手麻了, 聪明的你一定能看懂)：

```pseudocode
    P[1] = STRASSEN(A[1, 1], S[1])
    P[2] = STRASSEN(S[2], B[2, 2])
    P[3] = STRASSEN(S[3], B[1, 1])
    P[4] = STRASSEN(A[2, 2], S[4])
    P[5] = STRASSEN(S[5], S[6])
    P[6] = STRASSEN(S[7], S[8])
    P[7] = STRASSEN(S[9], S[10])
```

​      2.4 通过Pi的不同组合进行加减运算， 计算出$C_{11}, C_{12}, C_{21}, C_{22}$ , 复杂度为 $\Theta(n^2)$  :

```c++
    C[1,1] = P[5] + P[4] - P[2] + P[6]
    C[1,2] = P[1] + P[2]
    C[2,1] = P[3] + P[4]
    C[2,2] = P[5] + P[1] - P[3] - P[7]
```



一年前只是在youtube上无意中看到过Strassen算法的简要介绍https://www.youtube.com/watch?v=sZxjuT1kUd0，当时真没想到一年后自己竟然动手成功用C++实现了一遍，成就感拉满呀！

规模为任意正整数的方阵Strassen算法的整体c++实现如下(差不多花了4个小时才debug成功，lego爱好者的快乐 -> 搬砖人的泪水)：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
// author: Claude Du
using std::vector;
class Solution
{
private:
    int RoundUpPower(uint32_t size) {
        if ((size & (size - 1)) == 0) return size;
        uint32_t ansVal = 0x80000000;
        while ((ansVal & size) == 0) {
            ansVal = (anVal >> 1);
        }
        
        return (ansVal << 1);
    }
    // Padding zeros for the matrix to be resized
    void Resize(vector<vector<int>>& matrix, uint32_t minPow) {
        matrix.resize(minPow);
        for (int i = 0; i < matrix.size(); ++i) {
            matrix[i].resize(minPow);
        }
    }
    // add == true, C = A + B
    // add == false, C = A - B
    void MatrixAddition(vector<vector<int>>& A, int startAr, int startAc, int len,  
                        vector<vector<int>>& B, int startBr, int startBc, 
                        vector<vector<int>>& C, int startCr, int startCc, bool add) {
        for (int i = 0; i < len; ++i) {
            for (int j = 0; j < len; ++j) {
                if (add) C[startCr + i][startCc + j] = A[startAr + i][startAc + j] + B[startBr + i][startBc + j];
                else C[startCr + i][startCc + j] = A[startAr + i][startAc + j] - B[startBr + i][startBc + j];
            }
        }

    }
    void MatrixMultiplyStrassen(vector<vector<int>>& A, int startAr, int startAc, int len,  
                                 vector<vector<int>>& B, int startBr, int startBc, 
                                 vector<vector<int>>& C, int startCr, int startCc)
    {
        // recursion basis:
        //   1. There is single element for A, B, C
        if (len == 1) {
            C[startCr][startCc] = A[startAr][startAc] * B[startBr][startBc];
            return;
        }
        // Divide: 
        int newLen = len/2;
        // S1, S2, S3, S4, S5, S6, S7, S8, S9, S10
        vector<vector<vector<int>>> S(10, vector<vector<int>>(newLen, vector<int>(newLen, 0)));
        // S1 = B12 - B22
        MatrixAddition(B, startBr, startBc + newLen, newLen,   // B12
                       B, startBr + newLen, startBc + newLen,  // B22
                       S[0], 0, 0, false);                     // S1
        // S2 = A11 + A12
        MatrixAddition(A, startAr, startAc, newLen,            // A11
                       A, startAr, startAc + newLen,           // A12
                       S[1], 0, 0, true);                      // S2
        // S3 = A21 + A22
        MatrixAddition(A, startAr + newLen, startAc, newLen,   // A21
                       A, startAr + newLen, startAc + newLen,  // A22
                       S[2], 0, 0, true);                      // S3
        // S4 = B21 - B11
        MatrixAddition(B, startBr + newLen, startBc, newLen,   // B21
                       B, startBr, startBc,                    // B11
                       S[3], 0, 0, false);                     // S4
        // S5 = A11 + A22
        MatrixAddition(A, startAr, startAc, newLen,            // A11
                       A, startAr + newLen, startAc + newLen,  // A22
                       S[4], 0, 0, true);                      // S5 
        // S6 = B11 + B22
        MatrixAddition(B, startBr, startBc, newLen,            // B11
                       B, startBr + newLen, startBc + newLen,  // B22
                       S[5], 0, 0, true);                      // S6                      
        // S7 = A12 - A22
        MatrixAddition(A, startAr, startAc + newLen, newLen,   // A12
                       A, startAr + newLen, startAc + newLen,  // A22
                       S[6], 0, 0, false);                     // S7
        // S8 = B21 + B22
        MatrixAddition(B, startBr + newLen, startBc, newLen,   // B21
                       B, startBr + newLen, startBc + newLen,  // B22
                       S[7], 0, 0, true);                      // S7
        // S9 = A11 - A21
        MatrixAddition(A, startAr, startAc, newLen,            // A11
                       A, startAr + newLen, startAc,           // A21
                       S[8], 0, 0, false);                     // S9
        // S10 = B11 + B12
        MatrixAddition(B, startBr, startBc, newLen,            // B11
                       B, startBr, startBc + newLen,           // B12
                       S[9], 0, 0, true);                      // S10
        // P1, P2, P3, P4, P5, P6, P7
        vector<vector<vector<int>>> P(7, vector<vector<int>>(newLen, vector<int>(newLen, 0))); 
        // P1 = A11*S1
        MatrixMultiplyStrassen(A, startAr, startAc, newLen,
                               S[0], 0, 0,
                               P[0], 0, 0);
        // P2 = S2*B22
        MatrixMultiplyStrassen(S[1], 0, 0, newLen,
                               B, startBr + newLen, startBc + newLen, 
                               P[1], 0, 0);
        // P3 = S3*B11
        MatrixMultiplyStrassen(S[2], 0, 0, newLen,
                               B, startBr, startBc, 
                               P[2], 0, 0);
        // P4 = A22*S4
        MatrixMultiplyStrassen(A, startAr + newLen, startAc + newLen, newLen,
                               S[3], 0, 0,
                               P[3], 0, 0);
        // P5 = S5*S6
        MatrixMultiplyStrassen(S[4], 0, 0, newLen,
                               S[5], 0, 0,
                               P[4], 0, 0);
        // P6 = S7*S8
        MatrixMultiplyStrassen(S[6], 0, 0, newLen,
                               S[7], 0, 0,
                               P[5], 0, 0);
        // P7 = S9*S10
        MatrixMultiplyStrassen(S[8], 0, 0, newLen,
                               S[9], 0, 0,
                               P[6], 0, 0);
        // C11 = P5 + P4 - P2 + P6
        MatrixAddition(P[4], 0, 0, newLen,
                       P[3], 0, 0,
                       C, startCr, startCc, true);
        MatrixAddition(C, startCr, startCc, newLen,
                       P[1], 0, 0,
                       C, startCr, startCc, false);
        MatrixAddition(C, startCr, startCc, newLen,
                       P[5], 0, 0,
                       C, startCr, startCc, true);
        // C12 = P1 + P2
        MatrixAddition(P[0], 0, 0, newLen,
                       P[1], 0, 0,
                       C, startCr, startCc + newLen, true);
        // C21 = P3 + P4
        MatrixAddition(P[2], 0, 0, newLen,
                       P[3], 0, 0,
                       C, startCr + newLen, startCc, true);
        // C22 = P5 + P1 - P3 - P7
        MatrixAddition(P[4], 0, 0, newLen,
                       P[0], 0, 0,
                       C, startCr + newLen, startCc + newLen, true);
        MatrixAddition(C, startCr + newLen, startCc + newLen, newLen,
                       P[2], 0, 0,
                       C, startCr + newLen, startCc + newLen, false);
        MatrixAddition(C, startCr + newLen, startCc + newLen, newLen,
                       P[6], 0, 0,
                       C, startCr + newLen, startCc + newLen, false);
    }

public:

    void SquareMatrixMultiplyStrassen(vector<vector<int>>& A, vector<vector<int>>& B, vector<vector<int>>& C) 
    {
        // avoid invalid input
        if (A.empty() || B.empty()) {
            std::cerr << " invalid arguments!" << std::endl;
            return;
        }

        if (A.size() != B.size() || A[0].size() != A.size() || B[0].size() != B.size()) {
            std::cerr << " invalid arguments " << std::endl;
            return;
        }
        if (C.empty() || C.size() != A.size() || C[0].size() != A.size())
            C = vector<vector<int>>(A.size(), vector<int>(A.size(), 0));
        
        uint32_t size = A.size();
        uint32_t minPow = RoundUpPower(size);
        Resize(A, minPow);
        Resize(B, minPow);
        Resize(C, minPow);
        MatrixMultiplyStrassen(A, 0, 0, minPow,
                               B, 0, 0,
                               C, 0, 0);
        Resize(A, size);
        Resize(B, size);
        Resize(C, size);
    }
    // BruteForce is used for testing Strassen
    void SquareMatrixMulitplyBruteForce(vector<vector<int>>& A, vector<vector<int>>& B, vector<vector<int>>& C) {
        if (A.empty() || B.empty()) {
            std::cerr << " invalid arguments!" << std::endl;
            return;
        }

        if (A.size() != B.size() || A[0].size() != A.size() || B[0].size() != B.size()) {
            std::cerr << " invalid arguments " << std::endl;
            return;
        }
        if (C.empty() || C.size() != A.size() || C[0].size() != A.size())
            C = vector<vector<int>>(A.size(), vector<int>(A.size(), 0));
        for (int i = 0; i < C.size(); ++i) 
            for (int j = 0; j < C.size(); ++j)   
                for (int k = 0; k < C.size(); ++k) C[i][j] += A[i][k] * B[k][j];     
    }
};


int main()
{
    // Test for SquareMatrixMultiplyRecursive
    vector<vector<int>> A = {{1, 3, 1, 3, 1}, {7, 5, 1, 5, 7}, {1, 1, 1, 1, 1}, {7, 5, 1, 5, 7}, {1, 3, 1, 3, 1}};
    vector<vector<int>> B = {{6, 8, 1, 8, 6}, {4, 2, 1, 2, 4}, {1, 1, 1, 1, 1}, {4, 2, 1, 2, 4}, {6, 8, 1, 8, 6}};
    vector<vector<int>> C(A.size(), vector<int>(A.size(), 0));
    
    Solution sol;
    sol.SquareMatrixMultiplyStrassen(A, B, C);

    
    vector<vector<int>> D(A.size(), vector<int>(A.size(), 0));
    sol.SquareMatrixMulitplyBruteForce(A, B, D);
    for (int i = 0; i < A.size(); ++i) {
        for (int j = 0; j < A.size(); ++j) {
            std::cout << C[i][j] << " ";
        }
        std::cout << "\n";
    }
    std::cout << "\n";
    for (int i = 0; i < A.size(); ++i) {
        for (int j = 0; j < A.size(); ++j) {
            std::cout << D[i][j] << " ";
        }
        std::cout << "\n";
    }
}
```





