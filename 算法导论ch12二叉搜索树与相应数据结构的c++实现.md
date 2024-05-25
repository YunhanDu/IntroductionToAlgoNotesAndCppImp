# 算法导论ch12二叉搜索树笔记与其数据结构的c++实现

本篇笔记主要来源于算法导论第三版中文版与算法导论第四版英文版，c++代码实现有参考数据结构（c++语言版）第三版，如有错误请务必指出。

搜索树数据结构支持许多动态集合操作：SEARCH, MINIMUM, MAXIMUM, PREDECESSOR, SUCCESSOR, INSERT和DELETE等。因此，我们使用一棵搜索树既可以作为一个字典又可以作为一个优先队列。

二叉搜索树上的基本操作所花费的时间与这棵树的高度成正比。对于有n节点的完全二叉树，这些操作的最坏运行时间为 $\Theta(lgn)$ 。但对于有n节点的线性链形的二叉树，这些操作的最坏运行时间为 $\Theta(n)$ 。

实际上，我们经常使用的是二叉搜索树的变种，因为它们可以保证基本操作有较好的最坏情况性能。比如13章的红黑树，（相应笔记与c++实现链接https://3ms.huawei.com/km/blogs/details/15343190?l=zh-cn），18章的B树，特别适用于二级（磁盘）存储器上的数据库维护。





## 12.1 什么是二叉搜索树(BST)

顾名思义，一棵二叉搜索树是以一棵二叉树来组织的，如图12-1所示，这样一棵树可以使用一个链表数据结构来表示，其中每个节点就是一个对象。除了key_和卫星

12.2 查询BST（读操作）

BST支持这些查询操作：查找--TreeSearch，最大关键字元素--TreeMaximum， 最小关键字元素--TreeMinimum， 后继--TreeSuccessor 和 前驱--TreePredecessor。本节会讨论这些操作与c++实现细节，并说明在任何高度为h的BST上，如何在 �(ℎ) 时间内执行完每个操作。

查找--TreeSearch

最大关键字元素--TreeMaximum和最小关键字元素--TreeMinimum

后继--TreeSuccessor 和 前驱--TreePredecessor

## 12.3 插入和删除（写操作）



### 插入--TreeInsert



插入

### 删除--TreeDeletion



从一颗BST中删除一个节点z（假设删除该节点前，BST一定储存了该节点）的整个策略可以分为如下三种基本情况：

基本情况1：如果z没有child, 那只用简单地将它删除--通过修改它的父节点，用nullptr作为孩子替换z。

基本情况2：如果z只有一个child，那么把该child提升到树中z的位置即可（相应算法实现稍后讲）。（中文第三版中相应的表述个人认为属于翻译错误）

基本情况3：如果z有2个child, 则先找到z的后继y(一定在z的右子树中并且没有左孩子)， 让y占据树中z的位置。z原来右子树部分成为y的新右子树，并让z的左子树成为y的新左子树，此情形的相应算法实现有些棘手，因为还与y是否是z的right child相关。

删除节点z的程序TreeDeletion会取指向该BST和节点z的指针作为输入参数（在我的c++实现里，指向BST的指针就是BST类的this指针，所以不会显示表示该参数），之前划分的三种情形可以重组拆分成如下的更适用于我们算法实现的四种情形：

a. 节点z没有left child（如图12.4 a）, 那么用z的right child替换z, z可以是nullptr(对应基本情况1)，也可以不是nullptr。(对应基本情况2中z只有right child )

b. 节点z没有right child（如图12.4 b）, 那么用z的left child替换z（对应基本情况2中z只有left child）

否则，z一定有2个child，则先找到z的后继y。(一定在z的右子树中并且没有左孩子)：

c, 如果y是z的right child（如图12.4 c）, 那么用y替换z。（对应基本情况3中y是z的right child）

d, 否则，y不是z的right child（如图12.4 d)，在这种情况下，先用y的右孩子替换y，然后再用y替换z。（对应基本情况3中y不是z的right child）

图12.4如下：

![img](./treedeletion.PNG)

为了在二叉树内移动子树，定义一个子过程Transplant, 它用一棵子树A(代码中vertex) 替换一棵子树B(代码中toBeSubstituted)并成为B(代码中toBeSubstituted)在其parent原来相应的child节点，其c++实现如下：

