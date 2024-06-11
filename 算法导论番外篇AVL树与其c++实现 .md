# 算法导论番外篇：AVL树与其c++实现

作者：Claude Du

没错，这竟然不是算法导论章节笔记了，惊不惊喜，意不意外？

本文的动机：MIT算法公开课AVL树章节已听（有趣且有启发性，但内容有点少），之前算法导论第12章BST与第13章红黑树的笔记也都写过了，似乎没有太多理由放过AVL树，何况算法导论13章思考题13-3也要求读者论证AVL树的性质和实现AVL树的插入功能。而且对于我个人而言，似乎也是不错的算法锻炼与写作挑战机会。

PS：

​		MIT算法AVL树公开课链接：https://www.youtube.com/watch?v=FNeL18KsWPc

​		12章BST笔记与其数据结构c++实现的链接：

​		花了15天才写完的的13章红黑树笔记与其数据结构c++实现的链接：（有兴趣的观众佬爷们看一看吧，顺便评论一下）

我将尝试模仿算法导论13章红黑树的叙述结构与范式来写此文，其中本文的c++实现有部分参考邓教授的《数据结构（c++语言版）》第三版（有参考其中的思路，但是代码实现并不一样），这是本人的第一次尝试，如果有错误和建议，还请各位指出，感谢！



## 1 AVL树的性质

AVL树如下图所示，是一种高度平衡的二叉搜索树，**其核心的高度平衡性质为**：**对每一个节点x, 对x的左子树与右子树的高度差绝对值至多为1。** AVL树得名于它的发明者，两位苏联人，G. M. Adelson-Velsky和和E. M. Landis，他们在1962年的论文《An algorithm for the organization of information》中公开了这一数据结构。



要实现一棵AVL树，需要在每个节点维护一个额外的属性：x.height为节点x的高度。在本章节中，我们约定仅含单个节点的二叉搜索树高度为0，空树高度为-1。

AVL树的节点TreeNode的c++实现框架依然沿用12章笔记12.1节，c++代码实现如下：

