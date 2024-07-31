# 哈希表算法导论笔记与其c++实现（上）

作者：Claude Du

算法导论第四版英文版相比与第三版有较大改动(苍天。。。工作量几乎翻倍），本笔记是基本基于第四版英文版的最新内容，尝试浅谈哈希表背后的算法细节与数学理论分析，同时也是少有的一篇会实际兼顾哈希表的两种c++实现方式以及全域哈希函数的c++实现的文章，下面一起看看本篇笔记吧。

动态集合数据结构至少要支持Insert, Search和Delete这样的字典操作。例如，编译器都会维护一个字典（符号表），其中字典中元素的关键字（key）为任意字符串，它与程序语言·中的标识符相对应。哈希表（Hash Table）是实现字典的一种有效数据结构，在实际应用中，其Search性能极好，其查找一个元素的期望时间为 $O(1)$ （最坏情形下，Search期望时间为  $O(n)$ ，与链表相同）。实际上，Python的内置字典的底层实现就是哈希表。

PS: 字典的底层实现有很多种，除了哈希表，还有红黑树【1】, AVL树【2】，与跳表等。

这里的字典元素Entry类与字典操作接口的C++代码如下：

```c++
#ifndef Dictionary_h
#define Dictionary_h

template <typename K, typename V> struct Entry {
    K key_;
    V val_; 
    Entry(K key = K(), V val = V()): key_(key), val_(val) {}
    Entry(Entry<K, V> const& e) : key_(e.key_), val_(e.val_) {}
    bool operator> (Entry<K, V> const& e) { return key_ > e.key_; }
    bool operator< (Entry<K, V> const& e) { return key_ < e.key_; }
    bool operator== (Entry<K, V> const& e) { return key_ == e.key_; }
    bool operator!= (Entry<K, V> const& e) { return key_ != e.key_; }
};
template <typename K, typename V> struct Dictionary {
    virtual int size() const = 0;
    virtual bool Insert (K key, V val) = 0;
    virtual V* Get(K key) = 0;
    virtual bool Remove(K key) = 0;
};

#endif /*Dictionary_h*/
```

哈希表是普通数组的推广，普通数组通过直接寻址可以在 $O(1)$ 时间访问数组中的任意位置。11.1节就讨论直接寻址表与其c++实现。

当实际需求中要存储的关键字数目比全部的可能关键字总数要小很多时，采用哈希表来替代直接数组寻址会更有效，因为HashTable使用了一个长度与实际需求中要存储的关键字数目成比例的数组来存储（这里成比例指的是尺度相近，具体见11.2）。哈希表中的元素会根据关键字计算出相应的下标。11.2节就介绍哈希表的主要思想，着重介绍通过链接（chaining）方法解决冲突(collision)，并以该方式实现第一种哈希表。所谓冲突就是指多个关键字映射到同一个下标。11.3节介绍如何利用哈希函数来计算关键字对应的数组下标，还将介绍和分析散列技术的几种变形。11.4节介绍“开放寻址法”，这是另一种处理冲突的方法并基于此方法呈现哈希表的另一种实现方式。11.5节探讨了现代计算机系统的层级存储结构以及如何在这种系统下设计性能良好的哈希表，11.6节将探讨哈希表的应用。

本文章篇幅会非常长，所以会拆分上下两部分：

第一部分包含11.1节到11.3节，包含哈希表的链接法c++实现，以及全域哈希函数的c++实现。

第二部分包含11.4节到11.6节，包含哈希表的开放寻址法的实现，该方法的c++实现会一定程度上借鉴邓俊辉的《数据结构（c++语言版)》【】。

## 11.1直接寻址表

内容比较简单，和数组几乎没有区别，第四版内容也与第三版没有区别，具体内容可以直接看第三版中文版。

## 11.2哈希表

直接寻址表的缺点也很明显：

1.如果全域 $U$ 很大, 要存下大小为 $|U|$ 的一张表不太实际。

