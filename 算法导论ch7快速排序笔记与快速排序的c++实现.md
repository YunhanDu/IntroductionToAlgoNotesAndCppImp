# 算法导论ch7快速排序笔记与相应算法c++实现

作者：Claude Du

快速排序的优点：

1. 平均性能非常好，期望时间复杂度是$\Theta(nlgn)$  且常数因子非常小。
2. 能够原址排序，甚至在虚拟环境中也能很好地工作。 

快速排序的缺点：

1. 最坏情况下时间复杂度为$\Theta(n^2)$ 。

快排通常是实际排序应用中最好的选择。

## 7.1快速排序的描述

与并归排序类似，快排也使用分治思想。

对一个子数组进行一下快排的3步分治过程：

分解：数组$A[left:right]$ 被分解为两个子数组 $A[left:q-1]$ 和 $A[q + 1 : right]$ , 使得 $A[left:q-1]$中的每个元素都小于等于$A[q]$ , 而 $A[q]$ 也小于等于 $A[q+1:right]$ 的每个元素

解决：通过递归调用快速排序，对子数组 $A[left:q-1]$ 和 $A[q + 1 : right]$ 进行排序。

合并：因为子数组都是原址排序的，所以不需要合并操作。

快速排序基本流程c++实现如下:

```c++
void QuickSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int pivotIndex = Partition(arr, left, right);
        QuickSort(arr, left, pivotIndex -1);
        QuickSort(arr, pivotIndex + 1, right);
    }
}
void QuickSort(vector<int>& arr) {
    return QuickSort(arr, 0, arr.size() - 1);
}
```

### 数组的划分

Partition是快排的关键，它实现了对子数组 $arr[left:right]$ 的原址重排，其c++实现如下：

```c++
int Partition(vector<int>& arr, int left, int right) {
    int x = arr[right]; // the pivot
    int i = left - 1; // highest index into the low side
    // process each element other than the pivot
    for (int j = left; j < right; ++j) { 
        if (arr[j] <= x) { // does jth element belong on the low side
            ++i;
            // exchange A[i] with A[j]
            int temp = arr[j];
            arr[j] = arr[i];
            arr[i] = temp;
        }
    }
    // exchange A[i+1] with A[right]
    ++i;
    arr[right] = arr[i];
    arr[i] = x;
    return i;
}
```

Partition 总会选择一个 $x = arr[right]$ 作为主元(pivot element)，并使用它来划分子数组$arr[left:right]$ ，确保划分出来的左子数组 $A[left:q-1]$中的每个元素都小于等于 $x$ , 剩下的元素都是置换到右子数组中。

**简单验证Partition算法的正确性**：

对于Partition算法中的for循环部分，我们将以下这些性质作为循环不变量：对于for循环的每次迭代开始前，对于任何数组下标k, 以下性质都成立：
1. 如果 $left\leq k \leq i$ , 则 $A[k] \leq x$ ；
2. 如果 $i+1\leq k \leq j-1$ , 则 $A[k] \gt x$ ;
3. 如果 $ k = right$ , 则 $A[k] = x$ ;

该循环不变量在第一次迭代前是成立的，在每一轮迭代后仍然都成立，在循环结束时，该循环不变量还可以为证明算法正确性提供有用的信息。

**初始化（Initialization）**：在第一次迭代前，$i < left$  以及 $j = left$, 性质1和2中的区间为空，不存在符合条件的k, 则性质1和性质2都满足，性质3也显然成立。

**保持（Maintenance）** : 假设上一轮迭代后，循环不变量成立， 本轮迭代如图7.3所示【1】，根据Partition C++实现的第7行“if (arr[j] <= x) ”的不同结果，我们要考虑两种情况：

![](./7_3%E6%88%AA%E5%9B%BE.PNG)

情况一如图7.3（a）：当$arr[j] > x$ 时，我们只用 ++j。当本次迭代结束后（++j后), 性质2依然满足，性质1， 3则维持不变（依旧成立），则本轮迭代后，依然循环不变量都成立。