```c++

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

上面代码中遍历函数的实现在本章节中没那么重要，便不再赘述，具体实现参考12章BST笔记中附录部分。

AVL树可以实现近乎理想的平衡。在渐进意义下, AVL树可始终将高度控制在 $O(lgn)$ 以内（引理1证明在下文），从而保证每次查找，插入和删除操作，均可在 $O(lgn)$ 的时间内完成。

**引理1**：一棵有n个节点的AVL树高度为$O(lgn)$ 。

引理1的证明在MIT算法公开课中第18分钟有提到，感兴趣的可以去看一看。

文字版证明如下：

先看看一个高度为h的AVL树至少包含 多少个节点。令 $N(h)$ 表示高度为h的AVL树的最少节点数，此棵Avl树的最高的孩子的高度必然为h-1。为了让此棵AVL树的节点数最少，其另一个孩子的高度只能是h-2, 因此我们轻松求得了 $N(h)$ 的递推表达表达式：

$$N(h) = N(h-1) + N(h-2) + 1$$

对上式两边同时+1，并令 $F(h) = N(h) + 1$。我们得到：

$$F(h) = F(h-1) + F(h-2) $$

这是标准的斐波那契数列的递推表达式，当 $h=0$ 时，$N(0) = 1$ 以及 $F(0) = 2 =Fib(3)$ 。当 $h=1$ 时，$N(1) = 2$ 以及 $F(1) = 3 = Fib(4)$ 。猜测 $F(n) = Fib(n+3)$对于h-1和h-2, 我们假设$F(h-1) = Fib(h+2)$, $F(h-2) = Fib(h+1)$, 那么 $F(h) = Fib(h+2) +Fib(h+1) = Fib(h+3)$， 猜想成立。

则高度为h的AVL树至少包含$Fib(h+3)-1$个节点。则 有n个节点的AVL树高度为$O(lgn)$ 显然成立。

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

在常规二叉搜索树的动态操作（插入与删除节点）可能会破坏AVL树的平衡性，这破坏情形一定要被监测到，我们会使用如下的函数进行平衡监测，比如节点平衡因子（Balance Factor, BalFac）：

```c++
template <typename T>
inline bool Balanced(const TreeNode<T>* x) { return stature(x->lc_) == stature(x->rc_); }
template <typename T>
inline int BalFac(const TreeNode<T>* x) {
    return (stature(x->lc_) - stature(x->rc_));
}
template <typename T>
inline bool AvlBalanced(const TreeNode<T>* x) {
    return (-2 < BalFac(x)) && (BalFac(x) < 2);
}
```

AVL树的c++实现框架如下，其中所有查找操作和12章笔记中12.2节完全一致，其代码实现直接看附录，不同的部分只有写操作——插入与删除:

```c++
// author: Claude Du
template <typename T> class AVLTree {
protected:
    int size_; TreeNode<T>* root_; // size_ means # of nodes
    virtual int UpdateHeight(TreeNode<T>* node);
    void UpdateHeightAbove(TreeNode<T>* node);
    void Display(TreeNode<T>* cur, int depth = 0);
    void Transplant(TreeNode<T>* toBeSuAVLTreeituted, TreeNode<T>* vertex);
public:
    AVLTree() : size_(0), root_(nullptr) {}    
    ~AVLTree() { 
        if (size_ > 0) {
            std::cout << "Hi, my job is done. " << "\n";
            std::cout << Remove(root_);
        }
    }
    int size() const { return size_; }
    int height() const { return stature(root_); }
    bool empty() const { return root_ == nullptr; }
    TreeNode<T>* root() const { return root_; }
    TreeNode<T>* TreeMinimum() const {
        if (root_ == nullptr) return root_; 
        return TreeMinimum(root_); 
    }
    // read operations:
    TreeNode<T>* TreeMinimum(TreeNode<T>* cur) const;
    TreeNode<T>* TreeMaximum() const {
        if (root_ == nullptr) return root_; 
        return TreeMaximum(root_); 
    }
    TreeNode<T>* TreeMaximum(TreeNode<T>* cur) const;
    TreeNode<T>* TreeSuccessor(TreeNode<T>* cur) const;
    TreeNode<T>* TreePredecessor(TreeNode<T>* cur) const;
    // search
    TreeNode<T>*& TreeSearch(T const& keyVal);
    TreeNode<T>*& TreeSearchRecursive(TreeNode<T>*& cur, T const& keyVal);
    TreeNode<T>* TreeSearchIterative(TreeNode<T>* cur, T const& keyVal);
    void LeftRotate(TreeNode<T>* x);
    void RightRotate(TreeNode<T>* x);
    // insertion
    TreeNode<T>* TreeInsert(T const& val);
    // deletion
    bool TreeDeletion(T const& val);
    void TreeDeletion(TreeNode<T>*& cur);
    void TreeDeletion2(TreeNode<T>* cur);
    int Remove(TreeNode<T>* x);
    void ReBalance(TreeNode<T>* x);
    void PosiBalance(TreeNode<T>* x);
    // Traversal
    template <typename VST>
    void travLevel(VST& visit) { if (root_) root_->travLevel(visit); }
    template <typename VST>
    void travelPre(VST& visit) { if (root_) root_->travelPre(visit); }
    template <typename VST>
    void travelIn(VST& visit) { if (root_) root_->travelIn(visit); }
    template <typename VST>
    void travelPost(VST& visit) { if (root_) root_->travelPost(visit); }
    
    void Display();
}; 
```

可能大家要问为何实现不直接继承12章的二叉搜索树，主要是想为了避免读者反复去查阅12章笔记的内容。

## 2 旋转操作

其实和13.2节的旋转基本完全一样，实现细节唯一的不同点就是旋转完成后，要对高度发生变化的节点进行高度更新。

12章的二叉搜索树操作TreeInsert和TreeDelete在含有n个关键字的二叉搜索树上运行花费时间为$O(lgn)$ 。由于这两个操作对树做了修改，其操作结果可能违反1节中出现的高度平衡性质，为了维护该性质，必须要改变这些节点的指针结构。其中指针结构的修改是通过旋转来完成的。

旋转是一种能局部保持二叉搜索树性质的搜索树局部操作。下图（二叉搜索树中虚线箭头为child指向parent, 实线箭头为parent指向child）给出了两种旋转，左旋(LeftRotate)和右旋(RightRotate)。图中节点x上做左旋时，假设x的right child y 不是tNil。

LeftRotate的c++代码实现如下：

```c++
// author: Claude Du
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
// author: Claude Du
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

我们可以用12章的已实现的display功能查看左旋和右旋对树结构的更改，效果如下：

```
          L----[3] - (0)
     L----[5] - (1)
          R----[7] - (0)
[8] - (3)
               L----[9] - (0)
          L----[10] - (1)
               R----[11] - (0)
     R----[12] - (2)
          R----[13] - (1)
               R----[16] - (0)

Here is the left rotation of node 8

               L----[3] - (0)
          L----[5] - (1)
               R----[7] - (0)
     L----[8] - (2)
               L----[9] - (0)
          R----[10] - (1)
               R----[11] - (0)
[12] - (3)
     R----[13] - (1)
          R----[16] - (0)

Here is the right rotation of node 12

          L----[3] - (0)
     L----[5] - (1)
          R----[7] - (0)
[8] - (3)
               L----[9] - (0)
          L----[10] - (1)
               R----[11] - (0)
     R----[12] - (2)
          R----[13] - (1)
               R----[16] - (0)
```