2，当实际需求中要存储的关键集合 $K$ 比全域 $U$ 要小很多时，直接寻址表里的绝大部分空间都被浪费了。

当实际需求中要存储的关键集合 $K$ 比全域 $U$ 要小很多时，散列表需要的存储空间比直接寻址表小很多，其空间大小仅为 $\Theta(|K|)$ , 同时Search操作期望用时依然只有 $O(1)$  （最坏情形下，Search期望时间为  $O(n)$ ，与链表相同， 比直接寻址表要差）。

不同于直接寻址中把关键字k 的元素放在slot k 中，哈希表中，该元素存放在槽 $h(k)$ 中，即利用哈希函数(hash function) h, 由关键字k计算出slot的位置h(k), h(k)即是k的哈希值。这里 函数h将关键的全域 $U$ 映射到散列表 $T[0..m-1]$ 的槽位上：

$$h:U\rightarrow\left\{0,1,...,m-1\right\}$$

这里哈希表的大小m一般要比 $|U|$ 小很多，从而节省了大量存储空间。下图描述了这个基本方法。

这里存在一个问题：两个关键字可能映射到同一个下标上，即同一个slot中。我们称这个情形为冲突(Collision)。幸运的是，我们能找到一些有效的思路来解决冲突。

思路1：理想的解决办法是避免所有的冲突。我们可以试图选择一个合适的哈希函数h来尽量做到这点，让h尽可能的随机，从而让冲突发生的可能性最小化，11.3节就是交代合适的哈希函数。

思路2：当出现冲突时，我们最直观的冲突解决方法称为链接法(chaining)，本节余下部分介绍这一方法。11.4节介绍另一种冲突解决方法，开放寻址法（open addressing）。

思路1与思路2通常要一起结合使用。

### 通过链接法解决冲突

在链接法中, 把哈希到同一slot的所有元素都放在一个链表中，如下图所示：

链接法的哈希表实现框架如下：

```c++
// author: Claude Du
#include "BitMap.h"
#include "Dictionary.h"
#include <string>
#include <cstring>
#include <vector>
#include <list>


template <typename K, typename V> class HashTable: public Dictionary<K,V> {
private:
    std::vector<std::list<Entry<K, V>>>* ht;
    int M; // Slot Number
    int N; //  key Number
    typename std::list<Entry<K, V>>::iterator Find(int rank, K key);
protected:
    void Rehash();
public:
    HashTable(int c = 5);
    ~HashTable();
    int size() const { return N; }
    bool Insert (K key, V val);
    V* Get(K key);
    bool Remove(K key);
    size_t Hash(const K key);
};

template <typename K, typename V> HashTable<K, V>::HashTable(int c) {
    // Generate the smallest prime number which is not smaller than c 
    M = primeNLT(c, 1048576, "prime-bitmap.txt");
    N = 0;
    ht = new std::vector<std::list<Entry<K, V>>>(M, std::list<Entry<K, V>>{});   
}
template <typename K, typename V> HashTable<K, V>::~HashTable() {
    for (auto& lst : (*ht)) {
        if (!lst.empty()) {
            lst.clear();
        }
    }
    ht->clear();
    N = 0;
    M = 0;
    delete ht;
    ht = nullptr;
}
```

我这里第11行直接使用了std的vector和list来实现该数据结构中slot数组与双向链表的部分，这里第11行使用指针的原因，是为了避免哈希表扩容（Rehash）操作时会出现两次拷贝vector元素的操作。

在采用链接法解决冲突后，哈希表上的字典操作（插入，删除与查找）就很容易实现了, 其c++实现如下：