情况二如图7.3 (b) :  当$arr[j] \leq x$ , 此时 ++i后， 有 $arr[i] > x$ ，交换 $arr[i]$ 和  $arr[j]$ 和++j后(本轮迭代结束后)，我们有  $arr[i] \leq x$ 和 $arr[j-1] \gt x$ , 再基于我们已假设上轮迭代后循环不变量不变，则本轮的三个循环不变量的性质依然成立。

**终止 （Termination）**: 当for循环终止时， $j = r$ 。数组中的每个元素都在循环不变量描述的三个集合情形中。在PartitionC++代码中最后4行代码，将主元与最左侧的大于x的元素进行交换，主元移到了相应正确的位置，并返回其新下标。Partition的满足其划分数组的目的，其实比原来的目标还要更严格：$A[q]$ 严格小于 $A[q+1:right]$ 的每个元素。

快排的c++实现与简单用例验证：

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using std::vector;
class Solution {
private:
    int Partition(vector<int>& arr, int left, int right) {
        int x = arr[right]; // the pivot
        int i = left - 1; // highest index into the low side
        // process each element other than the pivot
        for (int j = left; j < right; ++j) { 

            if (arr[j] <= x) { // does thw element belong on the low side
                ++i;
                // exchange A[i] with A[j]
                int temp = arr[j];
                arr[j] = arr[i];
                arr[i] = temp;
            }
        }
        // exchange A[i+1] with A[right]
        ++i;
        arr[right] = arr[i];
        arr[i] = x;
        return i;
    }
public:
    void QuickSort(vector<int>& arr, int left, int right) {
        if (left < right) {
            int pivotIndex = Partition(arr, left, right);
            QuickSort(arr, left, pivotIndex -1);
            QuickSort(arr, pivotIndex + 1, right);
        }
    }
    void QuickSort(vector<int>& arr) {
        return QuickSort(arr, 0, arr.size() - 1);
    }
};