## 3 插入

在AVL树中插入一个值为val的节点newNode，首先以BST（二叉搜索树）的顺序把该节点放在适当的位置上，此时和12章二叉树的插入操作一模一样，该操作的c++实现如下(第2~19行)：

```c++
// author: Claude Du
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
    // to maintain the height Balance
    InsertRebalance(newNode);
    return newNode;
}
```

但该操作完成后，这棵树可能就不再是高度平衡的。具体来说，某些节点的左子树和右子树的高度差可能变成2。我们要用重平衡(Rebalance)过程来维护高度平衡的性质。

**插入操作的重平衡**

如上描述，在原二叉搜索树完成的基本插入过程后，从newNode沿着Parent指针往上找到第一个高度失衡节点firstUnbalancedNode，我们令p为firstUnbalanced的高度相对高的孩子节点，v为p的高度相对高的孩子节点。（当某节点x的两边高度一样时，选择与x同侧的节点，比如x是xparent的leftchild，选x的leftchild）。

PS: 这里的第一个高度失衡节点firstUnbalancedNode的高度不低于x的祖父。

节点x的高度相对更高的孩子节点TallerChild的c++实现：

```c++
// author: Claude Du
template <typename T>
inline TreeNode<T>* TallerChild(const TreeNode<T>* x) {
    if (stature(x->lc_) > stature(x->rc_)) return x->lc_;
    if (stature(x->lc_) < stature(x->rc_)) return x->rc_;
    return (x-> parent_ == nullptr) || (x == x->parent_->lc_) ? x->lc_ : x->rc_;   
}
```

这里我先放出插入重平衡的c++代码实现，该实现结合后面的场景讨论一起看，应该会更容易理解，不建议直接硬看：

```c++
// author: Claude Du
template <typename T> void AVLTree<T>::InsertRebalance(TreeNode<T>* x) {
    TreeNode<T>* g = x->parent_;
    while (g != nullptr) {
        if (!AvlBalanced(g)) {
            PosiBalance(g);
            break;
        } else g = g->parent_;
    }
}
template <typename T> void AVLTree<T>::PosiBalance(TreeNode<T>* g) {
    TreeNode<T>* p = TallerChild(g);
    TreeNode<T>* v = TallerChild(p);
    if ( p == g->rc_) {
        if (v == p->rc_) { // case 1: rightchild-rightchild
            LeftRotate(g);  
        } else { // case 2: rightchild-leftchild
            RightRotate(p);
            LeftRotate(g);
        }
    } else { // same as line 14~19, but with left and right exchanged
        if (v == p->lc_) {
            RighttRotate(g);  
        } else {
            LeftRotate(p);
            RightRotate(g);
        }        
    }
}

```

接下来我们要通过节点g, v, p的相对位置关系分不同情形来实现局部子树重平衡的操作，我称为**位置重平衡（PosiBalance）**，对应如上代码第10~28行。这里的我们实际上需要考虑4种g, v, p相对位置情形，但是前2种和后两种是对称的，我们也只用验证前2种情形即可。

**case 1 Right-Right**: p是g的rightChild, v是p的rightChild。如下图所示：



这种情形下，我们直接左旋g，g节点恢复平衡，同样g以上所有的祖先（包括左旋后的p）的平衡因子也会全部恢复平衡，此时整棵树的高度平衡性质得到恢复。对应代码为上面代码块中第15行

**case 3 Left-Left**是case1的对称情形，对应代码为上面代码块中第22行，无需再讨论。

**case2 Right-Left**: p是g的rightChild, v是p的leftChild。如下图所示：



这种情形下，我们可以先右旋p节点，右旋后，case2几乎变成了case1（区别仅仅是v和p的位置关系对调了而已）, 那么和case 1一样，再左旋g即可。对应代码为上面代码块中第17~18行。

**case 4 Left-Right**是case2的对称情形，对应代码为上面代码块中第24~25行，无需再讨论。