```c++
template <typename K, typename V> bool HashTable<K, V>::Insert(K key, V val) {
    int rank = Hash(key);
    if (Find(rank, key) != (*ht)[rank].end()) return false;
    ++N;
    (*ht)[rank].push_front(Entry<K, V>(key, val));
    if (N * 2 > M) {
        std::cout << "start rehashing.\n";
        Rehash();
    }
    return true;
}
template <typename K, typename V> bool HashTable<K, V>::Remove(K key) {
    int rank = Hash(key);
    typename std::list<Entry<K, V>>::iterator toBedeleted = Find(rank, key);
    if (toBedeleted == (*ht)[rank].end()) return false;
    (*ht)[rank].erase(toBedeleted);
    --N;
    return true;
}
template <typename K, typename V> V* HashTable<K, V>::Get(K key) {
    int rank = Hash(key);
    typename std::list<Entry<K, V>>::iterator toBeUsed = Find(rank, key);
    if (toBeUsed == (*ht)[rank].end()) return nullptr;
    return &(toBeUsed->val_);
}
// find the certain element of the hash table
template <typename K, typename V> typename std::list<Entry<K, V>>::iterator HashTable<K, V>::Find(int rank, K key) {
    if ((*ht)[rank].empty()) return (*ht)[rank].end();
    for (auto ele = (*ht)[rank].begin(); ele != (*ht)[rank].end(); ++ele) {
        if (ele->key_ == key) return ele;
    }
    return (*ht)[rank].end();
}
// expanding hash table
template <typename K, typename V> void HashTable<K, V>::Rehash() {
    int old_capacity = M;
    std::cout << "old capacity is " << old_capacity << std::endl;
    std::vector<std::list<Entry<K, V>>>* old_ht = ht;
    M = primeNLT(2*M, 1048576, "prime-bitmap.txt");
    std::cout << "new capacity is " << M << std::endl;

    ht = new std::vector<std::list<Entry<K, V>>>(M, std::list<Entry<K, V>>{}); 
    for (int i = 0; i < old_capacity; ++i) {
        if (!(*old_ht)[i].empty() ) {
            for (auto & ele : (*old_ht)[i]) {
                Insert(ele.key_, ele.val_);
            }
            (*old_ht)[i].clear();
        }
    }
    old_ht->clear();
    delete old_ht;
    old_ht = nullptr;
}
```

实际实现中，插入操作和删除的最坏情况运行时间为$O(n)$ 。因为在插入和删除执行前，我们要搜索相应的链表中是否有该元素，搜索这一步的最坏运行时间为$O(n)$。搜索的性能分析见后面的分析，

#### 链接法哈希表的分析

采用链接法的哈希表性能怎么样呢？比如查找一个具有给定关键字的元素需要多长时间呢？

