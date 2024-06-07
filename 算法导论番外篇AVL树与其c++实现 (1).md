# 算法导论番外：AVL树与其c++实现

作者：Claude Du

本文的动机：MIT算法公开课AVL树章节也听了（有趣且有启发性，但内容有点少），之前算法导论第12章BST与第13章红黑树的笔记也都写了，似乎没有太多理由放过AVL树，何况算法导论13章思考题13-3也要求读者论证AVL树的性质和实现AVL树的插入功能。而且对于我个人而言，似乎也是不错的算法锻炼与写作挑战机会。

PS：

​		MIT算法公开课AVL树链接：

​		12章BST笔记与其数据结构c++实现的链接：https://3ms.huawei.com/km/blogs/details/15350515?l=zh-cn

​		花10天写的13章红黑树笔记与其数据结构c++实现的链接：https://3ms.huawei.com/km/blogs/details/15343190?l=zh-cn

我将尝试模仿算法导论13章红黑树的叙述结构与范式来写此文，其中本文的c++实现有部分参考邓教授的《数据结构（c++语言版）》第三版（有参考思路，但是代码实现完全不一样），这是本人的第一次尝试，如果有错误和建议，还请各位指出，感谢！

PS: 哈哈哈，如果未来算法导论的后续版本加了AVL树，到时还可以对比本文和算导AVL树章节的相似度，想想就High到不行！

![](./%E7%9C%9F%E6%98%AFHigh%E5%88%B0%E4%B8%8D%E8%A1%8C.gif)

## 1 AVL树的性质

AVL树如下图所示，是一种高度平衡的二叉搜索树，**其核心的高度平衡性质为**：**对每一个节点x, 对x的左子树与右子树的高度差至多为1。** 

插入AVL树图片

那么要实现一棵AVL树，需要在每个节点维护一个额外的属性：x.height为节点x的高度。在本章节中，我们约定仅含单个节点的二叉搜索树高度为0，空树高度为-1。

AVL树的节点TreeNode的c++实现框架依然沿用12章笔记12.1节，代码如下：

```c++
#
#define stature(node) ((node) ? (node)->height_ : -1)
template<typename T> struct TreeNode {
    T data_;
    TreeNode<T>* parent_;
    TreeNode<T>* lc_;
    TreeNode<T>* rc_;
    int height_;
    TreeNode() : parent_(nullptr), lc_(nullptr), rc_(nullptr), height_(0) {}
    TreeNode(T data, TreeNode<T>* parent = nullptr, TreeNode<T>* lc = nullptr,
             TreeNode<T>* rc = nullptr, int h = 0) : 
             data_(data), parent_(parent), lc_(lc), rc_(rc), height_(0) {}
    TreeNode<T>* insertAsLC(T const& lcVal);
    TreeNode<T>* insertAsRC(T const& lcVal);
    // Traversal Method
    template <typename VST> void travelLevel(VST& visit) { TravelLevel(this, visit); }
    template <typename VST> void travelPre(VST& visit) { TravelPreIterative(this, visit); }
    template <typename VST> void travelIn(VST& visit) { TravelInIterative(this, visit); }
    template <typename VST> void travelPost(VST& visit) { TravelPostRecursive(this, visit); }
};
```



AVL树可以实现近乎理想的平衡。在渐进意义下, AVL树可始终将高度控制在 $O(lgn)$ 以内（引理1证明在下文），从而保证每次查找，插入和删除操作，均可在 $O(lgn)$ 的时间内完成。

引理1：一棵有n个节点的AVL树高度为$O(lgn)$ 。

引理1的证明在MIT算法公开课中有提到，感兴趣的可以去看一看。

以下为证明的文字内容：



AVL树是对节点高度敏感的，所以我们要经常更新二叉树的高度，每当某一节点的孩子或后代有所增减，其高度都有必要及时更新。然而实际上，节点自身很难发现后代的变化，因此我们不妨反过来用另一处理策略：如果有节点加入，离开AVL树或位置变更，则更新所有祖先的高度。其c++实现如下：

```c++

template <typename T> int BST<T>::UpdateHeight(TreeNode<T>* node) {
    node->height_ = 1 + std::max(stature(node->lc_), stature(node->rc_));
    return node->height_;
}
template <typename T> void BST<T>::UpdateHeightAbove(TreeNode<T>* node) {
    while (node) {
        UpdateHeight(node);
        node = node->parent_;
    }
}
```

在常规二叉搜索树的动态操作（插入与删除节点）可能会破坏AVL树的平衡性，这破坏情形一定要被监测到，我们会使用如下的函数进行平衡监测：

