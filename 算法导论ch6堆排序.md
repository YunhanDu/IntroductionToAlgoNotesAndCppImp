# 算法导论ch6堆排序笔记与最大堆的c++实现
作者：Claude Du
堆排序的时间复杂度与并归排序一样, 空间复杂度与插入排序同样有空间原址性。堆排序是集合了并归排序与插入排序两者优点的一种排序算法，其使用了称为“堆”的数据结构，该数据结构不仅可以用于堆排序，还可以构造有效的优先队列。

## 6.1 堆

堆：堆是一个数组，可以看成一个近似的完全二叉树

一个节点的父节点，左孩子和右孩子的相应下标，对应关系如下代码所示，相应图示如图6.1：

```c++
// the index of this MaxHeap follows the convention of the Introduction to Algorithm Chap6
// which means the root of a maxHeap is maxHeap[0] instead of maxHeap[1],
// and the last element of a maxHeap is  maxHeap[heapSize] instead of maxHeap[heapSize - 1]
inline size_t PARENT(size_t x) {
    return x / 2;
}
inline size_t LEFT(size_t x) {
    return x * 2;
}
inline size_t RIGHT(size_t x) {
    return x * 2 + 1;
}
```

最大堆的性质：$A[PARENT(i)] \geq A[i]$

最小堆的性质：$A[PARENT(i)] \leq A[i]$

![最大堆parentChild](D:\离职计划准备\可信专业级准备\科目一算法联系总结\算法基础Ch6\最大堆parentChild.PNG)

如果把堆看成一棵树，那么堆中某一节点的高度：该节点到叶结点的最长简单路径上边的数目。堆上的一些基本操作的运行时间基本和树的高度成正比。

接下来要介绍的是堆上的一些核心基本操作过程，并说明如何在排序算法和优先队列中应用它们：

6.2 维护最大堆的性质

6.3 构造最大堆

6.4 堆排序

6.5 利用堆实现优先队列

## 6.2 维护最大堆的性质

MaxHeapify是用于维护最大堆性质的重要过程。在任何时候调用MaxHeapify时，我们假定根节点为LEFT(i) 和RIGHT(i)的二叉树都是最大堆。

MaxHeapify 通过不断让vec_[i]在堆中“逐层下降”，从而使得以下标i为根节点的子树再次遵循最大堆的性质。

MAX-HEAPIFY c++代码实现如下：

```C++
template <typename T> class MaxHeap {
private:
    vector<T> vec_;
    size_t heapSize = 0;
    size_t size = 0;
    // recursion version of MaxHeapify, personally don't like it
    void MaxHeapify(size_t i) {
        size_t left = LEFT(i);
        size_t right = RIGHT(i);
        size_t largest = i;
        if (left <= heapSize && vec_[left] > vec_[largest]) largest = left;
        if (right <= heapSize && vec_[right] > vec_[largest]) largest = right;
        if (largest == i) return;
        // exchange vec_[i] with vec_[largest]
        T temp = vec_[i];
        vec_[i] = vec_[largest];
        vec_[largest] = temp;
        MaxHeapify(largest);
    }
public: 
    MaxHeap(vector<T> vec): vec_(vec){
        if (vec.size() < 1) std::cerr << "the size of Heap is smaller than one" << std::endl;
        heapSize = vec.size();
        size = vec.size();
        vec_.insert(vec_.begin(), 0); 
    }
};
```

时间复杂度为$O(lgn)$, 空间复杂度为$O(1)$，运行例子与图示见书。

其中习题6.2-6要求我们实现迭代版的MaxHeapify

迭代版MaxHeapify实现如下：

