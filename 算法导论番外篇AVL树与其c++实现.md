# 算法导论番外篇AVL树与其c++实现

作者：Claude Du

本文的动机：想到MIT算法公开课AVL树章节也听了（有趣且有启发性，但内容不全面），之前算法导论12章BST与13章红黑树的笔记也都写了，似乎没有太多理由放过AVL树，更何况算法导论13章思考题13-3也要求我论证AVL树的性质和实现AVL树的一些动态功能，而且对于我个人而言，似乎也是不错的算法锻炼与写作挑战机会，那么开整！

PS：

​		MIT算法公开课AVL树链接：

​		本人写的12章BST笔记与其数据结构c++实现的链接：https://3ms.huawei.com/km/blogs/details/15350515?l=zh-cn

​		本人写的13章红黑树笔记与其数据结构c++实现的链接：https://3ms.huawei.com/km/blogs/details/15343190?l=zh-cn

我将尝试模仿算法导论13章的叙述结构与范式来写此文，这是本人的第一次尝试，如果有错误和建议，还请各位指出，感谢！！

## 1 AVL树的性质

AVL树是一种高度平衡的二叉搜索树：**对每一个节点x, 对x的左子树与右子树的高度差至多为1。** 要实现一棵AVL树，需要在每个节点维护一个额外的属性：x.height为节点x的高度。

AVL树的节点TreeNode的c++实现框架如下：

```c++

```

AVL树可以实现近乎理想的平衡。在渐进意义下，AVL树可始终将高度控制在 $O(lgn)$ 以内（引理1证明在下文），从而保证每次查找，插入和删除操作，均可在 $O(lgn)$ 的时间内完成。

引理1：一棵有n个节点的AVL树高度为$O(lgn)$ 。

引理1的证明在MIT算法公开课中有提到，感兴趣的可以去看一看。

以下为我的证明：



AVL树是对节点高度敏感的，所以我们要经常更新二叉树的高度，每当某一节点的孩子或后代有所增减，其高度都有必要及时更新。然而实际上，节点自身很难发现后代的变化，因此我们不妨反过来用另一处理策略：如果有节点加入，离开AVL树或位置变更，则更新所有祖先的高度。

```c++
#define stature(node) ((node) ? (node)->height_ : -1)
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

AVL树的c++实现框架如下:

```c++

```

