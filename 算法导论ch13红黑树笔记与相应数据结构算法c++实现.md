算法导论ch13红黑树笔记与相应数据结构算法c++实现



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
using std::vector;
using std::queue;
using std::stack;

template <typename T>
struct BRNode {
    T data_;
    BRNode<T>* parent_;
    BRNode<T>* lc_;
    BRNode<T>* rc_;
    RBColor color_;
    BRNode() : lc_(nullptr), rc_(nullptr), color_(RB_RED) {};
    BRNode(T data, TreeNode<T>* parent = nullptr, TreeNode<T>* lc = nullptr,
            TreeNode<T>* rc = nullptr, RBColor color = RB_RED) : 
            data_(data), parent_(parent), lc_(lc), rc_(rc), color_(color) {}
};

template <typename T> class RBTree {
private:
    BRNode<T>* root_;
    BRNode<T>* tNil;
    void InitializeNullNode(BRNode<T>* node, BRNode<T>* parent);
public:
    RBTree() {
        tNil = new BRNode<T>();
        tNil->color_ = RB_BLACK;
        root_ = tNil;
    }
    void LeftRotate(BRNode<T>* x);
    void RightRotate(BRNode<T>* x);
    BRNode<T>* RBInsert(T const& val);
    void RBInsertFixup(BRNode<T>* z);


};
template <typename T>
void RBTree<T>::InitializeNullNode(BRNode<T>* node, BRNode<T>* parent) {
    node->data_ = 0;
    node->parent_ = parent;
    node->lc_ = nullptr;
    node->rc_ = nullptr;
    node->color_ = RB_BLACK;
}
template <typename T>
void RBTree<T>::LeftRotate(BRNode<T>* x) {
    BRNode<T>* y = x->rc_;
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
void RBTree<T>::RightRotate(BRNode<T>* x) {
    BRNode<T>* y = x->lc_;
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
BRNode<T>* RBTree<T>::RBInsert(T const& val) {
    BRNode<T>* curPosi = root_; // node being compared with z
    BRNode<T>* targetPosiParent = tNil; // targetPosiParent will be parent of the target node with the value val
    while (curPosi != tNil) {
        targetPosiParent = curPosi;
        if (curPosi->data_ > val) curPosi = curPosi->lc_;
        else curPosi = curPosi->rc_;
    }
    BRNode<T>* z = new BRNode<T>(val, targetPosiParent, tNil, tNil, RB_RED);
    if (targetPosiParent == tNil) {
        root_ = z;
        return z;
    } 
    else if (targetPosiParent->data_ > val) targetPosiParent->lc_ = z;
    targetPosiParent->rc_ = z;
    RBInsertFixUp(z);
    return z;
}
template <typename T>
void RBTree<T>::RBInsertFixUp(BRNode<T>* z) {
}

#endif /*BinarySearchTree_h*/
```