```c++
template <typename T> class MaxHeap {
private:
    vector<T> vec_;
    int heapSize;
    void MaxHeapifyIterationVersion(size_t i) {
        if (i == 0) return;
        while (i < heapSize) {
            size_t left = LEFT(i);
            size_t right = RIGHT(i);
            size_t largest = 0;
            if (left <= heapSize && vec_[left] > vec_[i]) {
                largest = left;
            } else largest = i;
            if (right <= heapSize && vec_[right] > vec_[largest]) {
                largest = right;
            }
            if (largest == i) return;
            // exchange vec_[i] with vec_[largest]
            std::swap(vec_[largest], vec_[i]);
            i = largest;
        }
    }
public:   
    MaxHeap(vector<T> vec): vec_(vec), heapSize(vec.size() - 1) {}
};
```

时间复杂度为$O(lgn)$，空间复杂度为$O(1)$。

## 6.3 建最大堆

基于6.2节6.1-7习题可知$\lfloor heapSize/2 \rfloor + 1$, $\lfloor heapSize/2 \rfloor + 2$, ..., $heapSize$都是叶结点以及 MaxHeapify的调用前提，我们必须要自底而下才能正确建立最大堆。

建堆的c++代码实现如下：

```c++
void BuildMaxHeap() {
    heapSize = vec_.size() - 1;
    for (int i = heapSize/2; i > 0; --i) {
        MaxHeapifyIteration(i);
    }
}
```

为了证明BuildMaxHeap的正确性，我们要使用如下的**循环不变式**：

每次for循环的开始，节点 i+1， i+2， ..., n 都是一个最大堆的根节点。

我们要证明该循环不变式在初始化(initialization)阶段，保持（maintenance）阶段都为真, 当循环结束时，该不变式可用于证明其正确性。

初始化（initialization）：在第一次循环迭代前，$i = \lfloor size/2 \rfloor$,  而$\lfloor size/2 \rfloor + 1$, $\lfloor size/2 \rfloor + 2$, ..., $size$都是叶结点，因为是平凡最大堆的根节点。

保持（maintenance) : 如果循环的第k次迭代前，节点 $i = \lfloor size/2 \rfloor-k + 1$,  节点$\lfloor size/2 \rfloor - k + 2$, $\lfloor size/2 \rfloor - k + 3$, ...,  $size$ 都是一个最大堆的根节点，

调用MaxHeapifyIteration(A, i)会使节点 $i$ 也成为最大堆的根。于是当循环的第k+1次迭代前，节点 $i = \lfloor size/2 \rfloor-k$，节点$\lfloor size/2 \rfloor - k + 1$, $\lfloor size/2 \rfloor - k + 2$, ...,  $size$ 都是根的最大节点，循环不变式为真。

终止（termination）：终止过程终止时， $i=0$。根据循环不变量，每个节点1，..., n都是一个最大堆的根。

Build-MAX-HEAP的运行时间上界是$O(n)$而非直觉中的$O(nlgn)$, 证明见书中内容，比较简单。

## 6.4 堆排序

初始时，堆排序算法用BuildMaxHeap 把数组vec_[1: n]建成最大堆, 其中 n = vec_.size() -1. 堆中最大元素永远是vec_[1]，通过将vec__[1]与vec_[n]互换，最大的元素来到数组末尾，我们把从堆中把节点n去掉（通过--heapSize实现），剩余节点中除了vec_[1]都是自己最大堆中的根节点，我们可以用maxHeapify来让维护最大堆。如此循环往复我们可以完成排序。

HeapSort的c++实现代码如下：

```c++
void HeapSort() {
    BuildMaxHeap();
    for (int i = heapSize; i >= 2; --i) {
        T max = vec_[1];
        vec_[1] = vec_[heapSize];
        vec_[heapSize] = max;
        --heapSize;
        MaxHeapifyIteration(1);
    }
}
```

时间复杂度为$O(nlgn)$.

## 6.5 优先队列

优先队列是一种用来维护一组元素构成的集合S的数据结构，其中的每一个元素都有一个相关的值，称为关键字（key）。一个最大优先队列S支持以下操作：

1 S.Insert(key): 把键值为key的元素插入集合S中。

2 S.Maximum(): 返回S中具有最大关键字值的元素。

