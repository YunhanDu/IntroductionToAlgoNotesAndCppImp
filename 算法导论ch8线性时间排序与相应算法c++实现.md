# 算法导论Ch8线性时间排序笔记与相应算法c++实现

作者： Claude Du

归并排序与堆排最差情形的运行时间上界为$O(nlgn)$ ，快排的平均运行时间上界为$O(nlgn)$。

并且以上三种算法的共同性质：

1. 运行时间上界也都为$\Omega(nlgn)$。
2. 排序顺序由输入元素之间的比较来决定。

我们把这种由性质2决定的算法称为比较排序，当前所算导中所引入的所有排序算法都是比较排序（归并排序，堆排，快排，插入排序，冒泡排序）。比较排序的最差情形运行时间为 $\Omega(nlgn)$ , 8.1节将证明该结论。

8.2节、8.3节和8.4节将讨论三种非比较排序且线性时间复杂度的排序算法：计数排序、基数排序和桶排序。

## 8.1 比较排序的下界

为了讨论比较算法的下界，我们不失一般性的假设**输入序列 $\left\{ a_{1}, a_{2}, ...,a_{n} \right\}$ 的所有元素都是互异的** 。在该假设前提下， 比较操作 $a_{i} \leq a_{j}$ ，   $a_{i} \geq a_{j}$  ， $a_{i} \lt a_{j}$  和  $a_{i} \gt a_{j}$ 都是等价的，因为通过以上4种比较操作得到 的$a_{i}$ 和 $a_{j}$ 相对次序信息是相同的。因此我们进一步假设所有的比较操作都采用   $a_{i} \leq a_{j}$ 的形式。

### 决策树模型（The decision-tree model）

比较排序可以被抽象为一棵决策树。决策树是一棵完全二叉树（每个节点要么是叶子，要么有两个孩子），它可以表示给定输入规模情况下，某一特定排序算法对所有元素的比较操作。其中，控制，数据移动等其他操作都被忽略了。下图【第四版图8.1】呈现了2.1节插入排序作用于3元素的输入序列的决策时情况：

![](./figure8_1.PNG)



上图8.1中每个内部节点都以 $i:j$ 标记，其中 $1 \leq i, j \leq n$ ， $n$ 是输入序列的元素个数，节点 $i:j$ 表示 $a_{i}$ 和  $a_{j}$ 之间的比较操作， "if ( $a_{i} \leq a_{j}$ )"。 

每个叶节点上都标注一个序列 $\left\langle \pi(1), \pi(2),...,\pi(n)\right\rangle$ (序列的背景知识参阅C.1节)。比较排序算法的执行对应于一条从树的根节点到叶节点的路径。

节点 $i:j$ 的左子树表示我们确定  $a_{i} \leq a_{j}$ 之后的后续比较。

节点 $i:j$ 的右子树表示我们确定  $a_{i} \gt a_{j}$ 之后的后续比较。

当到达一个叶节点时，表示排序算法已经确定了一个顺序 $a_{ \pi(1)} \leq a_{ \pi(2)} \leq ...\leq a_{ \pi(n)}$ 。

对一个正确的比较排序算法来说，n个输入元素的 $n!$ 种可能排序结果都得出现在决策树的叶节点上，并且该叶节点必须是可以从根节点经由某条路径达到的（我们称该叶节点为可达的）。

### 最坏情况下界

在决策树中，从根节点到叶节点的路径之间最长简单路劲的长度，即该树的高度，表示的就是对应的比较排序算法的比较次数。那么，该决策树的高度下界也就是该比较排序算法运行时间的下界。下面的定理给出这样的下界。

#### 定理8.1

在最坏情况下，任何比较排序算法都需要做 $\Omega(nlgn)$ 次比较。

#### 证明  

考虑一棵高度为 $h$ ， 具有 $l$ 个可达叶节点的决策树，它对应一个对 $n$ 个元素所做的比较排序。因为输入数据的 $n!$ 种可能的排列都是可达叶节点，所以有 $n! \leq l$ 。由于在一棵高度为 $h$ 的树的叶节点数不多于 $2^h$ ， 我们得到：
$$
n! \leq l \leq 2^h
$$
对该式两边取对数， 有:
$$
\begin{aligned}
h &\geq \lg(n!)
      = \Omega(nlgn) \\
\end{aligned}
$$
 得证。

## 8.2 计数排序

计数排序假设输入规模为n的数组中的每一个元素都为小于等于k的非负整数，它的运行时间为 $\Theta(n+k)$ ，因此当 $k= O(n)$ ，计数排序的运行时间为 $\Theta(n)$。

基本思想：对每个输入元素x，确定小于x的元素个数。利用该信息，直接把x放到它在输出数组中的位置即可。例如，如果有17个元素小于x, 就把x放在第18个输出位置上即可。当有几个元素相同时，要将次方案略微修改，因为不能把他们放在同一个位置上。