```c++
template <typename T> void BST<T>::Transplant(TreeNode<T>* toBeSubstituted, TreeNode<T>* vertex) {
    if (toBeSubstituted->parent_ == nullptr) root_ = vertex;
    else if (toBeSubstituted == toBeSubstituted->parent_->lc_) {
        toBeSubstituted->parent_->lc_ = vertex;
    } else toBeSubstituted->parent_->rc_ = vertex;
    if (vertex != nullptr) vertex->parent_ = toBeSubstituted->parent_;
}
```

利用现成的Transplant过程，下面是从二叉树T中删除节点的c++实现：

```c++
template <typename T> void BST<T>::TreeDeletion2(TreeNode<T>* z) {
    // node target is used to updateHeight, you may not put too much attention on it
    TreeNode<T>* target = z->parent_; // find the lowest node whose height may need to updated
    if (z->lc_ == nullptr) {
        Transplant(z, z->rc_); // case a: replace z by its right child
        
    } else if (z->rc_ == nullptr) {
        
        Transplant(z, z->lc_); // case b: replace z by its left child    
    } else {
        TreeNode<T>* y = TreeMinimum(z->rc_);
        if (y->parent_ != z) { // is y farther down the tree?
            // case d's first step: replace y by its right child 
            Transplant(y, y->rc_); 
            y->rc_ = z->rc_; // z's right child becomes
            y->rc_->parent_ = y; // y's right child
        }
        // case c's procedure is the same as case d's last step
        Transplant(z, y); // replace z by its successor y
        y->lc_ = z->lc_; // and give z's left child to y
        y->lc_->parent_ = y; // which had no left child
        target = (y->rc_ == nullptr) ? y : y-> rc_;     
    }
    UpdateHeightAbove(target);
    --size_;
}
```

c++实现中的target节点是用来维护删除过程中树中节点高度，非删除过程中的核心代码，可以不用给予过多关注。【见附录】



BinarySearchTree.h