int main()
{
    vector<int> arr = {13, 19, 5, 12, 8, 7, 4, 21, 2, 6, 11};
    Solution sol;
    sol.QuickSort(arr);
    for (int i = 0; i < arr.size(); ++i) {
        std::cout << arr[i] << " ";
    }
}
```



## 7.2 快速排序的性能

快排的运行时间依赖于划分是否左右平衡。而左右平衡与否又依赖于用于作为主元（pivot）的元素。

如果划分一直左右平衡，那么快排时间复杂度 $T(n)$ （最好情形）和并排的时间复杂度渐进相同：
$$
T(n) = \left\{\begin{array}{lcl}
c & { n =1}\\
2T(n/2) + \Theta(n) & {n>1}
\end{array} \right.
$$
可计算得出$T(n)$为 $\Theta(nlgn)$ 。

如果划分一直不平衡，则快排的时间复杂度 $T(n)$ （最差情形）就接近插入排序，为 $\Theta(n^2)$ 。

快排的平均运行时间更接近于最好情形，详情见 7.4章节快速排序分析。



## 7.3 快速排序的随机化版本

有时我们可以通过在算法中引入随机性，从而使得算法对于所有的输入都能获得较好的期望性能。很多人都选择随机化版本的快排作为大数据输入情况下的排序算法。

我们通过采用随机抽样（random sampling）技术, 可让分析变得更简单。和之前始终采用$arr[right]$ 作为主元的方法不同，随机抽样是从 $arr[left:right]$ 随机抽取一个元素与 $arr[right]$ 交换，交换过后，把现在新的 $arr[right]$ 作为主元。通过随机抽样，我们可以保证主元素 $x = arr[right]$ 是等概率地从子数组 $r-p +1$ 个元素中选取的，同时我们期望在平均情况下，对输入数组的划分是比较均衡的。

在新的划分程序（RandomizedPartition）中，我们要在真正划分前，随机抽样一个元素，并让该元素与 $arr[right]$ 交换，其c++实现如下：

```c++
int RandomizedPartition(vector<int>& arr, int left, int right) {
    // i = RANDOM(left, right)
    int randInd = left + (rand() % (right - left + 1));
    // exchange arr[right] eith arr[randInd]
    std::swap(arr[randInd], arr[right]);
    return Partition(arr, left, right);
}
```

随机化版的快排要调用的划分程序便是上面的RandomizedPartition，随机化版的快排主程序c++实现如下，（留意要使用c++的srand）：

```c++
void RandomizedQuickSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;
    int pivotIndex = RandomizedPartition(arr, left, right);
    RandomizedQuickSort(arr, left, pivotIndex - 1);
    RandomizedQuickSort(arr, pivotIndex + 1, right);
}
void RandomizedQuickSort(vector<int>& arr) {
    srand(time(NULL));
    RandomizedQuickSort(arr,  0, arr.size() - 1);
}
```

完整版快排与单个用例验证的整体代码如下：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <stdlib.h>
#include <time.h>

using std::vector;
class Solution {
private:
    int Partition(vector<int>& arr, int left, int right) {
        int x = arr[right]; // the pivot
        int i = left - 1; // highest index into the low side
        // process each element other than the pivot
        for (int j = left; j < right; ++j) { 
            if (arr[j] <= x) { // does jth element belong on the low side
                ++i;
                // exchange A[i] with A[j]
                int temp = arr[j];
                arr[j] = arr[i];
                arr[i] = temp;
            }
        }
        // exchange A[i+1] with A[right]
        ++i;
        arr[right] = arr[i];
        arr[i] = x;
        return i;
    }
    int RandomizedPartition(vector<int>& arr, int left, int right) {
        // i = RANDOM(left, right)
        int randInd = left + (rand() % (right - left + 1));
        // exchange arr[right] eith arr[randInd]
        std::swap(arr[randInd], arr[right]);
        return Partition(arr, left, right);
    }
public:
    void RandomizedQuickSort(vector<int>& arr, int left, int right) {
        if (left >= right) return;
        int pivotIndex = RandomizedPartition(arr, left, right);
        RandomizedQuickSort(arr, left, pivotIndex - 1);
        RandomizedQuickSort(arr, pivotIndex + 1, right);
    }
    void RandomizedQuickSort(vector<int>& arr) {
        srand(time(NULL));
        RandomizedQuickSort(arr,  0, arr.size() - 1);
    }

    
};

int main()
{
    vector<int> arr = {13, 19, 5, 12, 8, 7, 4, 21, 2, 6, 11};
    Solution sol;
    sol.RandomizedQuickSort(arr);
    for (int i = 0; i < arr.size(); ++i) {
        std::cout << arr[i] << " ";
    }
}
```

## 7.4 快速排序分析

我们需要给出快排性能更严谨的分析。先从最坏情况开始。

### 7.4.1 最坏情况分析

本篇章相对严谨地使用4.3节代入法证明了7.2节的最坏情况的时间复杂度就是 $\Theta(n^2)$ ，具体可以看算导书中7.4.1节。

### 7.4.2 期望运行时间（来自算导第四版英文版）

首先，在分析期望运行时间时，**有个假设前提：所有待排序的元素是始终互异的。**

先直观上理解为何RandomizedQuickSort的期望运行时间是$O(nlgn)$ : 如果在递归的每一层上，RandomizedPartition 将任意常数比例的元素划分到一个子数组中，则算法的递归树深度为  $\Theta(lgn)$ , 并且每一层上的工作量都为 $O(n)$ 。即使在最不平衡的划分情况下，会增加一些新层次， 但总运行时间依然是$O(nlgn)$ 。

**运行时间和比较操作**

QuickSort 和 RandomizedQuickSort 除了如何选择主元素有差异外，其他方面完全相同。因此我们可以先分析QuickSort 和 Partition 步骤，以此作为基础，再分析RandomizedQuickSort。