case 1与case 3是单次旋转，case 2与 case4是双次旋转，进过这一调整，不仅以g为根的局部子树恢复平衡，该局部子树的高度也复原，这也意味着g以上所有的祖先的平衡因子也会全部恢复平衡，此时整棵树的高度平衡性质得到恢复。--换言之，在AVL树插入新节点后，仅需不超过2次旋转，即可整树恢复平衡，运行耗时也仅为 $O(lgn)$ 。

**位置重平衡（PosiBalance）的操作也会在删除操作中得以继续使用。**

AVLTree的插入操作代码实现对比红黑树的插入操作，好像确实要简单可爱一些呀！

**AVLTree的插入验证**

简易的验证用例如下：

```c++
// author: Claude Du
#include "TreeNode.h"
#include "AVLTree.h"
int main()
{

    
    // Test AVLTree insertion operation
    AVLTree<int> bst;
    bst.TreeInsert(11);
    
    bst.TreeInsert(7);
    TreeNode<int>* node8 = bst.TreeInsert(8);
    bst.TreeInsert(9);
    bst.Display();
    std::cout << "A node with val 10 will be inserted. which satisfy case 2, right-left." << "\n";
    bst.TreeInsert(10);
    bst.Display();
    std::cout << "A node with val 12 will be inserted. " << "\n";
    TreeNode<int>* node12 = bst.TreeInsert(12);
    
    bst.Display();
    std::cout << "A node with val 13 will be inserted. which satisfy case 1, right-right." << "\n";
    bst.TreeInsert(13);
    bst.Display();

}

```

验证效果如下，高度平衡性质得以维护：

```
     L----[7] - (0)     
[8] - (2)
          L----[9] - (0)
     R----[11] - (1)    

A node with val 10 will be inserted. which satisfy case 2, right-left.

     L----[7] - (0)
[8] - (2)
          L----[9] - (0)
     R----[10] - (1)
          R----[11] - (0)

A node with val 12 will be inserted.

          L----[7] - (0)
     L----[8] - (1)
          R----[9] - (0)
[10] - (2)
     R----[11] - (1)
          R----[12] - (0)

A node with val 13 will be inserted. which satisfy case 1, right-right.

          L----[7] - (0)
     L----[8] - (1)
          R----[9] - (0)
[10] - (2)
          L----[11] - (0)
     R----[12] - (1)
          R----[13] - (0)
Hi, my job is done.
7
```

## 4 删除

和插入一个节点在流程上有一些相似之处的：我们还是先像二叉搜索树的删除操作一样，此时和12章二叉树的删除操作一模一样，该操作的c++实现如下(第2~20行)：

```c++
// author: Claude Du
template <typename T> void AVLTree<T>::TreeDeletion2(TreeNode<T>* z) {
    TreeNode<T>* target = z->parent_; // find the lowest node whose height may need to updated
    if (z->lc_ == nullptr) {
        Transplant(z, z->rc_); // replace z by its right child
        
    } else if (z->rc_ == nullptr) {
        Transplant(z, z->lc_); // replace z by its left child    
    } else {
        TreeNode<T>* y = TreeMinimum(z->rc_);
        if (y->parent_ != z) { // is y farther down the tree?
            Transplant(y, y->rc_); // replace y by its right child
            y->rc_ = z->rc_; // z's right child becomes
            y->rc_->parent_ = y; // y's right child
        }
        Transplant(z, y); // replace z by its successor y
        y->lc_ = z->lc_; // and give z's left child to y
        y->lc_->parent_ = y; // which had no left child
        target = (y->rc_ == nullptr) ? y : y-> rc_;     
    }
    UpdateHeightAbove(target);
    DeleteRebalance(target);
    --size_;
}
```

但是正常的删除完该节点z后，Avl树原先的高度平衡性质可能会被破坏，我们需要删除重平衡（DeleteRebalance，第21行）的操作来恢复该二叉搜索树的高度平衡。

### 删除的重平衡

如上描述，在原二叉搜索树完成的基本插入过程后，从target节点（第一个高度可能发生变化的节点）沿着Parent指针往上找, 经过 $O(lgn)$ 到第一个高度失衡节点g，作为失衡节点的g，在一开始不包含z的一侧必有非空孩子p, 其高度会相对于一开始包含z的那侧一定会更高(不然删除了z后，g不会不平衡)，其高度至少为1。和之前插入的重平衡一样，我们令v为p的高度相对高的孩子节点。（当某节点x的两边高度一样时，选择与x同侧的节点，比如x是xparent的leftchild，选x的leftchild）。

PS: 当完全按照二叉搜索树的方式删除了一个节点后，第一个高度失衡的节点有可能就是z的parent, 也可能是z的后继的parent。

