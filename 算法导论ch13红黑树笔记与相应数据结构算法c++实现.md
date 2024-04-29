算法导论ch13红黑树笔记与相应数据结构算法c++实现

作者：Claude Du





13.3 插入

为了在$O(lg(n))$ 时间内完成向一棵含 $n$ 个内部节点的红黑树 $T$ 插入一个新节点，我们要对12.3节的TreeInsert操作略作修改。在RBInsert流程中，首先要**通过和普通二叉搜索树一样的插入操作**，将key值为val的节点z插入红黑树 $T$ 内，然后将节点z着色成**红色**（着色成红色的原因见附录）。接下来，为了保证红黑性质能继续保持。我们要调用一个辅助程序RBInsertFixUp来对节点z再次着色并进行旋转操作，以下为在红黑树内插入值为val的新节点RBInsert程序的c++实现（辅助程序RBInsertFixUp具体实现稍后详解）：

```c++
template <typename T>
RBNode<T>* RBTree<T>::RBInsert(T const& val) {
    RBNode<T>* curPosi = root_; // node being compared with z
    RBNode<T>* targetPosiParent = tNil; // targetPosiParent will be parent of the target node with the value val
    while (curPosi != tNil) { // descending until reaching the sentinel
        targetPosiParent = curPosi;
        if (curPosi->data_ > val) curPosi = curPosi->lc_;
        else curPosi = curPosi->rc_;
    }
    // found the location, create node z, and insert z with its parent, targetPosiParent
    // moreover, make sure that both of z's children are the sentinel and z starts out red
    RBNode<T>* z = new RBNode<T>(val, targetPosiParent, tNil, tNil, RB_RED); 
    if (targetPosiParent == tNil) {
        // tree was empty
        root_ = z;
        root_->color_ = RB_BLACK;
        return z;
    } 
    else if (targetPosiParent->data_ > val) targetPosiParent->lc_ = z;
    else targetPosiParent->rc_ = z;
    RBInsertFixUp(z);
    return z;
}
```

程序Tree

