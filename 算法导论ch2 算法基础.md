# 算法导论ch2 笔记与插入排序，并归排序C++实现

作者：杜云汉

## 2.1.插入排序

插入排序算法思想：在已排序的子数组A[1:j-1]后，将单个元素A[j]插入子数组的适当位置，产生排序号的子数组A[1:j]。

其C++代码实现：

```c++
// author: Claude Du
#include <iostream>
#include <vector>

using std::vector;
using std::unordered_map;
using std::pair;

class Solution { 
public:
    void insertionSort(vector<int>& nums) {
        if (nums.size() <= 1) return;
        for (int j = 1; j < nums.size(); ++j) {
            int key = nums[j];
            int i = j -1;
            while (i >= 0 && nums[i] > key) {
                nums[i + 1] = nums[i];
                --i;
            }
            nums[i + 1] = key;
        }
    }
};
    
int main()
{
    Solution sol;
    vector<int> nums = {3,2,1,2,3,4};
    sol.insertionSort(nums);
    for (int i = 0; i < nums.size(); ++i) {
        std::cout <<  nums[i] << " ";
    }   
}
```

输出结果，排序成功

```shell
PS D:\WorkSpace> g++ -g insertion.cpp
PS D:\WorkSpace> ./a.exe
1 2 2 3 3 4 
```

英文版第四版书中P19(中文版第三版书中P10)提到了循环不变式（loop invariant）的重要概念（后续会反复用到）, 需要重点理解。

对于插入排序算法， 其循环不变式如下：

A loop invariant of insertion sort is shown here:

1. at the beginning of each iteration of the **for** loop, which is indexed by j, the subarray nums[0: j-1] is sorted.
2. at the beginning of each iteration of the **for** loop, which is indexed by j, the elements of the subarray nums[0: j-1] are the elements originally in positions 1 through j -1.

循环不变式可用来帮助我们证明算法的正确性。 当使用一个循环不变式时，我们要证明以下三条性质成立，即可证明算法的正确性：

1. **初始化**：在循环的第一次迭代前，该循环不变式为真
2. **保持（maintenance)**：如果循环的某次迭代前，该循环不变式为真，那么下次迭代前，它依然为真。
3. **终止（Termination)**：在循环终止时，该循环不变式提供给我们一个有用的性质，该性质可用于证明算法的正确性。

我们看看对于插入排序，以上三条性质是否成立：

1. **初始化**：在循环的第一次迭代前，i = 1, 子序列nums[0:0]只有一个元素nums[0], 循环不变式的两条性质都显然成立。

   第一条得证

2. **保持（maintenance)**：（非形式化的数学归纳法论证），如果循环的某次迭代前，该循环不变式为真，for循环体中将nums[j-1], nums[j-2],...nums[j-k +1]等都向右移动了一个位置（k满足nums[j-k +1] > nums[j] && nums[j-k ] <= nums[j] ），再在pos=j - k +1的位置上插入原来的nums[j], 此时nums[0: j]已完成排序，且nums[0:j]由原来nums[0:j]的元素组成，那么对于for循环下次迭代前，该循环不变式依然为真。

3. **终止（Termination)**：在for循环终止时，j = nums.size(); 在该循环不变式中，我们将j用nums.size()替代， nums[0: nums.size() - 1]是已排序的，并且其中的所有元素都由原先的数组的元素组成， 因为此时数组已排序且所有元素都是该数组原来的元素，该算法是正确的。

这种**循环不变式**的方法会在本书后面的内容持续使用，**必须通过刻意的练习掌握好！**



## 2.2分析算法

该章节的具体内容还是看书吧，这里只写个提要。

本章先简要介绍了RAM模型，其中包含RAM模型的常见指令和代价，RAM模型的数据类型等信息，

之后对插入排序的复杂度进行分析， 该分析方法非常严谨，值得学习，得到插入排序最坏情况运行时间为$\Theta(n^2)$ ， 平均依然为$\Theta(n^2)$



## 2.3设计算法

之前的插入排序使用**增量**方法：在已排序的子数组A[1:j-1]后，将单个元素A[j]插入子数组的适当位置，产生排序号的子数组A[1:j]。

接下来看另一种分治法的设计方法，并用分治法的思想设计排序算法。

2.3.1分治法（The divide-and-conquer method）

分而治之法的思想：将原问题分解为几个规模较小单类似于原问题的子问题，递归地求解这些子问题，然后再合并这些子问题的解来建立原问题的解。