与之前插入的重平衡一样，我们还是要通过g, v, p的相对位置关系来讨论如何实现局部子树的重平衡，内容和插入的重平衡的4个情形中的操作几乎一模一样，所以我这里不会再重复叙述。我们依然要PosiBalance来修复局部子树的高度平衡，与插入的重平衡不同之处在于删除过程中调用PosiBalance后，局部子树的高度平衡性质得到修复的同时g节点的高度变低，可能会引起g的某一更高祖先的高度失衡，这种现象称为**失衡传播**，还需要在一次调用PosiBalance来修复该祖先节点，直到最终确认根节点高度平衡为止。而插入的重平衡不会有这种现象，插入的重平衡调用PosiBalance后，不仅恢复了子树的平衡性还恢复了子树的高度。

删除的重平衡操作的c++实现如下：

```c++
// author: Claude Du
template <typename T> void DeleteRebalance(TreeNode<T>* target) {
    TreeNode<T>* g = target;
    while (g != nullptr) {
        if (!AvlBalanced(g)) {
            PosiBalance(g);
        }
        g = g->parent_;
    }
}
```

要注意的是失衡的传播方向必然自底而上，不会影响到后代节点。

与插入操作比较，删除操作可能在重平衡方面更加耗时，不过要做位置重平衡的节点都在树中一条简单路径上，所以耗时依然为 $O(lgn)$ 。

**AVLTree的删除验证**

简易的验证用例如下：

```c++
// author: Claude Du
#include "TreeNode.h"
#include "AVLTree.h"
int main()
{   
    // Test BST
    AVLTree<int> bst;
    bst.TreeInsert(11);    
    bst.TreeInsert(8);
    bst.TreeInsert(9);
    bst.TreeInsert(10);
    bst.TreeInsert(12);
    bst.TreeInsert(13);
    bst.TreeInsert(5);
    bst.TreeInsert(3);
    bst.TreeInsert(1);
    bst.TreeInsert(12);
    bst.TreeInsert(0);
    bst.TreeInsert(4);
    bst.TreeInsert(7);   
    bst.TreeInsert(6);

    std::cout << "The size of Avl tree is " << bst.size() << "\n";
    bst.Display();
    std::cout << "A node with val 0 will be deleteded, so that case 3, left-left, will be satisfied at the first iteration." 	 << "\n";
    std::cout << "Then it will satisfy case 2, Right-left, at the second iteration." << "\n";
    bst.TreeDeletion(0);
    std::cout << "The size of Avl tree is " << bst.size() << "\n";
    bst.Display();
}
```

验证效果如下，高度平衡性质得以维护：

```
The size of Avl tree is 14

          L----[0] - (0)
     L----[1] - (2)
          R----[3] - (1)
               R----[4] - (0)
[5] - (4)
                    L----[6] - (0)
               L----[7] - (1)
                    R----[8] - (0)
          L----[9] - (2)
               R----[10] - (0)
     R----[11] - (3)
               L----[12] - (0)
          R----[12] - (1)
               R----[13] - (0)

A node with val 0 will be deleteded, so that case 3, left-left, will be satisfied at the first iteration.
Then it will satisfy case 2, Right-left, at the second iteration.
The size of Avl tree is 13

               L----[1] - (0)
          L----[3] - (1)
               R----[4] - (0)
     L----[5] - (2)
               L----[6] - (0)
          R----[7] - (1)
               R----[8] - (0)
[9] - (3)
          L----[10] - (0)
     R----[11] - (2)
               L----[12] - (0)
          R----[12] - (1)
               R----[13] - (0)

Hi, my job is done.
13
```

哈哈哈，本篇到此结束。

如果未来算法导论的后续版本加了AVL树，到时还可以对比本文和算导AVL树章节的相似度，想想就High到不行！

![](./%E7%9C%9F%E6%98%AFHigh%E5%88%B0%E4%B8%8D%E8%A1%8C.gif)

## 5 附录

AVL树整体c++实现如下

AVLTree.h