```c++
#ifndef BlackRedTree_h
#define BlackRedTree_h
// author: Claude Du
#include "TreeNode.h"
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <algorithm>
#include <cmath>
#include <string>
using std::vector;
using std::queue;
using std::stack;

template <typename T>
struct RBNode {
    T data_;
    RBNode<T>* parent_;
    RBNode<T>* lc_;
    RBNode<T>* rc_;
    RBColor color_;
    // RBNode() : lc_(nullptr), rc_(nullptr), color_(RB_RED) {};
    RBNode(T data = 0, RBNode<T>* parent = nullptr, RBNode<T>* lc = nullptr,
            RBNode<T>* rc = nullptr, RBColor color = RB_RED) : 
            data_(data), parent_(parent), lc_(lc), rc_(rc), color_(color) {}
};

template <typename T> class RBTree {
private:
    RBNode<T>* root_;
    RBNode<T>* tNil;
    const vector<std::string> colorStr{"RED", "BLK"};
    void InitializeNullNode(RBNode<T>* node, RBNode<T>* parent);
    void Display(RBNode<T>* cur, int depth = 0);
public:
    RBTree() {
        tNil = new RBNode<T>();
        tNil->color_ = RB_BLACK;
        root_ = tNil;
    }
    void LeftRotate(RBNode<T>* x);
    void RightRotate(RBNode<T>* x);
    // insertion
    RBNode<T>* RBInsert(T const& val);
    void RBInsertFixUp(RBNode<T>* z);
    // deletion
    void RBTransplant(RBNode<T>* toBeSubstituted, RBNode<T>* vertex);
    bool RBDeletion(T const& val);
    void RBDeletion(TreeNode<T>*& cur);
    // debug
    void Display();


};
template <typename T>
void RBTree<T>::InitializeNullNode(RBNode<T>* node, RBNode<T>* parent) {
    node->data_ = 0;
    node->parent_ = parent;
    node->lc_ = nullptr;
    node->rc_ = nullptr;
    node->color_ = RB_BLACK;
}
template <typename T>
void RBTree<T>::LeftRotate(RBNode<T>* x) {
    RBNode<T>* y = x->rc_;
    if (y == nullptr) return;
    x->rc_ = y->lc_;
    if (y->lc_ != tNil) y->lc_->parent_ = x;
    y->parent_ = x->parent_;
    if (x->parent_ == tNil) root_ = y;
    else if (x == x->parent_->lc_) x->parent_->lc_ = y;
    else x->parent_->rc_ = y;
    y->lc_ = x;
    x->parent_ = y;   
}
template <typename T>
void RBTree<T>::RightRotate(RBNode<T>* x) {
    RBNode<T>* y = x->lc_;
    if (y == nullptr) return;
    x->lc_ = y->rc_;
    if (y->rc_ != tNil) y->rc_->parent_ = x;
    y->parent_ = x->parent_;
    if (x->parent_ == tNil) root_ = y;
    else if (x == x->parent_->lc_) x->parent_->lc_ = y;
    else x->parent_->rc_ = y;
    y->rc_ = x;
    x->parent_ = y;      
}
template <typename T>
RBNode<T>* RBTree<T>::RBInsert(T const& val) {
    RBNode<T>* curPosi = root_; // node being compared with z
    RBNode<T>* targetPosiParent = tNil; // targetPosiParent will be parent of the target node with the value val
    while (curPosi != tNil) { // descending until reaching the sentinel
        targetPosiParent = curPosi;
        if (curPosi->data_ > val) curPosi = curPosi->lc_;
        else curPosi = curPosi->rc_;
    }
    // found the location, create node z, and insert z with its parent, targetPosiParent
    // moreover, make sure that both of z's children are the sentinel and z starts out red
    RBNode<T>* z = new RBNode<T>(val, targetPosiParent, tNil, tNil, RB_RED); 
    if (targetPosiParent == tNil) {
        // tree was empty
        root_ = z;
        root_->color_ = RB_BLACK;
        return z;
    } 
    else if (targetPosiParent->data_ > val) targetPosiParent->lc_ = z;
    else targetPosiParent->rc_ = z;
    RBInsertFixUp(z);
    return z;
}
template <typename T>
void RBTree<T>::RBInsertFixUp(RBNode<T>* z) {
    while (z->parent_->color_ == RB_RED) {
        RBNode<T>* uncle = tNil; // uncle will be z's uncle
        if (z->parent_ == z->parent_->parent_->lc_) { // is z’s parent a left child?
            uncle = z->parent_->parent_->rc_; 
            if (uncle->color_ == RB_RED)  {// are z's parent and uncle both red?
                // case 1:  z's parent and uncle are both red
                z->parent_->color_ = RB_BLACK;
                uncle->color_ = RB_BLACK;
                z->parent_->parent_->color_ = RB_RED;
                z = z->parent_->parent_;
            } else {
                if (z == z->parent_->rc_) {
                    // case 2: uncle is red and 
                    z == z->parent_;
                    LeftRotate(z);
                }
                // case 3
                z->parent_->color_ = RB_BLACK;
                z->parent_->parent_->color_ = RB_RED;
                RightRotate(z->parent_->parent_);
            }
        } else { // same as lines 103~119, but with "lc_" and "rc_" exchanged
            uncle = z->parent_->parent_->rc_;
            if (uncle->color_ == RB_RED) {
                z->parent_->color_ = RB_BLACK;
                uncle->color_ = RB_BLACK;
                z->parent_->parent_->color_ = RB_RED;
                z = z->parent_->parent_;
            } else {
                if (z == z->parent_->lc_) {
                    z= z->parent_;
                    RightRotate(z);
                }
                z->parent_->color_ = RB_BLACK;
                z->parent_->parent_->color_ = RB_RED;
                LeftRotate(z->parent_->parent_);
            }
        }
    }
    root_->color_ = RB_BLACK;
}
template <typename T>
void RBTree<T>::RBTransplant(RBNode<T>* toBeSubstituted, RBNode<T>* vertex) {
    if (toBeSubstituted->parent_ == tNil) root_ = vertex;
    else if (toBeSubstituted == toBeSubstituted->parent_->lc_) toBeSubstituted->parent_->lc_ = vertex;
    else toBeSubstituted->parent_->lc_ = vertex;
    vertex->parent_ = toBeSubstituted->parent_;
}

template <typename T>
void RBTree<T>::Display(RBNode<T>* cur, int depth) {
    if (cur->lc_ != tNil && cur->lc_ != nullptr) Display(cur->lc_, depth + 1);
    for (int i = 0; i < depth; ++i) std::cout << "      ";
    if (cur != root_) {
        if (cur == cur->parent_->lc_) {
            std::cout << "L----";
        } else std::cout << "R----";
    }
    std::cout << "(" << cur->data_ << ")" <<"-"<< colorStr[cur->color_] << "\n";
    if (cur->rc_ != tNil && cur->rc_ != nullptr) Display(cur->rc_, depth + 1);
}
template <typename T>
void RBTree<T>::Display() {
    if (root_ != tNil && root_ != nullptr) Display(root_);
}


#endif /*BinarySearchTree_h*/
```