分治模式在内层递归有以下三个步骤：

	1. 分解 : 将原问题分解为若干子问题，这些子问题是原问题规模较小的实例
	1. 解决：递归的求解个子问题。当子问题规模足够小，则直接求解
	1. 合并：合并两个已排序的子序列以产生已排列的答案

归并排序：

1. 分解（divide）：将待排序的子序列 $A[p:r] $ 分解成两个子相邻序列，每个分解出来的子序列的规模为原来的一半。为了完成该分解步骤，算出 $A[p:r] $  的中间索引q，$q = \lfloor (p + r)/2\rfloor$ , 将 $A[p:r] $ 分解成 $A[p:q]$ 和 $A[q+1:r]$  。(该方法可以使得 两数组$A[p:q]$ 和 $A[q+1:r]$的规模值差异不超过1)
2. 解决（conquer）：使用并归排序递归地排序两个子序列 $A[p:q]$ 和 $A[q+1:r]$  。
3. 合并（combine）：合并两个已排序的子序列产生已排序的 $A[p:r] $ 。

根据上面的分析可以很轻松的写出如下的C++实现：

```c++
// author: Claude
void MergeSortRecursive(vector<int>& vec, int start, int end) {    
    if (start >= end) return; // recursive basis
    int mid = start + (end - start) / 2; // midpoint of vec[start:end]
    MergeSortRecursive(vec, start, mid); // recursively sort vec[start:mid]
    MergeSortRecursive(vec, mid + 1, end); // recursively sort vec[mid + 1:end] 
    // merge vec[start:mid] and vec[mid + 1:end] into sorted vec[start:end]
    Merge(vec, start, mid, end);
}
void MergeSort(vector<int>& vec) {
    // the convention of double closed interval is utilized    
    MergeSortRecursive(vec, 0, vec.size()-1);
} 
```

归并算法中的关键一步为合并（即以上代码中的merge函数),  类似于算导第四版合并的c++实现如下：

```c++
// author: Claude Du 
void Merge(vector<int>& vec, int leftStart, int leftEnd, int rightEnd) {
    vector<int> leftVec(vec.begin() + leftStart, vec.begin() + leftEnd + 1); // copy vec[leftStart: leftEnd] into leftVec
    // copy vec[leftEnd + 1: rightEnd] into rightVec
    vector<int> rightVec(vec.begin() + leftEnd + 1, vec.begin() + rightEnd + 1); 
    int leftPos = 0; // leftPos indexes the smallest element in leftVec
    int rightPos = 0; // leftPos indexes the smallest element in rightVec
    int vecPos = leftStart; // vecPos indexes the location in vec to fill
    while (vecPos <= rightEnd) {
        // As long as each of the arrays leftVec and rightVec contains an unmerged element,
        // copy the smallest unmerged element back into vec[vecPos, rightEnd]
        if (rightPos < rightVec.size() && leftPos < leftVec.size()) {
            if (leftVec[leftPos] < rightVec[rightPos]) {
                vec[vecPos] = leftVec[leftPos];
                ++leftPos;
            } else {
                vec[vecPos] = rightVec[rightPos];
                ++rightPos;                    
            }
		
        }
        // Having gone through one of leftVec and rightVec entirely, copy the
        // remainder of the other to the end of vec[vecPos, rightEnd]
        else if (rightPos < rightVec.size()) {
            vec[vecPos] = rightVec[rightPos];
            ++rightPos; 
        } else {
            vec[vecPos] = leftVec[leftPos];
            ++leftPos;
        }
        ++vecPos;
    }
}
```

合并的时间复杂度为 $\Theta(n)$ , 其中 $n = rightEnd - leftStart + 1$  。

强烈建议看看第四版英语版书中对应的伪代码和第三版中文版的伪代码：第四版比第三版的常规易懂很多，第三版则巧妙的采用了立哨兵的思路来避免数组越界，都是非常值得一看的，再来个第三版思路的C++实现吧：

```c++
// author: Claude Du
void Merge2(vector<int>& vec, int leftStart, int leftEnd, int rightEnd) {
    int leftLength = leftEnd - leftStart + 1;
    vector<int> leftVec(leftLength + 1, INT_MAX);
    // copy vec[leftStart: leftEnd] into leftVec[0, leftEnd - leftStart]
    // the last element, leftVec[leftEnd - leftStart + 1], has been set as INT_MAX on purpose,
    // which functions as a sentry
    for (int i = 0; i < leftLength; ++i) {
        leftVec[i] = vec[leftStart + i];
    }
    int rightLength = rightEnd - leftEnd;
    vector<int> rightVec(rightLength + 1, INT_MAX);
    // copy vec[leftEnd+1: rightEnd] into rightVec[0, rightEnd - leftEnd-1]
    // the last element, rightVec[leftEnd - leftStart], has been set as INT_MAX on purpose,
    // which functions as a sentry
    for (int i = 0; i < rightLength; ++i) {
        rightVec[i] = vec[leftEnd + i + 1];
    }
    int leftPos = 0; // leftPos indexes the smallest element in leftVec
    int rightPos = 0; // leftPos indexes the smallest element in rightVec
    for (int vecPos = leftStart; vecPos <= rightEnd; ++vecPos) {
        if (leftVec[leftPos] < rightVec[rightPos]) {
            vec[vecPos] = leftVec[leftPos];
            ++leftPos;
        } else {
            vec[vecPos] = rightVec[rightPos];
            ++rightPos; 
        }
    }
}
```

