算法导论ch12二叉搜索树与相应数据结构的c++实现

BinarySearchTree

```c++
#ifndef BinarySearchTree_h
#define BinarySearchTree_h
// author: Claude Du
#include "TreeNode.h" 
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
using std::vector;
using std::queue;
using std::stack;
template <typename T> class BST {
protected:
    int size_; TreeNode<T>* root_; // size_ means # of nodes
    virtual int UpdateHeight(TreeNode<T>* node);
    void UpdateHeightAbove(TreeNode<T>* node);
public:
    BST() : size_(0), root_(nullptr) {}    
    ~BST() { if (size_ > 0) std::cout << Remove(root_); }
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
    TreeNode<T>*& Search(T const& keyVal) const;
    // TreeNode<T>*& SearchRecursive(TreeNode<T>* cur, T const& keyVal) const;
    TreeNode<T>*& SearchIterative(TreeNode<T>* cur, T const& keyVal) const;
    // insertion
    // TreeNode<T>* insertAsRoot(T const& rootVal) { size_ = 1; root_ = new TreeNode<T>(rootVal); return root_; }
    TreeNode<T>* TreeInsert(T const& val);
    // deletion
    void Transplant()

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
template <typename T> TreeNode<T>*& BST<T>::Search(T const& keyVal) const {
    return SearchIterative(root_, keyVal);    
}
// template <typename T> TreeNode<T>* BST<T>::SearchRecursive(TreeNode<T>* cur, T const& keyVal) const {
//     if (cur == nullptr || keyVal == cur->data_) return cur;
//     if (keyVal < cur->data_)
//         return Search(cur->lc_, keyVal);
//     return Search(cur->rc_, keyVal);      
// }
template <typename T> TreeNode<T>*& BST<T>::SearchIterative(TreeNode<T>* cur, T const& keyVal) const {
    while (cur != nullptr && keyVal != cur->data_) {
        if (keyVal < cur->data_) cur = cur->lc_;
        else cur = cur->rc_;
    }
    return cur;
}

template <typename T> TreeNode<T>* BST<T>::TreeInsert(T const& val) {
    TreeNode<T>* targetPosiParent = nullptr;
    TreeNode<T>* curPosi = root_;
    while (curPosi) {
        targetPosiParent = curPosi;
        if (val < curPosi->data_) curPosi = curPosi->lc_;
        else curPosi = curPosi->rc_;
    }
    TreeNode<T>* newNode = new TreeNode<T>(val, targetPosiParent);
    // if tree is empty
    ++size;
    UpdateHeightAbove(newNode);
    if (targetPosiParent == nullptr) {
        root_ = newNode;   
        return newNode;
    }
    if (val < targetPosiParent->data_) {
       targetPosiParent->lc_ = newNode; 
    } else targetPosiParent->rc_ = newNode;
    return newNode;
}

template <typename T> int BST<T>::Remove(TreeNode<T>* x) {
    // cut the pointer from parent node
    TreeNode<T>* &pointerOfParent = (x->parent == nullptr) ? root_ : ((x == x->parent_->lc_) ? x->parent_->lc_ : x->parent_->rc_);
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
#define stature(node) ((node) ? (node)->height : -1)
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
    bool operator< (TreeNode const& bn) { return data_ < bn.data_; }
    bool operator== (TreeNode const& bn) {return data_ < bn.data_; }
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
    };
    root->travelLevel(visit);
    for (auto& ele : record) {
        std::cout << ele << " ";
    }
    record.clear();
    // Test BST


    
    std::cout << "\n";
}
```