其c++实现如下：

```c++
// author: Claude Du
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

总运行时长为$\Theta(n)$ ，优于比较排序最差下界 $\Omega(nlgn)$ 。计数排序另一大优点是它是稳定的。我们接下来简要证明下其稳定性：

计数排序是稳定的另一种数学表述：如果$arr[n]= arr[m]$ 且 $n \lt m$ ，经过计数排序后，$arr[n]$ 的新位置 $n^{'}$ 和$arr[m]$ 的新位置 $m^{'}$ 一定有 $n^{'}\lt m^{'}$ 。

证明该表述正确：首先在19行for循环中，从左往右进行同元素值计数统计时，刚刚扫描完数组小标$n$ 时的$count_n = cArr[arr[n]]$ 一定小于刚刚扫描完数组小标$m$ 时的$count_m = cArr[arr[m]]$。  

再在经过 第22行的for循环后，这一关系依然得到保持 , 两个计数$count_n, count_m$ 都增加了相同的数值 $cArr[arr[m]-1]$。

最后在第27行的for循环中，我们从右往左扫描$arr$，当$ cArr[arr[m]] = count_m$ 时，$arr[count_m]$ 被赋值 $arr[m]$, 即 $m' = count_m$ 。同理， $n' = count_n$ ， 前面已知 $count_n \lt count_m$ ，于是 $n^{'}\lt m^{'}$ ，稳定性得证 。

计数排序的稳定性之所以重要的另一个原因是：计数排序常常会被用作基数排序（Radix Sort）的子程序(subroutine)。

## 8.3 基数排序（Radix Sort）

讲真，书中讲解卡片排序机的例子我没看懂（澳门赌场在线发牌的核心技术？），还是看看d位10进制数字情形下的基数排序吧。

 7个3位10进制数的基数排序过程如下图所示【第四版图8.3】：

![](./figure8_3.PNG)

基数排序是先按照最低有效位（the least significant  digit）来进行一轮稳定排序，再按照第2低有效位再进行一轮稳定排序，直到对所有的d位数字都完成了排序，此时便已排序完成。

伪代码实现如下：

```pseudocode
RADIX-SORT.A; n; d /
1 for i = 1 to d
2 	use a stable sort to sort array A[1:n] on digit i 
```

在基数排序中使用会另一个稳定的排序子程序，通常计数排序会被选为该子程序。

10进制情形下的Radix排序c++实现如下，其中稳定排序子程序使用了计数排序：

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
        // cArr[i] now contains the number of elements less than or equal to i.
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

基数排序的正确性证明会放在后续的习题答案中。

### 引理8.3

给定n个d位数，其中每一个数位有k个可能的取值。如果基数排序使用的稳定排序耗时$\Theta(n + k)$ ，那么它就可以在 $\Theta(d(n + k))$ 时间内将这些数排好序。

具体证明内容请见书。

### 引理8.4

给定n个b位数和任何正整数 $r \leq b$ , 如果 RadixSort 使用的稳定排序算法对数据取值间是 0 到 k 的输入进行排序耗时 $\Theta(n+k)$ ，那么它就可以在 $\Theta((b/r)(n + 2^r))$ 时间内将这些数排好序

具体证明内容请见书。

基数排序是否比其他优秀的比较排序算法（如快排）更好呢？

得从两个维度去分析这个问题；

1.时间复杂度：如果 $b = lgn$ , 我们选择 $r \approx lgn$ ，则基数排序运行时间为 $\Theta(n)$ ， 该结果看起来比快排的期望运行时间  $\Theta(nlgn)$ 快。但是两个运行时间表达式中 $\Theta$ 符号背后的常数项因子是不同的。在处理n个关键字时，基排执行的循环轮数会比快排少，但基排每一轮耗费时间比快排长很多。

2.空间复杂度：哪个排序更合适依赖于具体实现和底层硬件的特性（通常快排可以比基排更高效的使用硬件缓存），以及输入数据的特征。此外，使用计排作为中间子程序的基排不是原址排序。当主存容量宝贵时，我们更倾向选择快排这样的原址排序。

## 8.4 桶排序

桶排序的假设前提：输入数据均匀分布，即**输入是由一个随机过程产生的，该过程将元素均匀、独立地分布在[0, 1)区间上。**

桶排序的基本思想：将[0, 1)区间划分成n个大小相同的子区间$[0, 1/n),[1/n, 2/n),...,[(n-1)/n,1)$，其中每个子区间称为**桶**。然后将n个输入数分别放在自己所属的桶中(数学语言表示，将每一个输入元素 $arr[i]$ ，其中 $1 \leq i \leq n $ ，放入 第 $\lfloor n\cdot arr[i]\rfloor$ 个子区间中，即 $[(\lfloor n\cdot arr[i]\rfloor)/n, (\lfloor n\cdot arr[i]\rfloor + 1)/n)$ 中）。再对每个桶中的数进行排序，最后按照次序把各个桶中的元素列出来即可。

下图【第四版图8.3】显示了一个包含10个元素的输入数组的桶排序过程：

![](./figure8_4.PNG)



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

桶排序的正确性验证桶排序的期望运行时间为$\Theta(n)$可直接看书。

本篇笔记终于结束啦！

原书第3版期望运行时间的证明非常精彩，放在附录里了。

## 附录

### 桶排序的期望运行时间为$\Theta(n)$的证明

桶排序c++代码中除了第33行，所有其他各行的总时间代价都为$O(n)$ 。

分析调用插入排序的时间代价，假设 $n_{i}$ 是表示桶 bArr[i] 中元素个数的随机变量，则桶 bArr[i] 的插入排序时间代价为 $O(n_{i}^2)$ , 则桶排序的时间代价 $T(n)$ 为：
$$
T(n) = \Theta(n) + \sum _{i= 0}^{n-1} O(n_{i}^2)
$$
对上式两边取期望，并利用期望的线性性质，我们有：
$$
\begin{aligned}
E[T(n) ]&=E\left[\Theta(n) + \sum _{i= 0}^{n-1} O(n_{i}^2)\right] \\
     &=\Theta(n) + \sum _{i= 0}^{n-1} E\left[O(n_{i}^2)\right] \\
     &= \Theta(n) + \sum _{i= 0}^{n-1} O\left(E\left[n_{i}^2\right]\right)
\end{aligned}
$$
我们断言：
$$
E\left[n_{i}^2\right] = 2 - 1/n
$$
对所有 $i = 0, 1,..., n-1$ 都成立。这点不足为奇：因为输入数组 arr的每一个元素是等概率地落入任意一个桶中，所以每一个桶 $i$ 具有相同期望值 $E\left[n_{i}^2\right]$ 。为了证明该断言。我们定义指示器随机变量：对所有 $i = 0, 1,..., n-1$ 和 $j = 1, 2,..., n$ ，
$$
X_{ij} = I \left\{arr[j]落入桶i\right\}
$$
因此：
$$
n_{i} = \sum_{j=1}^{n}X_{ij}
$$
为了计算 $E\left[n_{i}^2\right]$ ，我们展开平方项，并重新组合各项：
$$
\begin{aligned} 
E\left[n_{i}^2\right]&=E\left[\left(\sum_{j=1}^{n}X_{ij}\right)^2\right] = E\left[\sum_{j=1}^{n}\sum_{k=1}^{n}X_{ij}X_{ik}\right]  = E\left[\sum_{j=1}^{n}X_{ij}^2+\sum_{1\leq j \leq n}\space\sum_{1\leq k \leq n, \space k \neq j}X_{ij}X_{ik}\right]\\
     &=\sum_{j=1}^{n}E\left[X_{ij}^2\right]+\sum_{1\leq j \leq n}\space\sum_{1\leq k \leq n, \space k \neq j}
     E\left[X_{ij}X_{ik}\right]  \\
   
\end{aligned}
$$
上式的最后一行由期望的线性性质得出的。我们分别计算最后一行的两项累加和，指示器随机变量 $X_{ij}$ 为1的概率为 $1/n$ ，其他情况下的概率为0.于是有：
$$
E\left[X_{ij}^2\right] = 1^2 \cdot \frac{1}{n} + 0^2 \cdot \left(1- \frac{1}{n}\right) =  \frac{1}{n}
$$
当 $k \ne j$ 时，随机变量 $X_{ij}$ 和 $X_{ik}$ 是互相独立的，因此有：
$$
E\left[X_{ij}X_{ik}\right] = E\left[X_{ij}\right]E\left[X_{ik}\right] =  \frac{1}{n} \cdot  \frac{1}{n} =  \frac{1}{n^2}
$$
将两个 $E\left[n_{i}^2\right]$ 表达式中最后一行的两项累加和进行替代，得到：
$$
\begin{aligned} 
E\left[n_{i}^2\right]
     &=\sum_{j=1}^{n} \frac{1}{n}+\sum_{1\leq j \leq n}\space\sum_{1\leq k \leq n, \space k \neq j}\frac{1}{n^2} 
     = n \cdot \frac{1}{n} + n(n-1)\cdot \frac{1}{n^2}  = 2-\frac{1}{n}\\
   
\end{aligned}
$$
断言得证。

利用该断言，求出$E[T(n) ]$：
$$
\begin{aligned}
E[T(n) ] &= \Theta(n) + \sum _{i= 0}^{n-1} O\left(E\left[n_{i}^2\right]\right)=  \Theta(n) + n\cdot O(2-1/n)=\Theta(n)
\end{aligned}
$$
我们可以得出结论，桶排序的期望运行时间为$\Theta(n)$ 。