以上两段merge代码是可以优化空间复杂度的，这里从原数组vec里拆分并拷贝了两个数组leftVec 和 rightVec，我们其实可以只用拷贝leftVec, 这样可以降低merge中1/2的空间复杂度。其c++代码实现如下：

```c++
// author: Claude Du
void Merge3(vector<int>& vec, int leftStart, int leftEnd, int rightEnd) {
    vector<int> leftVec(vec.begin() + leftStart, vec.begin() + leftEnd + 1); // copy vec[leftStart: leftEnd] into leftVec
    int leftPos = 0; // leftPos indexes the smallest element in leftVec
    int rightPos = leftEnd + 1; // leftPos indexes the smallest element in vec[rightPos: rightEnd]
    int vecPos = leftStart; // vecPos indexes the location in A to fill
    // As long as each of the arrays leftVec and vec[rightPos: rightEnd] contains an unmerged element,
    // copy the smallest unmerged element back into vec[vecPos, rightEnd]    
    while (leftPos < leftVec.size() && rightPos <= rightEnd) {
        if (leftVec[leftPos] < vec[rightPos]) {
            vec[vecPos] = leftVec[leftPos];
            ++leftPos;
        } else {
            vec[vecPos] = vec[rightPos];
            ++rightPos;                    
        }
        ++vecPos;
    }
    // Having gone through one of leftVec and vec[rightPos: rightEnd] entirely, copy the
    // remainder of the other to the end of vec[vecPos, rightEnd]
    while (leftPos < leftVec.size()) {
        vec[vecPos] = leftVec[leftPos];
        ++leftPos; 
        ++vecPos;
    }
    while (rightPos <= rightEnd) {
        vec[vecPos] = vec[rightPos];
        ++rightPos; 
        ++vecPos;
    }
}
```

 这里可能会产生一个疑问：vecPos会不会提前覆盖rightPos, 也是vecPos > rightPos, 导致写入修改vec[rightPos: rightEnd]的数值（数据污染），从而导致下次造成merge算法错误？

哈哈哈，其实不会污染， 可以在附录里简单证明一下【1】。

回到并归排序，分析其复杂度，一下给出归并排序的最坏情况的运行时间$T(n)$的递归式：
$$
T(n) = \left\{\begin{array}{lcl}
c & { n =1}\\
2*T(n/2) + cn & {n>1}
\end{array} \right.
$$
可计算得出$T(n)$为 $\Theta(nlgn)$

## 附录：



1 merge3的优化代码中不会存在vecPos > rightPos, 导致写入修改vec[rightPos: rightEnd]的数值（数据污染）的情形，证明如下

每一次++vecPos, 都会伴随着++leftPos或者++rightPos。

这意味着 vecPos的总变化量 为leftPos变化量与rightPos变化量只和：$\Delta(vecPos) =\Delta(leftPos) + \Delta(rightPos)$

即 $vecPos(n) - vecPos(0) = (leftPos(n) - leftPos(0)) + (rightPos(n) -rightPos(0)) $ ,  这里$n$ 表示第 $n$ 次迭代后，如$vecPos(n)$表示第n次迭代后vecPos的数值, 这里 $ rightEnd - leftStart + 1\geq n \geq1$

把各变量初始值代入:

```c++
int leftPos = 0;
int rightPos = leftEnd + 1;
int vecPos = leftStart;
```

得到：

$rightPos(n) = vecPos(n) + (leftVec.size() - leftPos(n)) $

很显然 $(leftVec.size() - leftPos(n))$ 这项永远 $\geq 0$ ，则 $rightPos \geq vecPos$ , 当  $= 0$ 时意味着leftPos已刚刚数组越界， 此时vec[vecPos]只能读取vec[rightPos]，不存在数据污染的情形。哈哈哈，一个博尔特终究跑不赢两个切换摸鱼的博尔特。



