算法导论ch12二叉搜索树与相应数据结构的c++实现



```c++
// author: Claude Du
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <cmath>
using std::vector;
using std::queue;
using std::stack;
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
    template <typename VST> void travelLevel(VST& visit);
    template <typename VST> void travelPre(VST& visit);
    template <typename VST> void travelIn(VST& visit);
    template <typename VST> void travelPost(VST& visit);
    bool operator< (TreeNode const& bn) { return data_ < bn.data_; }
    bool operator== (TreeNode const& bn) {return data_ < bn.data_; }
};
template<typename T> TreeNode<T>* TreeNode<T>::insertAsLC(T const& lcVal) {
    lc_ = new TreeNode<T>(lcVal, this);
    return lc_;
}
template<typename T> TreeNode<T>* TreeNode<T>::insertAsRC(T const& rcVal) {
    rc_ = new TreeNode<T>(rcVal, this);
    return rc_;
}
template <typename T> template <typename VST> 
void TreeNode<T>::travelLevel(VST& visit) {
    queue<TreeNode<T>*> levelQue;
    
    levelQue.push(this);
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
// 1. InorderTraversal
template <typename T> template <typename VST> 
void TreeNode<T>::travelIn(VST& visit) {
    TravelInIteration(this, visit);
}

// 1.1 recursive version of inorderTraversal
template <typename T, typename VST>
void TravelInRecursive(TreeNode<T>* node, VST& visit) {
    if (!node) return;
    TravelInRecursive(node->lc_, visit);
    visit(node);
    TravelInRecursive(node->rc_, visit);
}
// 1.2 iteration version of inorderTraversal
template <typename T, typename VST>
void TravelInIteration(TreeNode<T>* node, VST& visit) {
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
// 1. Preorder traversal
template <typename T> template <typename VST> 
void TreeNode<T>::travelPre(VST& visit) {
    TravelPreIteration(this, visit);
}

// 1.1 recursive version of Preorder traversal
template <typename T, typename VST>
void TravelPreRecursive(TreeNode<T>* node, VST& visit) {
    if (!node) return;
    visit(node);
    TravelPreRecursive(node->lc_, visit);
    TravelPreRecursive(node->rc_, visit);
}
// 1.2 iteration version of Preorder traversal
template <typename T, typename VST>
void TravelPreIteration(TreeNode<T>* node, VST& visit) {
    stack<TreeNode<T>*> stk;
    TreeNode<T>* cur = node;
    stk.push(cur);
    while (!stk.empty()) {
        cur = stk.top();
        stk.pop();
        visit(cur);
        if (cur->rc_) stk.push(cur->rc_);
        if (cur->lc_) stk.push(cur->lc_);
    } 
}
// 2. Postorder traversal
template <typename T> template <typename VST> 
void TreeNode<T>::travelPost(VST& visit) {
    TravelPostRecursive(this, visit);
}

// 2.1 recursive version of Postorder traversal
template <typename T, typename VST>
void TravelPostRecursive(TreeNode<T>* node, VST& visit) {
    if (!node) return;
    TravelPostRecursive(node->lc_, visit);
    TravelPostRecursive(node->rc_, visit);
    visit(node);    
}

template <typename T> class BST {
protected:
    int size_; TreeNode<T>* root_; // size_ means # of nodes
    virtual int UpdateHeight(TreeNode<T>* node);
    void updateHeightAbove(TreeNode<T>* node);
public:
    BST() : size_(0), root_(nullptr) {}
    ~BST() { if (size_ > 0) remove(root_); }
    int size() const { return size_; }
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
    TreeNode<T>* Search(T const& keyVal) const;
    TreeNode<T>* SearchRecursive(TreeNode<T>* cur, T const& keyVal) const;
    TreeNode<T>* SearchIterative(TreeNode<T>* cur, T const& keyVal) const;
    // insertion
    TreeNode<T>* insertAsRoot(T const& rootVal);
    // deletion
    int remove(TreeNode<T>* x);
    // Traversal
    template <typename VST>
    void travLevel(VST& visit) { if (root_) root_->travLevel(visit); }
    template <typename VST>
    void travelPre(VST& visit) { if (root_) root_->travelIn(visit); }
    template <typename VST>
    void travelPost(VST& visit) { if (root_) root_->travelPost(visit); }
}; 
template <typename T> TreeNode<T>* BST<T>::Search(T const& keyVal) const {
    return SearchIterative(root_, keyVal);    
}
template <typename T> TreeNode<T>* BST<T>::SearchRecursive(TreeNode<T>* cur, T const& keyVal) const {
    if (cur == nullptr || keyVal == cur->data_) return cur;
    if (keyVal < cur->data_)
        return Search(cur->lc_, keyVal);
    return Search(cur->rc_, keyVal);      
}
template <typename T> TreeNode<T>* BST<T>::SearchIterative(TreeNode<T>* cur, T const& keyVal) const {
    while (cur != nullptr && keyVal != cur->data_) {
        if (keyVal < cur->data_) cur = cur->lc_;
        else cur = cur->rc_;
    }
    return cur;
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
    while (y != nullptr && cur == y.right) {
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

int main()
{
    TreeNode<int>* root = new TreeNode<int>(8);
    root->insertAsLC(5);
    root->insertAsRC(11);
    root->lc_->insertAsLC(3);
    root->lc_->insertAsRC(7);
    root->rc_->insertAsRC(12); 
    vector<int> record;
    auto visit = [&](TreeNode<int>* node) {
        record.emplace_back(node->data_);
    };
    root->travelPre(visit);
    for (auto& ele : record) {
        std::cout << ele << " ";
    }
    record.clear();

    
    std::cout << "\n";
}
```