3 S.ExtractMax(): 去掉并返回S中具有最大关键字值的元素。

4 S.IncreaseKey(i, key): 将元素i的关键字值增加到key,  这里key必须不小于i的原来关键字值。

最大优先队列的应用：共享计算机系统的作业调度，具体见书。

最小优先队列的应用：基于事件驱动的模拟器，具体见书。

显然优先队列可以用堆实现，第四版书简要讨论了要如何将优先队列的元素与应用中的对象进行对应：1.使用句柄（handles）2. 在优先级队列里使用映射，比如哈希表（hash Table）。本文不想扯得太复杂，以下优先队列的基本操作算法实现还是遵循第三版的书来实现，第三版中， vec_ 的index对应元素， vec_ [index]为元素的关键字值，并不严谨但无伤大雅（毕竟容易实现，-_-||）

回到最大优先队列的基本操作，并一一实现它们。

S.Maximum(): 返回S中具有最大关键字值的元素, 实现如下：

```c++
T HeapMaximum() {
    return vec_[1];
}
```



S.ExtractMax(): 去掉并返回S中具有最大关键字值的元素，, 实现如下：

```c++
T HeapExtractMax() {
    if (heapSize < 1) {
        std::cerr << "heapUnderflow" << std::endl;
    }
    T max = vec_[1];
    vec_[1] = vec_[heapSize];
    --heapSize;
    MaxHeapifyIteration(1);
    return max;
}
```

时间复杂度为$O(lgn)$

S.IncreaseKey(i, key): 将元素i的关键字值增加到key, 这里要留意一下，元素i的关键字值增加到key可能会违反最大堆的性质，我们要向insertionsort那样， 在节点i到根节点的路径上，为新增的关键字寻找恰当的交换位置，具体实现如下：

```c++
void HeapIncreaseKey(size_t i, T key) {
    if (key < vec_[i]) {
        std::cerr << "new key is smaller than the current Key" << std::endl;
        return;
    }
    vec_[i] = key;
    while (i > 1 && vec_[PARENT(i)] < vec_[i]) {
        // exchange vec_[i] with  vec_[PARENT(i)]
        std::swap(vec_[PARENT(i)], vec_[i]);
        i = PARENT(i);
    }
}
```

时间复杂度为$O(lgn)$

S.Insert(ey): 把键值为key的元素插入集合S中。思路很简单，首先通过增加一个关键字为$-\infty$ (其实任意小于等于key的值都行，我用了key)来扩展最大堆，然后用ncreaseKey(heapSize, key)来维护最大堆的性质，具体实现如下：

```c++
void MaxHeapInsert(T key) {
++heapSize;
if (heapSize > size) {
++size;
// we should set it as -infinity: vec_[heapSize] = -infinity;
// But I have no idea how to do it
vec_.emplace_back(key);
} else vec_[heapSize] = key;
HeapIncreaseKey(heapSize, key);
}
```

时间复杂度为$O(lgn)$