```c++
#ifndef AVLTree_h
#define AVLTree_h
// author: Claude Du
#include "TreeNode.h" 
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <algorithm>
#include <cmath>
using std::vector;
using std::queue;
using std::stack;
template <typename T>
inline bool Balanced(const TreeNode<T>* x) { return stature(x->lc_) == stature(x->rc_); }
template <typename T>
inline int BalFac(const TreeNode<T>* x) {
    return (stature(x->lc_) - stature(x->rc_));
}
template <typename T>
inline bool AvlBalanced(const TreeNode<T>* x) {
    return (-2 < BalFac(x)) && (BalFac(x) < 2);
}

template <typename T>
inline TreeNode<T>* TallerChild(const TreeNode<T>* x) {
    if (stature(x->lc_) > stature(x->rc_)) return x->lc_;
    if (stature(x->lc_) < stature(x->rc_)) return x->rc_;
    return (x-> parent_ == nullptr) || (x == x->parent_->lc_) ? x->lc_ : x->rc_;   
}
template <typename T> class AVLTree {
protected:
    int size_; TreeNode<T>* root_; // size_ means # of nodes
    virtual int UpdateHeight(TreeNode<T>* node);
    void UpdateHeightAbove(TreeNode<T>* node);
    void Display(TreeNode<T>* cur, int depth = 0);
    void Transplant(TreeNode<T>* toBeSuAVLTreeituted, TreeNode<T>* vertex);
public:
    AVLTree() : size_(0), root_(nullptr) {}    
    ~AVLTree() { 
        if (size_ > 0) {
            std::cout << "Hi, my job is done. " << "\n";
            std::cout << Remove(root_);
        }
    }
    int size() const { return size_; }
    int height() const { return stature(root_); }
    bool empty() const { return root_ == nullptr; }
    TreeNode<T>* root() const { return root_; }
    TreeNode<T>* TreeMinimum() const {
        if (root_ == nullptr) return root_; 
        return TreeMinimum(root_); 
    }
    // read operations:
    TreeNode<T>* TreeMinimum(TreeNode<T>* cur) const;
    TreeNode<T>* TreeMaximum() const {
        if (root_ == nullptr) return root_; 
        return TreeMaximum(root_); 
    }
    TreeNode<T>* TreeMaximum(TreeNode<T>* cur) const;
    TreeNode<T>* TreeSuccessor(TreeNode<T>* cur) const;
    TreeNode<T>* TreePredecessor(TreeNode<T>* cur) const;
    // search
    TreeNode<T>*& TreeSearch(T const& keyVal);
    TreeNode<T>*& TreeSearchRecursive(TreeNode<T>*& cur, T const& keyVal);
    TreeNode<T>* TreeSearchIterative(TreeNode<T>* cur, T const& keyVal);
    void LeftRotate(TreeNode<T>* x);
    void RightRotate(TreeNode<T>* x);
    // insertion
    TreeNode<T>* TreeInsert(T const& val);
    void InsertRebalance(TreeNode<T>* x);
    // deletion
    bool TreeDeletion(T const& val);
    void TreeDeletion(TreeNode<T>*& cur);
    void TreeDeletion2(TreeNode<T>* cur);
    void DeleteRebalance(TreeNode<T>* z);
    int Remove(TreeNode<T>* x);
    
    void PosiBalance(TreeNode<T>* x);
    // Traversal
    template <typename VST>
    void travLevel(VST& visit) { if (root_) root_->travLevel(visit); }
    template <typename VST>
    void travelPre(VST& visit) { if (root_) root_->travelPre(visit); }
    template <typename VST>
    void travelIn(VST& visit) { if (root_) root_->travelIn(visit); }
    template <typename VST>
    void travelPost(VST& visit) { if (root_) root_->travelPost(visit); }
    
    void Display();
}; 

template <typename T> int AVLTree<T>::UpdateHeight(TreeNode<T>* node) {
    node->height_ = 1 + std::max(stature(node->lc_), stature(node->rc_));
    return node->height_;
}
template <typename T> void AVLTree<T>::UpdateHeightAbove(TreeNode<T>* node) {
    while (node) {
        UpdateHeight(node);
        node = node->parent_;
    }
}

template <typename T> TreeNode<T>* AVLTree<T>::TreeMinimum(TreeNode<T>* cur) const {
    while (cur->lc_ != nullptr) {
        cur = cur->lc_;
    } 
    return cur;  
}
template <typename T> TreeNode<T>* AVLTree<T>::TreeMaximum(TreeNode<T>* cur) const {
    while (cur->rc_ != nullptr) {
        cur = cur->rc_;
    } 
    return cur;  
}
template <typename T> TreeNode<T>* AVLTree<T>::TreeSuccessor(TreeNode<T>* cur) const {
    if (cur->rc_ != nullptr) return TreeMinimum(cur->rc_);
    TreeNode<T>* y = cur->parent_;
    while (y != nullptr && cur == y.rc_) {
        cur = y;
        y = y->parent_;
    }
    return y;
}
template <typename T> TreeNode<T>* AVLTree<T>::TreePredecessor(TreeNode<T>* cur) const {
    if (cur->lc_ != nullptr) return TreeMaximum(cur->lc_);
    TreeNode<T>* y = cur->parent_;
    while (y != nullptr && cur == y.lc_) {
        cur = y;
        y = y->parent_;
    }
    return y;
}
template <typename T> TreeNode<T>*& AVLTree<T>::TreeSearch(T const& keyVal) {
    return TreeSearchRecursive(root_, keyVal);    
}
template <typename T> TreeNode<T>*& AVLTree<T>::TreeSearchRecursive(TreeNode<T>*& cur, T const& keyVal) {
    if (cur == nullptr || keyVal == cur->data_) return cur;
    if (keyVal < cur->data_)
        return TreeSearchRecursive(cur->lc_, keyVal);
    return TreeSearchRecursive(cur->rc_, keyVal);      
}
template <typename T> TreeNode<T>* AVLTree<T>::TreeSearchIterative(TreeNode<T>* cur, T const& keyVal) {
    while (cur != nullptr && keyVal != cur->data_) {
        if (keyVal < cur->data_) cur = cur->lc_;
        else cur = cur->rc_;
    }
    return cur;
}

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
    InsertRebalance(newNode);
    return newNode;
}
template <typename T> bool AVLTree<T>::TreeDeletion(T const& val) {
    TreeNode<T>*& cur = TreeSearch(val);
    if (cur == nullptr) return false;
    TreeDeletion2(cur); 
    return true;  
}
// Intro To Algo's Method
template <typename T> void AVLTree<T>::Transplant(TreeNode<T>* toBeSuAVLTreeituted, TreeNode<T>* vertex) {
    if (toBeSuAVLTreeituted->parent_ == nullptr) root_ = vertex;
    else if (toBeSuAVLTreeituted == toBeSuAVLTreeituted->parent_->lc_) {
        toBeSuAVLTreeituted->parent_->lc_ = vertex;
    } else toBeSuAVLTreeituted->parent_->rc_ = vertex;
    if (vertex != nullptr) vertex->parent_ = toBeSuAVLTreeituted->parent_;
}
template <typename T> void AVLTree<T>::TreeDeletion2(TreeNode<T>* cur) {
    TreeNode<T>* target = cur->parent_; // find the lowest node whose height may need to updated
    if (cur->lc_ == nullptr) {
        Transplant(cur, cur->rc_); // replace cur by its right child
        
    } else if (cur->rc_ == nullptr) {
        Transplant(cur, cur->lc_); // replace cur by its left child    
    } else {
        TreeNode<T>* y = TreeMinimum(cur->rc_);
        if (y->parent_ != cur) { // is y farther down the tree?
            Transplant(y, y->rc_); // replace y by its right child
            y->rc_ = cur->rc_; // z's right child becomes
            y->rc_->parent_ = y; // y's right child
        }
        Transplant(cur, y); // replace cur by its successor y
        y->lc_ = cur->lc_; // and give z's left child to y
        y->lc_->parent_ = y; // which had no left child
        target = (y->rc_ == nullptr) ? y : y-> rc_;     
    }
    UpdateHeightAbove(target);
    DeleteRebalance(target);
    --size_;
}
// 邓俊辉的取巧办法，我并不喜欢,有点取巧走后门的感觉
template <typename T> void AVLTree<T>::TreeDeletion(TreeNode<T>*& cur) {
    TreeNode<T>* toBeDeleted = cur; // 
    TreeNode<T>* succ = nullptr;
    if (cur->lc_ == nullptr) {
        cur = cur->rc_;
        succ = cur;
    } else if (cur->rc_ == nullptr) {
        cur = cur->lc_;
        succ = cur;
    } else {
        toBeDeleted = TreeMinimum(toBeDeleted->rc_);
        std::swap(cur->data_, toBeDeleted->data_);
        TreeNode<T>* u = toBeDeleted->parent_;
        succ = toBeDeleted->rc_;
        if (u == cur) u->rc_ = succ;
        else u->lc_ = succ;
    }
    if (succ != nullptr) succ->parent_ = toBeDeleted->parent_;
    UpdateHeightAbove(toBeDeleted->parent_);
    --size_;
    delete toBeDeleted;
    toBeDeleted = nullptr;    
}
template <typename T> void AVLTree<T>::LeftRotate(TreeNode<T>* x) {
    TreeNode<T>* y = x->rc_;
    if (y == nullptr) return;
    x->rc_ = y->lc_;
    if (y->lc_ != nullptr) y->lc_->parent_ = x;
    y->parent_ = x->parent_;
    if (x->parent_ == nullptr) root_ = y;
    else if (x == x->parent_->lc_) x->parent_->lc_ = y;
    else x->parent_->rc_ = y;
    x->parent_ = y;
    y->lc_ = x;    
    UpdateHeightAbove(x);
}
template <typename T> void AVLTree<T>::RightRotate(TreeNode<T>* x) {
    TreeNode<T>* y = x->lc_;
    if (y == nullptr) return;
    x->lc_ = y->rc_;
    if (y->rc_ != nullptr) y->rc_->parent_ = x;
    y->parent_ = x->parent_;
    if (x->parent_ == nullptr) root_ = y;
    else if (x == x->parent_->lc_) x->parent_->lc_ = y;
    else x->parent_->rc_ = y;
    x->parent_ = y;
    y->rc_ = x; 
    UpdateHeightAbove(x);   
}
template <typename T> void AVLTree<T>::PosiBalance(TreeNode<T>* g) {
    TreeNode<T>* p = TallerChild(g);
    TreeNode<T>* v = TallerChild(p);
    if ( p == g->rc_) {
        if (v == p->rc_) {
            LeftRotate(g);  
        } else {
            RightRotate(p);
            LeftRotate(g);
        }
    } else {
        if (v == p->lc_) {
            RightRotate(g);  
        } else {
            LeftRotate(p);
            RightRotate(g);
        }        
    }
}
template <typename T> void AVLTree<T>::InsertRebalance(TreeNode<T>* x) {
    TreeNode<T>* g = x->parent_;
    while (g != nullptr) {
        if (!AvlBalanced(g)) {
            PosiBalance(g);
        } else {
            g = g->parent_;
        }
    }
}
template <typename T> void  AVLTree<T>::DeleteRebalance(TreeNode<T>* target) {
    TreeNode<T>* g = target;
    while (g != nullptr) {
        if (!AvlBalanced(g)) {
            PosiBalance(g);
        }
        g = g->parent_;
    }
}
template <typename T> int AVLTree<T>::Remove(TreeNode<T>* x) {
    // cut the pointer from parent node
    TreeNode<T>* &pointerOfParent = (x->parent_ == nullptr) ? root_ : ((x == x->parent_->lc_) ? x->parent_->lc_ : x->parent_->rc_);
    pointerOfParent = nullptr;
    // Remove all the nodes of this subtree
    // use Preorder way to remove all the nodes.
    UpdateHeightAbove(x->parent_);
    int toBeDel = 0;
    auto del =[&toBeDel](TreeNode<T>* cur) {
        delete cur;
        cur = nullptr;
        ++toBeDel;
    };
    x->travelPre(del);
    size_ -= toBeDel;
    return toBeDel;
}
template <typename T> void AVLTree<T>::Display() {
    std::cout << "\n";
    if (root_ != nullptr) Display(root_);
    else std::cout << "Empty";
    std::cout << "\n";
}
template <typename T>
void AVLTree<T>::Display(TreeNode<T>* cur, int depth) {
    if (cur->lc_) Display(cur->lc_, depth + 1);

    for (int i=0; i < depth; i++)
        printf("     ");

    if (cur->parent_ != nullptr) {
        if ( cur == cur->parent_->lc_) {
            printf("L----");
        } else printf("R----");
    }
    std::cout << "[" << cur->data_ << "] - (" << cur->height_ << ")" << "\n";
    if (cur->rc_) Display(cur->rc_, depth + 1);    
}


#endif /*AVLTree_h*/
```

main.cpp

```c++
// author: Claude Du
#include "TreeNode.h"
#include "AVLTree.h"
#include "BlackRedTree.h"
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
int main()
{   
    // Test BST
    AVLTree<int> bst;
    bst.TreeInsert(11);    
    bst.TreeInsert(8);
    bst.TreeInsert(9);
    bst.TreeInsert(10);
    bst.TreeInsert(12);
    bst.TreeInsert(13);
    bst.TreeInsert(5);
    bst.TreeInsert(3);
    bst.TreeInsert(1);
    bst.TreeInsert(12);
    bst.TreeInsert(0);
    bst.TreeInsert(4);
    bst.TreeInsert(7);   
    bst.TreeInsert(6);

    std::cout << "The size of Avl tree is " << bst.size() << "\n";
    bst.Display();
    std::cout << "A node with val 0 will be deleteded, so that case 3, left-left, will be satisfied at the first iteration." << "\n";
    std::cout << "Then it will satisfy case 2, Rightt-left, at the second iteration." << "\n";
    bst.TreeDeletion(0);
    std::cout << "The size of Avl tree is " << bst.size() << "\n";
    bst.Display();
}

```