**引理 7.1**

在n个元素的数组上进行QuickSort的运行时间是  $O(n + X)$ ， 其中 $X$ 表示 Partition中 第5行的元素比较（“if (arr[j] <= x)”）的运行次数。

**引理7.1的证明**：QuickSort的运行时间主要由Partition操作上所花费的时间决定的。每一次Partition的调用都会选择一个主元，而且该主元不会出现在后续的QuickSort和Partition的调用中，因此，在快排整个执行期间，Partition最多能被调用n次。每次QuickSort调用Partition， 也会递归调用2次自己， 因此QuickSort自身最多也只能被调用2n次。

调用一次Partition的时间为$O(1)$ 和内部for循环所消耗的时间。很显然，内部for循环所消耗时间和（“if (arr[j] <= x)”）的运行次数成正比，这意味着在整个快排整个执行期间，所有调用Partition的内部for循环消耗时间之和 和 $X$成正比。

到这里，因为Partition之多调用n次，在partition中所有除了内部for循环外的总耗时是$O(n)$ （因为调用一次Partition除去内部for循环外的耗时为$O(1)$），因此quickSort的总耗时为  $O(n + X)$ ， 引理 7.1 得证。

为了分析 RandomizedQuickSort ，我们要计算随机变量$X$ 的期望值 $E[X]$ ， 这里$X$ 代表RandomizedQuickSort整个执行过程中所有调用Partition中元素比较的总次数 。  为此，我们必须了解何时元素比较会发生，何时不会发生。为了便于分析，我们将数组$A$ 的各个元素重新命名成 $z_{1}, z_{2},...,z_{n} $ , 其中  $z_{1}\lt z_{2}\lt ...\lt z_{n}$ ， 所有元素严格不等，因为**所有待排序的元素是始终互异的** 。 我们还定义 $Z_{ij} = \left\{ z_{i}, z_{i+1},...,z_{j}\right\}$ 。

下一个引理描述两个元素何时会互相比较。

**引理7.2**

在一个有n个元素， $z_{1}\lt z_{2} \lt . . . \lt z_{n} $ , 的数组上执行RandomizedQuickSort过程中，一个元素$z_{i}$ 会与另一个元素$z_{j}$ （其中 $ i \lt j$） 发生一次比较， 当且仅当$z_{i}$ 与 $z_{j}$ 中的某一个元素在集合 $Z_{ij}$ 的所有其他元素前选为主元（pivot）。而且，每对元素最多比较一次。

**引理7.2的证明**：首先在RandomizedPartition算法执行过程中，集合$ Z_{ij}$ 中的一个元素 $x$ 被选为主元的话，会有以下三种情形需要考虑：

1. 若 $z_{i} \lt x \lt z_{j}$ , 那么 $z_{i}$ 与 $z_{j}$ 之间以后就不会相互比较了，因为它们会被分到 主元$x$的两侧，属于不同的子数组了。
2. 若 $x = z_{i}$ ，则Partition 会比较 $z_{i}$ 和 集合$ Z_{ij}$ 的每个其他元素。
3. 若 $x = z_{j}$ ，则Partition 会比较 $z_{j}$ 和 集合$ Z_{ij}$ 的每个其他元素。

因此 $z_{i}$ 与 $z_{j}$ 之间发生比较操作当且仅当 集合$ Z_{ij}$ 中第一个被选为主元的元素为 $z_{i}$ 与 $z_{j}$ 的其中之一。在最后两个情形中， $z_{i}$ 与 $z_{j}$ 的其中之一被选为主元，由于主元在一次partition完成后，无法再次参与后续的元素比较，所以任意 $z_{i}$ 与 $z_{j}$ 之间比较过一次后，再也无法比较第二次， 引理7.2得证。

下一个引理是关于两个元素之间发生比较的概率。

**引理7.3**