给定一个能存放n个元素的，具有m个slot的散列表T, 定义T的装载因子(load factor）$\alpha$ 为 $n/m$ , 即一个链的平均存储元素数。我们的分析将借助 $\alpha$ 来说明， $\alpha$ 可以小于，等于或大于 1。

其最坏情形性能很差：当所有n个元素都在一个slot中，查找时间为 $\Theta(n)$ ，与链表的查找元素时间一致。

但我们不能因为其最坏性能差，就不使用哈希表。哈希表的平均性能依赖与所选的散列函数将关键字分布在m个slot上的均匀程度，我们在11.3节讨论这个问题。**我们先假定任何一元素都等可能的哈希映射到m个slot中的任何一个，且与其他元素哈希映射到哪无关。我们称该假设为独立均匀哈希（independent uniform hashing）。**

对于 $j=0,1,...,m-1$, 列表 $T[j]$ 的长度用 $n_j$ 表示，于是有
$$
n = n_0 + n_1+...+n_{m-1}
$$
并且 $n_j$ 的期望值为 $E[n_j]=\alpha = n/m$ （简单证明可以用期望的线性相关性，也可用二项展开求导的方式硬证）。

假定可以在 $O(1)$ 时间内计算出哈希值 $h(k)$ ，从而查找关键字为k的元素的时间线性地依赖于表 $T[h(k)]$ 的长度 $n_{h(k)}$ 。先不考虑计算哈希函数和访问slot $h(k)$ 的 $O(1)$ 时间，我们来看看查找算法查找元素的期望数，即为比较元素的关键字是否为k而检查表 $T[h(k)]$ 中的元素数。分两种情形来考虑：

1，表中没有元素关键字为 $k$ 。

2，成功地查找到关键字为 $k$ 的元素。

**定理11.1** 在简单均匀哈希的假设下，对于用链接法解决冲突的散列表，一次成功查找所需的平均时间为 $\Theta(1+\alpha)$ 。

**证明：**在简单均匀哈希的假设下，任何尚未被存储在表中的关键字k都等可能地哈希到m个slot中的任何一个。因而当查找一个关键字 $k$ 时，在不成功的情况下，查找的期望时间就是查找至链表 $T[h(k)]$ 末尾的期望时间，这一时间的期望长度为  $E[n_{h(k)}]=\alpha = n/m$ 。于是，一次不成功的查找平均要查 $\alpha$ 个元素，且所需要的总时间为 $\Theta(1+\alpha)$ 。

对于成功的查找来说，情况略有不同，这是因为每个链表并不是等可能地被查找到的，比如空slot是无法成功查找的。**假定要查找的元素是表中存放的n个元素中任何一个，且是等可能的。**所以slot中的链表长度越长，我们越有可能查找该链表中的元素。即使这样， 期望的查找时间是 $\Theta(1+\alpha)$ 

**定理11.2** 在简单均匀哈希的假设下，对于用链接法解决冲突的哈希表，一次成功查找所需的平均时间为 $\Theta(1+\alpha)$。

**证明：**我们已经假设要查找的元素是表中存放的n个元素中任何一个，且是等可能的。在对元素x的一次成功查找中，对x所在的链表中x前面的元素数多1。在该链表中，因为新的元素都是在链表头插入的，所以出现在x之前的元素都是在x插入后插入的。设 $x_i$ 表示插入到表中的第i个元素，$i=1,2,...,n,$  并设 $k_i = x_i.key$ 。

对每个哈希表中的slot q， 关键字 $k_i$ 和 $k_j$ , 定义指示器随机变量 

$$X_{ijq} = I\left\{\text{the search for } x_i\text{, }h(k_i)=q \text{，and } h(k_j)=q\right\}$$ 

那么，当搜索 的元素是$x_i$，关键字 $k_i$ 和 $k_j$ 在slot q碰撞时，$X_{ijq} = 1$ 。因为 

$Pr\left\{\text{the search is for }x_i\right\} = 1/n$ ,

 $Pr\left\{h(k_i)=q\right\}=1/m$ ,

 $Pr\left\{h(k_j)=q\right\}=1/m$ , 

三项事件相互独立，我们有 $Pr\left\{X_{ijq}\right\}=Pr\left\{\text{the search is for }x_i\right\}\cdot Pr\left\{h(k_i)=q\right\}\cdot Pr\left\{h(k_j)=q\right\}= 1/nm^2$,

 根据第5章的引理5.1，我们得到 $E[X_{ijq}]=1/nm^2$ 。

PS: 一次成功查找中，我们有 $n*m*m$ 个可能的 $X_{ijq}$ , 其中只有一个 $X_{ijq}$ 能为1，其余都为0。 

接下来，对于每个元素$x_j$ , 我们定义指示器随机变量

$$\begin{aligned}Y_j &= I\left\{x_j\text{ appears in a list prior to the element being searched for } \right\}\\ &=\sum_{q=0}^{m-1} \sum_{i=1}^{j-1}X_{ijq}\end{aligned}$$

由于链地址法中采用的是头插法，当目标元素 $x_i$ 和 $x_j$ 在同一个链表中（slot q指向的那个链表），以及 $i<j$ (这样的话，在链表中 $x_i$ 会出现在 $x_j$ 的后面 )，那么 $x_j$ 会在 $x_i$ 之前出现。

我们最后要定义的随机变量是$Z$ , 用来统计有多少元素会出现在要搜索的元素前：

 $$\begin{aligned}Z=\sum_{j=1}^{n}Y_j\end{aligned}$$

因为我们必须要统计要被成功搜索的元素加上其链表中在其链表位置之前的所有元素，我们要计算 $E[Z+1]$ 。使用期望的线性性质，我们得到
$$
\begin{aligned}
E[Z+1] &=  E[1+\sum_{j=1}^{n}Y_j] \\
& = 1 + E[\sum_{j=1}^{n}\sum_{q=0}^{m-1} \sum_{i=1}^{j-1}X_{ijq}] \\ 
& =  1 + E[\sum_{q=0}^{m-1}\sum_{j=1}^{n}\sum_{i=1}^{j-1}X_{ijq}] \\ 
& = 1 + \sum_{q=0}^{m-1}\sum_{j=1}^{n}\sum_{i=1}^{j-1} E[X_{ijq}] &&\text{(by linearity of expectation)}\\ 
& = 1 + \sum_{q=0}^{m-1}\sum_{j=1}^{n}\sum_{i=1}^{j-1} \frac{1}{nm^2} \\
& = 1 + \sum_{q=0}^{m-1}\sum_{j=1}^{n}\frac{j-1}{nm^2} \\
& = 1 + \sum_{q=0}^{m-1}\frac{n(n-1)}{2}.\frac{j-1}{nm^2} \\
& = 1 + m.\frac{n(n-1)}{2}.\frac{j-1}{nm^2} \\
& = 1 + \frac{(n-1)}{2m} \\
& = 1 + \frac{n}{2m}- \frac{1}{2m}\\
& = 1 + \frac{\alpha}{2}- \frac{\alpha}{2n}\\
\end{aligned}
$$
因此，成功搜索的总耗时（包括计算哈希函数的时间）为 $\Theta(2+\alpha/2-\alpha/2n) = \Theta(1+\alpha)$ 。

上面的分析意味着什么呢？如果散列表中slot数与表中的元素数成正比，则有 $n = O(m)$ , 从而 $\alpha = n/m = O(m)/m = O(1)$。所以，查找操作平均需要常数时间，由于插入与删除操作这里耗时最多的操作也是查找，所以这两个操作的平均运行时间也是$O(1)$ 。



## 11.3哈希函数

为了让哈希表正常工作，我们需要好的哈希函数。除了让哈希函数能够快速计算，一个好的哈希函数要有什么样的性质呢？我们该如何设计好的哈希函数呢？

本节首先用两种临时的方法去创造哈希函数来回答上面的问题：用除法进行哈希和用乘法进行哈希。这两种方法是受限的，因为他们试图通过提供一个固定的哈希函数让其在任何数据上都能正常工作，这个目标并不能很好地做到。

之后我们会看到：想要获得对于任何数据都可以有较好的平均性能的哈希函数，我们可以设计一个合适的哈希函数簇，然后运行时随机从哈希函数簇中选择一个出来作为正式哈希函数，该随机选择和输入数据无关。这个方法被称为随机哈希。这个方法称为随机哈希，有一种特殊的随机哈希，全域哈希法，性能很好。就像第7章的随机快速排序（），随机算法是算法设计中真的很重要。

### 什么是好的哈希函数？

一个好的哈希函数近似满足**独立均匀哈希**：每个关键字都被等可能地哈希到m个slot中的任何一个，并与其他关键字哈希到哪个slot无关。遗憾的是，一般无法检查这一条件是否成立除非你本来就知道输入数据的关键字的分布模式。

一个好的静态哈希法产生哈希值的方式要和数据关键字的分布模式独立。举个例子，比如除法哈希计算哈希值时除数得是一个特定的质数（11.3.1节），如果该质数和输入数据关键字的概率分布模式无关，该方法可能会有好结果。

随机哈希，在11.3.2节中会详细描述，会随机从哈希函数簇中选择一个出来作为正式哈希函数。该方法完全不需要知道输入数据的概率分布模型，所以强烈推荐使用随机哈希。

11.3.1 静态哈希（Static Hashing）

除法哈希



乘法哈希



乘法移位哈希

实践中，乘法哈希法在哈希表的slot数是2的精确幂的时候表现是最好的，即$m = 2^l, l\in \mathbb{N} \text{ }\wedge \text{ }l\leq w$ ，其中 $w$ 为机器字比特数，取 $a=A2^w$ , 其中 $0 < A < 1$ (Knuth建议取黄金分割 $A\approx (\sqrt{5} -1)/2$) , 则 $0<a<2^w$ , 所以该方法在大多数计算机上都可以运用。这里假设k的比特数不超过$w$。

根据图11.4，首先将乘以 w比特位的整数 $a$ , 乘积可以表示为 $2w$ 比特位的值 $r_12^w + r_0$ , $r_0$ 为乘积的低 $w$ 比特位，$r_1$ 为乘积的高 $w$ 比特位。然后将 $r_1$ 截断，取 $r_0$ 的高 $l$ 比特位为所求的 $l$ 位散列值。

散列函数：
$$
h(k)=h_a(k)= (ka \text{ }mod \text{ }2^w) \text{ }\ggg\text{ }(w-l)
$$
计算机只需要三种机器指令就能实现这个哈希函数，乘法，减法和逻辑右移。

虽然乘法移位法很快，但不能保证在平均情况下运行良好。下一节中将介绍的全域散列（universal hashing）方法提供了这种保证。在程序开始时，随机选择一个奇数对a进行赋值，这样一个简单的随机变量能够使得乘法移位法在平均情况下运行良好。

### 11.3.2随机哈希法（Random Hashing）

假设有个小可爱知道我们的静态哈希函数是什么，专门选了一些键值会哈希到一个slot, 其查找性能必然是 $\Theta(n)$ 。此时唯一有效的解决办法就是随机选择一个哈希函数，该随机选择与要被存储的关键字完全无关。该方法叫作随机哈希，随即哈希的一个特例叫作全域哈希，能产生可证明的好通过链接处理冲突时的平均性能，无论对手选择何种关键字。

随机哈希法在执行开始时，就从一组精心设计的函数中，随机的选择一个作为散列函数。和随机快排类似，随机化确保了没有哪一种输入会始终导致最坏情形性能。

设$\mathscr{H}$ 为一组有限散列函数簇，它将给定的关键字全域 $U$ 映射到 $\left\{0, 1, ...,m-1\right\}$ 中。 如果对每一对不同的关键字 $k_1,k_2 \in U$ , 满足 $h(k_1) = h(k_2)$ 的哈希函数$h\in \mathscr{H}$的个数最多为 $|\mathscr{H}|/m$ ，这样的一个函数簇被称为全域的（universal。换种说法，如果从 $\mathscr{H}$ 中随机选取一个哈希函数，任意一对不同的关键字 $k_1,k_2 \in U$ 发生冲突的概率不大于 $1/m$ 。

下面基于定理11.2的推论11.3说明了全域哈希达到了如下期望的效果：对手无法通过选择一个操作序列来迫使哈希表一直出现最坏运行时间了。

**推论11.3** 对于一个使用全域哈希法，且具有m个slot的空表，其只要 $\Theta(s)$ 的期望时间来处理任何包含了 $s$ 个插入，搜索和搜查操作的序列，其中该操作序列中包含了 $n = O(m)$ 的插入操作。

证明：插入和删除除了其内部的搜索检查元素操作，每次只需要常量时间，由定理11.2以及全域哈希的限制，每一个搜索操作的期望时间为$O(1)$ 。（定理11.2成立基于碰撞概率为$1/m$ 这一重要假设，而全域哈希把这一假设限制的更严格，即全域哈希下碰撞概率至多为$1/m$ ）于是，根据期望值的线性性质可知，整个n个操作序列的期望时间为 $O(n)$ 。因为每个操作所用时间为$\Omega(1)$ , 所以 $\Theta(n)$ 的界成立。 

### 11.3.3随机哈希的可实现特性

有大量的文献描述过随机哈希函数的性质，以及这些性质与哈希效率的联系，我们这里总结其中最有趣的一些性质。

令 $\mathscr{H}$ 为一个有限散列函数簇，他们中的任何一个函数将给定的关键字全域 $U$ 映射到 $\left\{0, 1, ...,m-1\right\}$ 中，h是从 $\mathscr{H}$ 中均匀随机选取的一个哈希函数，则有如下性质：

1. 如果对 $U$ 中任一关键字 $k$ 和 $\left\{0, 1, ...,m-1\right\}$ 中的slot q ， 满足 $h(k) = q$ 的概率为 $1/m$ , 则簇$\mathscr{H}$ 为均匀的。

2. 如果任何一对不同的关键字 $k_1,k_2 \in U$， 满足 $h(k_1) = h(k_2)$ 的概率至多为 $1/m$ , 则簇$\mathscr{H}$ 为全域的。（与全域定义相似）
3. 如果任何一对不同的关键字 $k_1,k_2 \in U$， 满足 $h(k_1) = h(k_2)$ 的概率至多为 $\epsilon$ , 则簇$\mathscr{H}$ 为 $\epsilon-$全域的。所以一个全域哈希函数簇是 $1/m-$全域的。
4. 如果对任何d个不同的关键字 $k_1,k_2，...,k_d \in U$和 $\left\{0, 1, ...,m-1\right\}$ 中的d个slot $q_1, q_2,...,q_d$， 对于任意$i=1,2,...,d$满足 $h(k_i)=q_i$ 的概率为 $1/m^d$ , 那么 $\mathscr{H}$ 簇是d独立的（d-independent）。

全域散列函数簇特别有趣，因为它们是已证明为有效的支持任何输入数据集的散列表操作的最简单类型。许多其他非常有趣和我们期望的性质，例如上面提到的性质，在全域散列函数簇中都是完全可能的，而且特定的散列表操作实现起来也非常高效。

### 11.3.4设计一个全域哈希函数簇（Designing a universal family of hash functions）

这里会介绍两种设计全域哈希函数簇的方法，第一种会用到点数论，第二种会基于11.3.1节中介绍的乘法移位法并对该法中的一个参数进行随机化。第一个方法数学上证明其全域性质很容易，第二个方法更新颖和高效。

基于数论的全域哈希函数簇

选一个足够大的质数$p$ , 任一关键字 $k$ 满足 $0\leq k\leq p-1$ 。令 $\mathbb{Z}_p$ 代表集合 $\left\{0, 1, ...,p-1\right\}$ , 令 $\mathbb{Z}_p^*$  代表集合  $\left\{1, 2, ...,p-1\right\}$ ，由于关键字域的范围比哈希表槽位数最多，即 $p>m$ 。

对于给定的任一 $a \in \mathbb{Z}_p^*$ 和任一 $b \in \mathbb{Z}_p$ ,定义哈希函数 $h_{ab}$ 为如下的哈希变换：
$$
h_{ab}(k)=((ak+b)\text{ }mod\text{ }p) \text{  }mod\text{  } m \tag{11.3}
$$


对于给定的$p$  和 $m$ , 这样的散列函数簇为：
$$
\mathscr{H}_{pm} = \left\{h_{ab}\text{ }  : \text{ } a \in \mathbb{Z}_p^* \wedge b \in \mathbb{Z}_p\right\} \tag{11.4}
$$
每个散列函数 $h_{ab}$ 将 $\mathbb{Z}_p$ 映射到 $\mathbb{Z}_m$ 。这个函数簇有个很好的性质，那就是哈希表大小， $m$ , 可以为任一值，不用是质数，簇中有 $p(p-1)$ 个哈希表。

**定理11.4** 

由公式11.3和11.4定义的$\mathscr{H}_{pm}$ 簇是全域的。

**证明**：设两个不同的 $k_1,k_2 \in \mathbb{Z}_p$ (其中$k_1 \neq k_2$)，对于给定的哈希函数 $h_{ab}$ , 有
$$
\begin{aligned}
&r_1 = (ak_1+b) \text{  }mod\text{  }p \\
&r_2 = (ak_2+b) \text{  }mod\text{  }p
\end{aligned}
$$
其中 $r_1 \neq r_2$ 。为什么？我们有 $r_1-r_2=a(k_1-k_2)\text{ } (mod \text{ }p)$ 。由于 $a$ 和 $(k_1-k_2)$ 模 $p$ 后都非零，根据第908的数论定理31.6，$a(k_1-k_2) \text{ }mod\text{ }p\neq0$ 。所以  $r_1 \neq r_2$ 。

因此对任一 $h_{ab}$ ，不同的输入 $k_1,k_2$ 会映射到互不相同的 $r_1,r_2$。所以在$mod \text{ }p$ 这一步， $k_1,k_2$ 的映射值是没有冲突的。此外，我们可以根据 $(r_1,r_2)$ 解出 $p(p-1)$ 对中的一对 $(a,b)$ :
$$
\begin{aligned}
&a = ((r_1-r_2)((k_1-k_2)^{-1} \text{  }mod \text{  }p))\text{  }mod\text{  }p \\
&b = (r_1-ak_1) \text{  }mod \text{  }p
\end{aligned}
$$
其中 $(k_1-k_2)^{-1} \text{  }mod \text{  }p$ 表明唯一·乘法逆元模p。对于任一$r_1$ ，有且仅有 $p-1$ 个可能的值 $r_2$ 使得 $r_1 \neq r_2$ ，所以总共有 $p(p-1)$ 对 $(r_1, r_2)$ ，正好与 $p(p-1)$ 对 $(a,b)$ 形成一对一映射（可数学上证明）。因此，对于给定的 $k_1,k_2$ ，如果从 $\mathbb{Z}_p \times \mathbb{Z}_p^*$ 中均匀随机选择 $(a, b)$ , 则 $(r_1, r_2)$ 也等概率的为互不相同的数值对。

因此，两个不同的关键字 $k_1,k_2$ 最终碰撞的概率等于 $r_1 \equiv r_2(\text{  }mod\text{  }m)$ 的概率。对任一 $r_1$ 值，$r_2$ 只能是剩余的 $p-1$ 个值中的其中一个 ，其中满足  $r_1 \equiv r_2(\text{  }mod\text{  }m)$ 的数量最多为：$\lceil \frac{p}{m}\rceil-1 \leq \frac{p+m-1}{m}-1 = \frac{p-1}{m}$ (这里书中计算是正确的), 则 $r_1 \equiv r_2(\text{  }mod\text{  }m)$ 的概率至多为 $\frac{p-1}{m}/(p-1)=1/m$ 。

因此，对于任何一对不同的关键字 $k_1,k_2 \in \mathbb{Z}_p$， 满足 $h_{ab}(k_1) = h_{ab}(k_2)$ 的概率至多为 $1/m$ , 则簇$\mathscr{H}_{pm}$ 为全域的。很精彩的证明!!

基于乘法位移法的$2/m-$全域的哈希函数簇 

建议在实践中使用如下基于乘法移位法的散列函数簇，它不仅非常高效而且是$2/m-$全域的。
$$
\mathscr{H} = \left\{h_{a}\text{ }  : \text{ a为奇数}  \wedge 1\leq a<m\text{ }\wedge\text{ }h_a\text{由等式(11.2)定义}\right\} \tag{11.5}
$$
**定理11.5**

由等式(11.5)定义的散列函数簇$\mathscr{H}$ 是 $2/m$ - 全域的。 

以下为本人的证明。

证明：

也就是说，任意两个不同关键字发生冲突的概率最多为 $2/m$ 。在许多实践场景中，与全域散列函数相比，$2/m$-全域的哈希函数对计算散列函数的速度的提升大大补偿了两个不同关键字发生冲突概率上界的提升。