```c++
#ifndef BinarySearchTree_h
#define BinarySearchTree_h
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
template <typename T> class BST {
protected:
    int size_; TreeNode<T>* root_; // size_ means # of nodes
    virtual int UpdateHeight(TreeNode<T>* node);
    void UpdateHeightAbove(TreeNode<T>* node);
    void Display(TreeNode<T>* cur, int depth = 0);
    void Transplant(TreeNode<T>* toBeSubstituted, TreeNode<T>* vertex);
public:
    BST() : size_(0), root_(nullptr) {}    
    ~BST() { 
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
    TreeNode<T>* TreeMinimum(TreeNode<T>* cur) const;
    TreeNode<T>* TreeMaximum() const {
        if (root_ == nullptr) return root_; 
        return TreeMaximum(root_); 
    }
    TreeNode<T>* TreeMaximum(TreeNode<T>* cur) const;
    TreeNode<T>* TreeSuccessor(TreeNode<T>* cur) const;
    TreeNode<T>* TreePredecessor(TreeNode<T>* cur) const;
    // search
    TreeNode<T>*& Search(T const& keyVal);
    TreeNode<T>*& SearchRecursive(TreeNode<T>*& cur, T const& keyVal);
    TreeNode<T>* SearchIterative(TreeNode<T>* cur, T const& keyVal);
    // insertion
    TreeNode<T>* TreeInsert(T const& val);
    // deletion
    bool TreeDeletion(T const& val);
    void TreeDeletion(TreeNode<T>*& cur);
    void TreeDeletion2(TreeNode<T>* cur);
    int Remove(TreeNode<T>* x);
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

template <typename T> TreeNode<T>* BST<T>::TreeMinimum(TreeNode<T>* cur) const {
    while (cur->lc_ != nullptr) {
        cur = cur->lc_;
    } 
    return cur;  
}
template <typename T> TreeNode<T>* BST<T>::TreeMaximum(TreeNode<T>* cur) const {
    while (cur->rc_ != nullptr) {
        cur = cur->rc_;
    } 
    return cur;  
}
template <typename T> TreeNode<T>* BST<T>::TreeSuccessor(TreeNode<T>* cur) const {
    if (cur->rc_ != nullptr) return TreeMinimum(cur->rc_);
    TreeNode<T>* y = cur->parent_;
    while (y != nullptr && cur == y.rc_) {
        cur = y;
        y = y->parent_;
    }
    return y;
}
template <typename T> TreeNode<T>* BST<T>::TreePredecessor(TreeNode<T>* cur) const {
    if (cur->lc_ != nullptr) return TreeMaximum(cur->lc_);
    TreeNode<T>* y = cur->parent_;
    while (y != nullptr && cur == y.lc_) {
        cur = y;
        y = y->parent_;
    }
    return y;
}
template <typename T> TreeNode<T>*& BST<T>::Search(T const& keyVal) {
    return SearchRecursive(root_, keyVal);    
}
template <typename T> TreeNode<T>*& BST<T>::SearchRecursive(TreeNode<T>*& cur, T const& keyVal) {
    if (cur == nullptr || keyVal == cur->data_) return cur;
    if (keyVal < cur->data_)
        return SearchRecursive(cur->lc_, keyVal);
    return SearchRecursive(cur->rc_, keyVal);      
}
template <typename T> TreeNode<T>* BST<T>::SearchIterative(TreeNode<T>* cur, T const& keyVal) {
    while (cur != nullptr && keyVal != cur->data_) {
        if (keyVal < cur->data_) cur = cur->lc_;
        else cur = cur->rc_;
    }
    return cur;
}

template <typename T> TreeNode<T>* BST<T>::TreeInsert(T const& val) {
    TreeNode<T>* targetPosiParent = nullptr;
    TreeNode<T>* curPosi = root_;
    while (curPosi != nullptr) {
        targetPosiParent = curPosi;
        if (val < curPosi->data_) curPosi = curPosi->lc_;
        else curPosi = curPosi->rc_;
    }
    TreeNode<T>* newNode = new TreeNode<T>(val, targetPosiParent);
    // if tree is empty
    ++size_;
    
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
template <typename T> bool BST<T>::TreeDeletion(T const& val) {
    TreeNode<T>*& cur = Search(val);
    if (cur == nullptr) return false;
    TreeDeletion2(cur); 
    return true;  
}
// Intro To Algo's Method
template <typename T> void BST<T>::Transplant(TreeNode<T>* toBeSubstituted, TreeNode<T>* vertex) {
    if (toBeSubstituted->parent_ == nullptr) root_ = vertex;
    else if (toBeSubstituted == toBeSubstituted->parent_->lc_) {
        toBeSubstituted->parent_->lc_ = vertex;
    } else toBeSubstituted->parent_->rc_ = vertex;
    if (vertex != nullptr) vertex->parent_ = toBeSubstituted->parent_;
}
template <typename T> void BST<T>::TreeDeletion2(TreeNode<T>* cur) {
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
    --size_;
}
// 邓俊辉的取巧办法，我并不喜欢,有点取巧走后门的感觉
template <typename T> void BST<T>::TreeDeletion(TreeNode<T>*& cur) {
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
        else u->rc_ = succ;
    }
    if (succ != nullptr) succ->parent_ = toBeDeleted->parent_;
    UpdateHeightAbove(toBeDeleted->parent_);
    --size_;
    delete toBeDeleted;
    toBeDeleted = nullptr;
    
}

template <typename T> int BST<T>::Remove(TreeNode<T>* x) {
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
template <typename T> void BST<T>::Display() {
    std::cout << "\n";
    if (root_ != nullptr) Display(root_);
    else std::cout << "Empty";
    std::cout << "\n";
}
template <typename T>
void BST<T>::Display(TreeNode<T>* cur, int depth) {
    if (cur->lc_) Display(cur->lc_, depth + 1);

    for (int i=0; i < depth; i++)
        printf("     ");

    if (cur->parent_ != nullptr) {
        if ( cur == cur->parent_->lc_) {
            printf("┌───");
        } else printf("└───");
    }
    std::cout << "[" << cur->data_ << "] - (" << cur->height_ << ")" << "\n";
    if (cur->rc_) Display(cur->rc_, depth + 1);    
}

#endif /*BinarySearchTree_h*/
```

TreeNode.h