在一个有n个元素， $z_{1}\lt z_{2} \lt . . . \lt z_{n} $ , 的数组上执行RandomizedQuickSort过程中，任意两个元素 $z_{i}$ 与 $z_{j}$ （其中 $ i \lt j$），之间发生比较的概率为 $2/(j - i + 1)$。

**引理7.3的证明**：我们来看看RandomizedQuickSort产生的递归调用树：初始时，根节点的集合包含数组中每一个元素，在集合$ Z_{ij}$ 中的一个元素 $x$ 被选为主元完成划分前，集合$ Z_{ij}$ 的所有元素都呆在一个递归调用节点上。在划分后，主元 $x$ 不会 出现在后续的递归调用的输入集合里。第一次任意 $x \in Z_{ij}$ 被选为主元的概率为 $1/ (j -i +1)$ , 因为 集合$ Z_{ij}$ 中的每个元素被选为主元的概率时相等的。那么，基于引理7.2， 我们有 :
$$
\begin{aligned}Pr \left\{z_{i}与z_{j}之间发生比较\right\} &= Pr \left\{z_{i}或z_{j}在集合Z_{ij}中被选为第一个主元\right\} \\ 
&=  Pr \left\{z_{i}在集合Z_{ij}中被选为第一个主元\right\} \\ 
& \space\space\space\space\space\space\space\space +  Pr \left\{z_{j}在集合Z_{ij}中被选为第一个主元\right\} \\
&= \frac{2}{j - i +1}
\end{aligned}
$$
上式中的第二行成立的原因在于两个事件是互斥的，引理7.3得证。

现在我们可以完成RandomizedQuickSort的分析了。

**引理7.4**

对于一个输入为有n个唯一值的元素， $z_{1}\lt z_{2} \lt . . . \lt z_{n}$ , 的数组的RandomizedQuickSort的期望运行时间为 $O(nlgn)$。

**引理7.4的证明**：我们的分析要用到指示器分析变量（见5.2节) 。 对于 $1 \leq i \lt j \leq n$ , 定义一个随机变量$X_{ij} = I \left\{z_{i}与z_{j}之间发生比较\right\}$ , 从引理2，每一对 $i, j$ 最多比较1次， 我们可以把总比较次数$X$表达成：
$$
X = \sum _{i = 1} ^{n-1} \sum_{j = i + 1}^{n}X_{ij}
$$
对上式两边取期望，并利用期望值的线性相关性 和引理5.1 以及引理7.3，我们得到
$$
\begin{aligned}
E[X ]&=E\left[ \sum _{i = 1} ^{n-1} \sum_{j = i + 1}^{n}X_{ij}\right] \\
     &=\sum _{i = 1} ^{n-1} \sum_{j = i + 1}^{n}E[X_{ij}] \\
     &= \sum _{i = 1} ^{n-1} \sum_{j = i + 1}^{n}Pr \left\{z_{i}与z_{j}之间发生比较\right\} \\
     &= \sum _{i = 1} ^{n-1} \sum_{j = i + 1}^{n}\frac{2}{j - i +1}
\end{aligned}
$$
求解该累加和的时候，我们可以使用变量替换（$ k = j - i$）和 公式（A.9）中给出的调和级数的界，得到：
$$
\begin{aligned}
E[X ] &= \sum _{i = 1} ^{n-1} \sum_{j = i + 1}^{n}\frac{2}{j - i +1} \\
      &= \sum _{i = 1} ^{n-1} \sum_{k = 1}^{n} \frac{2}{k +1} \\
      &\lt \sum _{i = 1} ^{n-1} \sum_{k = 1}^{n} \frac{2}{k} \\
      &= \sum _{i = 1} ^{n-1}O(lg n) \\
      &= O(nlgn)
\end{aligned}
$$
上式和引理7.1可以让我们得出结论，在输入元素互异的情况下，RandomizedQuickSort的期望运行时间为 $O(nlgn)$。