整体最大堆的代码实现如下：

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using std::vector;
// the index of this MaxHeap follows the convention of the Introduction to Algorithm Chap6
// which means the root of a maxHeap is maxHeap[0] instead of maxHeap[1],
// and the last element of a maxHeap is  maxHeap[heapSize] instead of maxHeap[heapSize - 1]
inline size_t PARENT(size_t x) {
    return x / 2;
}
inline size_t LEFT(size_t x) {
    return x * 2;
}
inline size_t RIGHT(size_t x) {
    return x * 2 + 1;
}
template <typename T> class MaxHeap {
private:
    vector<T> vec_;
    size_t heapSize = 0;
    size_t size = 0;
    // recursion version of MaxHeapify, personally don't like it
    void MaxHeapify(size_t i) {
        size_t left = LEFT(i);
        size_t right = RIGHT(i);
        size_t largest = i;
        if (left <= heapSize && vec_[left] > vec_[largest]) largest = left;
        if (right <= heapSize && vec_[right] > vec_[largest]) largest = right;
        if (largest == i) return;
        // exchange vec_[i] with vec_[largest]
        T temp = vec_[i];
        vec_[i] = vec_[largest];
        vec_[largest] = temp;
        MaxHeapify(largest);
    }
    // iteration version of MaxHeapify
    void MaxHeapifyIteration(size_t i) {
        if (i == 0) return;
        while (i <= heapSize/2) {
            size_t left = LEFT(i);
            size_t right = RIGHT(i);
            size_t largest = i;
            if (left <= heapSize && vec_[left] > vec_[largest]) largest = left;
            if (right <= heapSize && vec_[right] > vec_[largest]) largest = right;
            if (largest == i) return;
            // exchange vec_[i] with vec_[largest]
            std::swap(vec_[largest], vec_[i]);
            i = largest;
        }
    }

public: 
    void BuildMaxHeap() {
        heapSize = vec_.size() - 1;
        for (int i = heapSize/2; i > 0; --i) {
            MaxHeapifyIteration(i);
        }
    }

    size_t Size() {
        return size;
    }
    T& operator[] (size_t i) {
        return vec_[i];
    }
    void HeapSort() {
        BuildMaxHeap();
        for (int i = heapSize; i >= 2; --i) {
            T max = vec_[1];
            vec_[1] = vec_[heapSize];
            vec_[heapSize] = max;
            --heapSize;
            MaxHeapifyIteration(1);
        }
    }
    MaxHeap(vector<T> vec): vec_(vec){
        if (vec.size() < 1) std::cerr << "the size of Heap is smaller than one" << std::endl;
        heapSize = vec.size();
        size = vec.size();
        vec_.insert(vec_.begin(), 0); 
        BuildMaxHeap();
    }
    T HeapMaximum() {
        return vec_[1];
    }
    T HeapExtractMax() {
        if (heapSize < 1) std::cerr << "heapUnderflow" << std::endl;
        T max = vec_[1];
        vec_[1] = vec_[heapSize];
        --heapSize;
        MaxHeapifyIteration(1);
        return max;
    }
    void HeapIncreaseKey(size_t i, T key) {
        if (key < vec_[i]) std::cerr << "new key is smaller than the current Key" << std::endl;
        vec_[i] = key;
        while (i > 1 && vec_[PARENT(i)] < vec_[i]) {
            // exchange vec_[i] with  vec_[PARENT(i)]
            std::swap(vec_[PARENT(i)], vec_[i]);
            i = PARENT(i);
        }
    }
    void MaxHeapInsert(T key) {
        ++heapSize;
        if (heapSize > size) {
            ++size;
            // we should set it as -infinity: vec_[heapSize] = -infinity;
            // But I have no idea how to do it
            vec_.emplace_back(key);
        } else vec_[heapSize] = key;
        HeapIncreaseKey(heapSize, key);
    }
};



    
int main() {
    MaxHeap<int> maxHeap(vector<int>{27, 17, 3, 16, 13, 10, 1, 5, 7, 12, 4, 8, 9, 0});
    maxHeap.HeapSort();
    for (int i = 1; i <= maxHeap.Size(); ++i) {
        std::cout << maxHeap[i] << " ";
    }
    std::cout << "\n";
    maxHeap.BuildMaxHeap();
    maxHeap.HeapIncreaseKey(5, 25);
    maxHeap.MaxHeapInsert(41);
    maxHeap.HeapSort();
    for (int i = 1; i <= maxHeap.Size(); ++i) {
        std::cout << maxHeap[i] << " ";
    }
    std::cout << "\n";
    maxHeap.BuildMaxHeap();
    int rootVal = maxHeap.HeapExtractMax();
    std::cout << "the value of the root which has been deleted: " << rootVal << "\n";
    rootVal = maxHeap.HeapExtractMax();
    std::cout <<  "the value of the root which has been deleted: " << rootVal << "\n";
}
```