```
#ifndef TreeNode_h
#define TreeNode_h
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
using std::vector;
using std::queue;
using std::stack;
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
    template <typename VST> void travelPost(VST& visit) { TravelPostRecursive(this, visit);}
};
// 1 level traversal
template <typename T, typename VST>  
void TravelLevel(TreeNode<T>* node, VST& visit) {
    queue<TreeNode<T>*> levelQue;
    
    levelQue.push(node);
    while (!levelQue.empty()) {
        TreeNode<T>* cur = levelQue.front();
        visit(cur);
        levelQue.pop();
        if (cur->lc_) {
            levelQue.push(cur->lc_);
        }
        if (cur->rc_) {
            levelQue.push(cur->rc_);
        }
    }
}
// 2.2 Iterative version of inorderTraversal
template <typename T, typename VST>
void TravelInIterative(TreeNode<T>* node, VST& visit) {
    stack<TreeNode<T>*> stk;
    TreeNode<T>* cur = node;
    while (cur || !stk.empty()) {
        if (cur) {
            stk.push(cur);
            cur = cur->lc_;
            continue;        
        }
        cur = stk.top();
        stk.pop();
        visit(cur);
        cur = cur->rc_;
    } 
}

// 3.2 Iterative version of Preorder traversal
template <typename T, typename VST>
void TravelPreIterative(TreeNode<T>* node, VST& visit) {
    stack<TreeNode<T>*> stk;
    TreeNode<T>* cur = node;
    stk.push(cur);
    while (!stk.empty()) {
        cur = stk.top();
        stk.pop();
        if (cur->rc_) stk.push(cur->rc_);
        if (cur->lc_) stk.push(cur->lc_);
        visit(cur);
    } 
}

// 4.1 recursive version of Postorder traversal
template <typename T, typename VST>
void TravelPostRecursive(TreeNode<T>* node, VST& visit) {
    if (!node) return;
    TravelPostRecursive(node->lc_, visit);
    TravelPostRecursive(node->rc_, visit);
    visit(node);    
}



#endif /*TreeNode_h*/
```

TreeNode.cpp

```
// author Yunhan Du
#include "TreeNode.h"
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
using std::vector;
using std::queue;
using std::stack;
template<typename T> TreeNode<T>* TreeNode<T>::insertAsLC(T const& lcVal) {
    lc_ = new TreeNode<T>(lcVal, this);
    return lc_;
}
template<typename T> TreeNode<T>* TreeNode<T>::insertAsRC(T const& rcVal) {
    rc_ = new TreeNode<T>(rcVal, this);
    return rc_;
}

template class TreeNode<int>;
```

main.cpp

```
// author: Claude Du
#include "TreeNode.h"
#include "BinarySearchTree.h"
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>

int main()
{
    // Test TreeNode
    TreeNode<int>* root = new TreeNode<int>(8);
    root->insertAsLC(5);
    root->insertAsRC(11);
    root->lc_->insertAsLC(3);
    root->lc_->insertAsRC(7);
    root->rc_->insertAsRC(12); 
    std::vector<int> record;
    auto visit = [&](TreeNode<int>* node) {
        record.emplace_back(node->data_);
        std::cout << node->data_ << " ";
    };
    root->travelLevel(visit);
    std::cout << "\n";
    record.clear();
    // Test BST
    BST<int> bst;
    bst.TreeInsert(8);
    bst.TreeInsert(12);
    bst.TreeInsert(5);
    bst.TreeInsert(3);
    bst.TreeInsert(7);
    bst.TreeInsert(13);
    bst.TreeInsert(10);
    bst.TreeInsert(11);
    bst.TreeInsert(9);
    bst.TreeInsert(16);
    bst.travelIn(visit);
    std::cout << "\n";
    std::cout << bst.size() << "\n";
    record.clear();
    bst.Display();
    bst.TreeDeletion(10);
    std::cout << bst.size() << "\n";
    bst.Display();
    bst.TreeDeletion(9);
    std::cout << bst.size() << "\n";
    bst.Display();
    bst.TreeDeletion(16);
    std::cout << bst.size() << "\n";
    bst.Display();
    std::cout << "TreeMaximum: " << bst.TreeMaximum()->data_ << "\n";
}
```