```c++
inline bool Balanced(TreeNode<T>* x) { return stature(x->lc_) == stature(x->rc_); }
inline int BalFac(TreeNode<T>* x) {
    return (stature(x->lc_) - stature(x->rc_));
}
inline bool AvlBalanced(TreeNode<T>* x) {
    return (-2 < BalFac(x)) && (BalFac(x) < 2);
}
```

AVL树的c++实现框架如下:

```c++

```

可能大家要问为何不直接继承12章的二叉搜索树，主要是想为了避免读者反复去查阅12章笔记的内容。

## 2 旋转操作与平衡过程

其实和13.2节的旋转基本一样，实现细节唯一的不同点就是旋转完成后要高度发生变化的节点进行高度更新。

12章的二叉搜索树操作TreeInsert和TreeDelete在含有n个关键字的二叉搜索树上运行花费时间为$O(lgn)$ 。由于这两个操作对树做了修改，其操作结果可能违反13.1节中列出的红黑性质，为了维护这些性质，必须要改变这些节点的颜色以及指针结构。其中指针结构的修改是通过旋转来完成的。

旋转是一种能局部保持二叉搜索树性质的搜索树局部操作。下图（二叉搜索树中虚线箭头为child指向parent, 实线箭头为parent指向child）给出了两种旋转，左旋(LeftRotate)和右旋(RightRotate)。图中节点x上做左旋时，假设x的right child y 不是tNil。



LeftRotate的c++代码实现如下：

```c++
template <typename T>
void AVLTree<T>::LeftRotate(RBNode<T>* x) {
    RBNode<T>* y = x->rc_;
    if (y == tNil) return;
    x->rc_ = y->lc_;
    if (y->lc_ != tNil) y->lc_->parent_ = x;
    y->parent_ = x->parent_;
    if (x->parent_ == tNil) root_ = y;
    else if (x == x->parent_->lc_) x->parent_->lc_ = y;
    else x->parent_->rc_ = y;
    y->lc_ = x;
    x->parent_ = y;  
    UpdateHeightAbove(x);
}
```

LeftRotate程序实现左旋功能的详细流程解析如下图所示：

RightRotate的代码实现和LeftRotate的代码实现是对称的，把 rc和 lc 对调一下即可。

RightRotate的c++代码实现如下（Exercise 13.2-1）：

```c++
template <typename T>
void AVLTree<T>::RightRotate(RBNode<T>* x) {
    RBNode<T>* y = x->lc_;
    if (y == tNil) return;
    x->lc_ = y->rc_;
    if (y->rc_ != tNil) y->rc_->parent_ = x;
    y->parent_ = x->parent_;
    if (x->parent_ == tNil) root_ = y;
    else if (x == x->parent_->lc_) x->parent_->lc_ = y;
    else x->parent_->rc_ = y;
    y->rc_ = x;
    x->parent_ = y;   
    UpdateHeightAbove(x);
}
```

LeftRotate和RightRotate都在 $O(1)$ 的时间内运行完成。在旋转操作中只有指针改变，其他所有属性都保持不变。

在不同的书中，LeftRotate和RightRotate有不同的叫法，



3插入

在AVL树中插入一个值为val的节点newNode，首先以BST（二叉搜索树）的顺序把该节点放在适当的位置上，此时和12章二叉树的插入操作一模一样，该操作的c++实现如下：

```c++
template <typename T> TreeNode<T>* AVLTree<T>::TreeInsert(T const& val) {
    TreeNode<T>* targetPosiParent = nullptr;
    TreeNode<T>* curPosi = root_;
    while (curPosi != nullptr) {
        targetPosiParent = curPosi;
        if (val < curPosi->data_) curPosi = curPosi->lc_;
        else curPosi = curPosi->rc_;
    }
    TreeNode<T>* newNode = new TreeNode<T>(val, targetPosiParent);
    ++size_;
    // if tree is empty
    if (targetPosiParent == nullptr) {
        root_ = newNode;   
        return newNode;
    }
    if (val < targetPosiParent->data_) {
       targetPosiParent->lc_ = newNode; 
    } else targetPosiParent->rc_ = newNode;
    UpdateHeightAbove(newNode);
    return newNode;
}
```

但该操作完成后，这棵树可能就不再是高度平衡的。具体来说，某些节点的左子树和右子树的高度差可能变成2。我们要用重平衡(Rebalance)过程来维护高度平衡的性质。

**重平衡**

如上描述，在原二叉搜索树完成的基本插入过程后，从newNode沿着Parent指针往上找到第一个高度失衡节点firstUnbalancedNode，我们令p为firstUnbalanced的高度相对高的孩子节点，v为p的高度相对高的孩子节点。

这里为了后续编程的便利，我们

4删除