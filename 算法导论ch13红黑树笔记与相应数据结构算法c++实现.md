# 算法导论ch13红黑树笔记与相应数据结构算法c++实现

作者：Claude Du

框架和核心c++代码已写完，全文暂时未写完，期望本周可以完成。

本文主要内容来源于算法导论2022年第四版英文版，13.3节和13.4节基本就是英文翻译，与修改了原书中部分描述性的错误。

红黑树整体的c++实现已放在附录中。

## 13.1红黑树性质

红黑树是一棵二叉搜索树，它在每个节点增加了一个存储位来表示节点的颜色，RED或BLACK。通过对任何一条从根到叶子的简单路径上各个节点的颜色进行约束，红黑树确保没有一条路径会比其他路径长出2倍。因而可以近似认为是平衡的。

一棵红黑树满足下面红黑性质的二叉树：

1. **每个节点或是红色的，或是黑色的。**
2. **根节点是黑色的。**
3. **每个叶节点（Nil）是黑色的**
4. **如果一个节点是红色的，则它的两个子节点都是黑色的。**
5. **对每个节点，从该节点到其所有后代叶结点的简单路径上，均包含相同数目的黑色节点。**

## 13.2旋转

12章的二叉搜索树操作TreeInsert和TreeDelete在含有n个关键字的红黑树上运行花费时间为$O(lgn)$ 。由于这两个操作对树做了修改，其操作结果可能违反13.1节中列出的红黑性质，为了维护这些性质，必须要改变这些节点的颜色以及指针结构。其中指针结构的修改是通过旋转来完成的。

旋转是一种能局部保持二叉搜索树性质的搜索树局部操作。下图（二叉搜索树中虚线箭头为child指向parent, 实线箭头为parent指向child）给出了两种旋转，左旋和右旋。图中节点x上做左旋时，假设x的right child y 不是tNil。



![](./%E6%97%8B%E8%BD%AC13-2.PNG)

LeftRotate的c++代码实现如下：

```c++
template <typename T>
void RBTree<T>::LeftRotate(RBNode<T>* x) {
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
}
```

LeftRotate程序实现左旋功能的详细流程如下图所示：

![](./%E6%97%8B%E8%BD%AC%E8%AF%A6%E7%BB%86%E6%B5%81%E7%A8%8B.PNG)

RightRotate的代码实现和LeftRotate的代码实现是对称的，把 rc和 lc 对调一下即可。

RightRotate的c++代码实现如下（Exercise 13.2-1）：

```c++
template <typename T>
void RBTree<T>::RightRotate(RBNode<T>* x) {
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
}
```

LeftRotate和RightRotate都在 $O(1)$ 的时间内运行完成。在旋转操作中只有指针改变，其他所有属性都保持不变。

## 13.3 插入（遵循第4版英文版）

为了在$O(lg(n))$ 时间内完成向一棵含 $n$ 个内部节点的红黑树 $T$ 插入一个新节点，我们要对12.3节的TreeInsert操作略作修改。在RBInsert流程中，首先要**通过和普通二叉搜索树一样的插入操作**，将key值为val的节点z插入红黑树 $T$ 内，然后将节点z着色成**红色**（着色成红色的原因见附录，习题13.3-1）。接下来，为了保证红黑性质能继续保持。我们要调用一个辅助程序RBInsertFixUp来对节点z再次着色并进行旋转操作，以下为在红黑树内插入值为val的新节点RBInsert程序的c++实现（辅助程序RBInsertFixUp具体实现稍后详解）：

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

12.3节的TreeInsert操作和RBInsert之间有4处不同：

1. TreeInsert内所有nullptr都被tNil代替。
2. 第12行将z节点的left child和right child置为tNil，以保持合理的红黑树结构。
3. 第12行将z节点的颜色着为红色。
4. 因为将z着为红色可能违反其中的一条红黑性质，所以在21行调用RBInsertFixUp(z)来维护红黑性质。

其中RBInsertFixUp的c++代码实现如以下代码块所示。为了理解该段RBInsertFixUp程序如何工作，我们把代码分为三个主要步骤：

1，要确定当前节点z被插入并着色为红色后，红黑性质有哪些不能继续保持。

2，要分析第4行到40行中while循环的总目标。

3， 要分析while循环体中的3种case（case 2执行完后紧跟着case 3, case 2 和case 3不是互相独立的情形），看看它们如何完成目标的。

```c++
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
                    // case 2: uncle is black and z is a right child 
                    z = z->parent_;
                    LeftRotate(z);
                }
                // case 3: uncle is black and z is a left child
                z->parent_->color_ = RB_BLACK;
                z->parent_->parent_->color_ = RB_RED;
                RightRotate(z->parent_->parent_);
            }
        } else { // same as lines 4~22, but with "lc_" and "rc_" exchanged
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
```

在描述红黑树结构中，我们经常要提到某个节点的parent的兄弟，我们会用uncle称呼该节点。下图显示了RBInsertFixUp要如何在一个具体红黑树上操作。

![](./RBInsertFixUp%E6%93%8D%E4%BD%9C.PNG)

在调用RBInsertFixUp操作时，哪些红黑性质会被破坏呢？

性质1，性质3和性质5都不会被破坏。由于z一开始被着色为红色，只有性质2和性质4有可能被破坏。上图(a)显示在插入节点z后，性质4被破坏的情况。

while循环中的第5至第40行有两个等价对称情形：第5行到23行处理z的parent 是 z的grandParent 的leftChild的情形，第24行到40行处理的是z的parent 是z的grandParent 的rightChild的情形。基于第24行到40行和第5行到23行的对称等价性，我们的论证只聚焦于第5行到23行。

第5行到23行的while循环在每次迭代开头保持下面3个部分的**循环不变式**：

a. 节点z是红节点。

b. 如果z.parent是根节点，那么z.parent是黑节点。

c. 如果有任何红黑性质被破坏，则至多只有一条被破坏，或是性质2(因为此时z为根节点且为红节点)，或是性质4（因为此时z和z.parent都是红节点）。

我们要证明上面的循环不变式（a, b, c）在第一次迭代前，后续每一次迭代开头为真，并在while循环终止时，这些不变式会给出一个有用的性质。同时循环中每次迭代都有两种可能的结果：

a.节点沿着树上移。

b.执行某些旋转操作后循环终止（我开始对这里的描述是有疑虑的，后面保持阶段的循环不变式的论证会消除这个疑虑）。

**初始化阶段：** 在RBInsert调用前，红黑树的所有性质都未违背。RBInsert被调用时，新增一个红节点z和调用了RBInsertFixUp。我们要证明当RBInsertFixUp被调用时，循环不变式的每一部分都成立：

a. 当RBInsertFixUp刚被调用时，z是刚加的红节点，a成立。

b. 当z.parent是根节点，则z.parent最开始是黑色的，在RBInsertFixUp调用前不变，b成立。

c. 性质1,3,5都不会被违背，初始阶段中，性质2也不会违背，因为初始阶段z为根节点时，我的c++代码不会调用RBInsertFixUp。

如果违反了性质4，此时z和z.parent都是红色的，**且没有其他红黑性质被违背**，c成立。（性质4的破坏只是因为z和z.parent都是红色的。）

**保持阶段：**实际需要考虑while循环内的6种情形，由于其中前3种和后3种是对称的。我们只用验证前3种情况（第5行到23行）。**由于初始化阶段已经证明循坏不变式成立，此时我们可以假设本轮迭代开头循环不变式成立（数学归纳法的基本套路）**。那么根据循环不变式b部分（如果z.parent是根，那么z.parent为黑色的），可知z.parent.parent存在。因为只有z.parent为红色才进入一次循环迭代，所以z.parent不可能是根节点，所以z.parent.parent必然存在。

case1 与 case2, case3的区别在于z的uncle, y的颜色。第6行y指向z的uncle, z.parent.parent.rc, 第7行判断y的颜色，如果为红色，则执行case1，否则转向 case2, case3上。在所有3种情况中，**z的祖父节点都是黑色的**（当前z和z.parent都是红色的，性质4已违背，z的祖父节点必须是黑色）。

**case 1: z的uncle是红色的。**

下图显示了case1的情形，这种情况在z.p和y都是红色时发生。因为z.parent和y都是红色时发生。此时z.parent.parent是黑色的，我们要将z.parent和y都着为黑色，以此解决z和z.parent都是红色的问题，将z.parent.parent着色为红色以保持性质5.然后，把z.parent.parent作为新节点z来重复while循环，指针z在树中上移两层

![](./InsertCase1.PNG)

现在，我们要证明执行了case 1后，在下一次循环迭代的开头，循环不变式依然成立。我们用 $z$ 表示当前迭代中的节点z，用 $z^{'} = z.parent.parent$  标识在下一次迭代第3行测试时的节点z。

a. 因为本次迭代z.parent.parent被着色为红色，所以 下次迭代的开始时 $z^{'}$ 是红色的，因此下次迭代的开始时a成立。

b. 节点 $z^{'}.parent$ 的颜色在本次迭代未改变，如果这个节点是根，在这次迭代前为黑色，在下次迭代开始时也是黑色的，因此下次迭代的开始时b成立。

c. 我们已经证明了情况1保持性质5，性质1,3也不会破坏。

​    如果节点 $z^{'}$ 在下一次迭代开始时是根节点，那么case 1修正了性质4的同时破坏了性质2（因为根节点 $z^{'}$ 在本轮迭代中变为红色）。

​	如果节点 $z^{'}$ 在下一次迭代开始时不是根节点，则本次迭代不会修改根节点颜色，那么case 1不会违背性质2。case 1修正了在本次迭代的开始唯一违反性质4，             之后把节点 $z^{'}$ 染红而 不改变 $z^{'}.parent$ 。如果 $z^{'}.parent$ 是黑色，则没违反性质4，也不会进入下次迭代，迭代结束。若 $z^{'}.parent$ 是红色，则再次违反性质4。

​    因此下次迭代的开始时c成立。

**case 2: z的uncle是黑色的且z是一个右孩子。**

**case 3: z的uncle是黑色的且z是一个左孩子。**

代码16到17行构成了 case 2, 它和 case 3一起显示在下图13-6中。

![](./RBInsertFIxUpCase2and3.PNG)

 在case 2中，z是一个右孩子，我们通过一个左旋来将case 2转变为case 3, 此时节点z变为左孩子。此时z和z.parent都是红色的，所以该旋转对节点的黑高和性质5都无影响。无论是直接进入case 2, 还是通过case 3进入case 2，z的uncle y总是黑色的，因为否则就要执行case 1。此外，节点z.parent.parent存在（进入情况2已经经过了第5行的判断），且在第16行将z往上移一层，然后在第17行将z往下移一层后，z.parent.parent的身份依然不变。在case3中，改变某些节点的颜色并做一次右旋，以此保持性质5。此时，z节点所在的路径中不存在两个连续的红色节点（个人认为第三版中文版与第四版英文版在这里都写错了，我修正了一下）。所有的处理到此结束了。此时z.parent是黑色的，所以无需再次执行一次while循环。

现在我们来证明case 2和case 3保持了循环不变式(如刚刚论证的结果，执行完case2和case3, 不会再次执行一次while循环)：

a. case 2让z指向红色的z.parent, 之后在case2 和case3中z的颜色没有改变，下次迭代开始前z是红色（下次迭代不会执行），a成立。

b. case 3将z.parent着色为黑色，此时如果z.parent在下次迭代开始前是根节点，那么根节点依然为黑色，b成立。

c. 性质1,3,5在case 2和case 3中依然保持。

因为节点z在case 2和case 3中不是根（因为节点z有uncle, 所以不可能是根），所以性质2不会被破坏。因为唯一一个着色为红色的节点在case 3中通过右旋变为一个黑色节点的子节点。

情况2和情况3修正了对性质4的违反。由于没有其他红黑性质的违反，此时已是一棵合格的红黑树了。

**终止阶段：**循环终止是因为z.parent是黑色的。此时树不会违反性质4。根据循环不变式，唯一会违反的是性质2，第42行会修复性质2，所以当RBInsertFixUp终止时，所有红黑性质都成立。

此时，我们已证明了RBInsertFixUp能正确地修复所有的红黑性质。

**运行时间分析**

RBInsert中除了RBInsertFixUp的耗时为$O(lgn)$。在RBInsertFixUp中，仅当case1发生，while循环体才会重复执行。所以while循环可能被执行的总次数为$O(lgn)$ ，则RBInsert总耗时依然为$O(lgn)$ ，要注意RBInsertFixUp至多执行2次旋转操作。

## 13.4 删除（遵循第4版英文版）

与插入操作相比，删除操作要稍微复杂些。

从一棵红黑树中删除节点的过程是基于12.3节的TreeDeletion过程而来的。首先和之前BST一样，来个红黑树定制版的RBTransplant:

```c++
template <typename T>
void RBTree<T>::RBTransplant(RBNode<T>* toBeSubstituted, RBNode<T>* vertex) {
    if (toBeSubstituted->parent_ == tNil) root_ = vertex;
    else if (toBeSubstituted == toBeSubstituted->parent_->lc_) toBeSubstituted->parent_->lc_ = vertex;
    else toBeSubstituted->parent_->rc_ = vertex;
    // yes, the assignment to v.p happens unconditionally
    vertex->parent_ = toBeSubstituted->parent_;
}
```

过程RBTransplant和12.3节的Transplant有2点不同：

1. 第3行引用哨兵tNil而不是nullptr。
2. 第7行对vertex.parent的赋值是无条件执行：即使vertex指向哨兵（vertex = tNil），也对 vertex.parent_赋值。

过程RBDelete与12.3节的TreeDeletion类似，但多了几行代码。新增的代码用于记录节点y的踪迹，y有可能导致红黑性质的破坏。当想要删除节点z, 且此时z的子节点少于2个时，z从树中删除，并让y成为z。当z有两个子节点时，y应该是z的后继，并且y将移至树中的z位置。在节点被移除或者在树中移动前，必须记住y的颜色，并记录节点x的踪迹，将x移至树中y的原来位置，因为节点x也可能引起红黑性质的破坏。删除节点z之后，RBDelete调用辅助过程RBDeleteFixUp通过改变颜色和执行旋转来恢复红黑性质。

RBDeletion的c++实现与RBDeletion中的TreeMinimum如下:

```c++
template <typename T>
void RBTree<T>::RBDeletion(RBNode<T>* z) {
    RBNode<T>* y = z;
    RBColor yOriginalColor = y->color_;
    RBNode<T>* x = tNil;
    if (z->lc_ == tNil) {
        x = z->rc_;
        RBTransplant(z, z->rc_); // replace z by its right child
    } else if (z->rc_ == tNil) {
        x = z->lc_;
        RBTransplant(z, z->lc_); // replace z by its left child
    } else {
        y = TreeMinimum(z->rc_); // y is z's successor
        yOriginalColor = y->color_;
        x = y->rc_;
        if (y != z->rc_) {
            RBTransplant(y, y->rc_); // replace y by its right child
            y->rc_ = z->rc_; // z's right child becomes y's right child
            y->rc_->parent_ = y;
        } else x->parent_ = y; // in case x is tNil
        RBTransplant(z, y); // replace z by its successor y
        y->lc_ = z->lc_; // and give z's left child to y
        y->lc_->parent_ = y; // which had no left child
        y->color_ = z->color_;
    }
    if (yOriginalColor == RB_BLACK) RBDeleteFixUp(x); // if any red-back violations occured, correct them
}
template <typename T>
RBNode<T>* RBTree<T>::TreeMinimum(RBNode<T>* cur) const {
    while (cur->lc_ != tNil) {
        cur = cur->lc_;       
    }
    return cur;
}
```

即使RBDelete包含的代码行数几乎是TreeDelete的2倍，但这2个过程具有相同的结构。

下面为两个过程的区别：

1. RBDelete用tNil代替TreeDelete的tNil。

2. 始终维持节点y为从树中删除的节点, 当第3行y指向z,且后续未发生变化，则z至多有1个节点，第12行的情形里z有2个孩子节点，y将指向z的后继，这与TreeDelete相同，y将移至树中z的位置。

3. 由于y的颜色可能改变，变量yOriginalColor存储了y的变化前颜色，第3行和第13行在给y赋值之后，立即设置该变量。当z有两个节点时，则$y \ne z$ 且第21行将y移动到z的位置，第23行将y的颜色着为z的颜色，当y为黑色时，移动y会违反红黑性质。

4. RBDelete保持对x的跟踪，使其移动到y的初始位置上，第7,10,15行令x指向y的唯一孩子或哨兵tNil。

5. 由于x移动到y的原始位置上，属性x.parent总是被设置指向树中y.parent的原始位置，甚至当x是哨兵tNil也是这样（对x.parent的赋值在RBTransplant第7行）。除非z是y的原始parent（此时由于z要被删除且y将占据z的位置，第20行将x.parent设置成y）。我曾认为第20行x.parent指向y是多此一举，由于x已经是y的孩子了，但RBDeleteFixUp依赖x.parent有明确指向，即使x是tNil，所以第20行代码是有必要的。

6. 最后，如果y原本是黑色的，应该有1条或多条红黑性质被破坏，所以在第26行调用RBDeleteFixUp来恢复红黑性质。如果y原本是红色， 有移动前后依然保持红黑性质，原因如下：

   1. 树中黑高没变。（第四版练习13.4-1，证明见附录）
   2. 不存在两个相邻的红节点。因为y会占据原先z的位置，并着色为原先z的颜色，树中的y的新位置（原先z的位置）不存在相邻的红节点。另外，如果y不是z的孩子。y原先的右孩子节点x会占据y的位置，由于y为红色，所以x为黑色，这个替换操作不会导致出现两个红色相连的情况。
   3. y原本为红色，所以y不可能为根节点，根节点保持黑色。

   如果y原本为黑色，会产生三个问题，需要调用RBDeleteFixUp进行矫正：

   1. 如果y最初为根节点，被删除后，y的一个红色节点成为新的根节点，那么违反性质2。
   2. 如果x和x的新的父节点都是红色，那么违反了性质4。
   3. 如果在y所在的简单路径上移动y导致先前包含y的任何简单路径上黑节点个数减少1个，违反原来的性质5。这个问题可以通过把y的黑色传递给x解决，这样x多了一个额外的黑色，如果x原来为黑色，变成双重黑色。如果x原来为红色，变成了红黑双色，这样虽然维护了性质5，但违反了性质1。所以我们应该通过修改颜色来解决这个问题（性质1）。

   现在看RBFDeleteFixUp如何恢复搜索树的红黑性质的。

   其c++实现如下：

   ```c++
   template <typename T>
   void RBTree<T>::RBDeleteFixUp(RBNode<T>* x) {
       while (x != root_ && x->color_ == RB_BLACK) {
           if (x == x->parent_->lc_) {
               RBNode<T>* w = x->parent_->rc_;
               if (w->color_ == RB_RED) {
                   // case 1
                   w->color_ = RB_BLACK;
                   x->parent_->color_ = RB_RED;
                   LeftRotate(x->parent_);
                   w = x->parent_->rc_;
               }
               if (w->lc_->color_ == RB_BLACK && w->rc_->color_ == RB_BLACK) {
                   // case 2
                   w->color_ = RB_RED;
                   x = x->parent_;
               } else {
                   if ( w->rc_->color_ == RB_BLACK) {
                       // case 3
                       w->lc_->color_ = RB_BLACK;
                       w->color_ = RB_RED;
                       RightRotate(w);
                       w = x->parent_->rc_;
                   }
                   // case 4
                   w->color_ = x->parent_->color_;
                   x->parent_->color_ = RB_BLACK;
                   w->rc_->color_ = RB_BLACK;
                   LeftRotate(x->parent_);
                   x = root_;
               }
           } else { // same as above cases, but with "right" and "left" exchanged
               RBNode<T>* w = x->parent_->lc_;
               if (w->color_ == RB_RED) {
                   // case 1
                   w->color_ = RB_BLACK;
                   x->parent_->color_ = RB_RED;
                   RightRotate(x->parent_);
                   w = x->parent_->lc_;
               }
               if (w->rc_->color_ == RB_BLACK && w->lc_->color_ == RB_BLACK) {
                   // case 2
                   w->color_ = RB_RED;
                   x = x->parent_;
               } else {
                   if ( w->rc_->color_ == RB_BLACK) {
                       // case 3
                       w->rc_->color_ = RB_BLACK;
                       w->color_ = RB_RED;
                       LeftRotate(w);
                       w = x->parent_->lc_;
                   }
                   // case 4
                   w->color_ = x->parent_->color_;
                   x->parent_->color_ = RB_BLACK;
                   w->lc_->color_ = RB_BLACK;
                   RightRotate(x->parent_);
                   x = root_;
               }
           }
       }
       x->color_ = RB_BLACK;
   }
   ```

   正文这里优先证明如何恢复性质1，至于恢复性质2和性质4的证明请看附录里Exercise13.4-2和13.4-3的解答。

    RBDeleteFixUp中第4-61行，中while循环的作用就是在树中移动额外的黑色直到以下情况停止：

   1. x指向一个红黑双色节点(我不懂)，此时在第62行，将 x 着为黑色。
   2. x指向根节点，由于根节点已经为黑色，额外的黑色自动消失。
   3. 经过合适的旋转和着色操作，退出循环。

   和RBInsertFixUp类似，RBDeleteFixUp也处理了两种对称情形：第5~31行处理节点x为left child, 第33~60行处理x为right child情形。我们只关注第一种情况，即第4~31行。

   在while循环中，x总指向非根双重黑色的节点（不太懂为啥是双重黑色，黑腿加黑丝？）, 在第4行判断x是否是left child，用w指向x的兄弟，因为x是双重黑节点以及性质5的限制，所以w不可能指向tNil。

   重新想到RBDelete中第17行RBTransplant中或第20行进行了配置x.parent（即使x是哨兵也要配置x.parent)。这是因为RBDeleteFixUp要多次访问x.parent。

   下图13.7描绘了x是left child的四种情况，核心思路就是要维护性质5，从树或者子树的根到子树 $\alpha, \beta, \gamma, \epsilon, \varepsilon, \zeta$ 之间黑色（包含额外黑色）节点个数不变。

   ![](./RBDeletionFixUpCases.PNG)

图13.7(a)展现了情况1，从图中子树【】根节点到子树 $\alpha$ 或 $\beta$ 之间的黑色节点个数在旋转前后均为3（x为双重黑色，统计时+2）， 相似的，从图中子树根节点到子树 $\gamma$ , $\delta$ ，$\epsilon$ , $\varepsilon$ 或 $\zeta$ 之间的黑色节点个数在旋转前后均为2 。

图13.7(b)中必须考虑图中子树【】根节点颜色c的影响，$c=RED$ 表示节点是红黑双色，$c=BLACK$ 表示节点是双重黑色。如果我们定义 $count(RED)=0$  和 $count(BLACK)=1$, 那么从子树【】根节点到子树 $\alpha$ 或 $\beta$ 之间的黑色节点个数为 $2+count(c)$， 且变换前后（case2部分代码被执行）保持不变。在此情况下，变换之后新节点x具有color属性值c, 但是这个节点是红黑（如果 $c=RED$）或者双重黑色的（如果 $c=BLACK$）, 在case2两行代码前和该两行代码执行后, 从子树【】根节点到子树 $\gamma$，$\delta$ , $\varepsilon$或 $\zeta$ 之间的黑色节点个数都为$2+count(c)$（执行前：2个正常黑节点 +根节点$count(c)$，执行后：1个正常黑节点 + 双色根节点（$1+count(c)$） ）。

图13.7(c)中也必须考虑图中子树【】根节点颜色c的影响,那么从子树【】根节点到子树 $\alpha$ 或 $\beta$ 之间的黑色节点个数为 $2+count(c)$，且变换前后保持不变。同样在变换前后，子树【】根节点到子树 $\gamma$ 或 $\delta$ 之间的黑色节点个数为  $1+count(c)$ , 子树【】根节点到子树  $\varepsilon$或 $\zeta$ 之间的黑色节点个数为  $2+count(c)$ 。

图13.7(d)中也必须考虑图中子树【】根节点颜色c和节点C颜色 $c^{'}$ 的影响,那么从子树【】根节点到子树 $\alpha$ 或 $\beta$ 之间的黑色节点个数为 $2+count(c)$，且变换前后保持不变(变换前：x为双重黑色，统计时+2，再加上可能的根节点，变换后 ：2个正常黑节点 +可能的根节点)。同样在变换前后，子树【】根节点到子树 $\gamma$ 或 $\delta$ 之间的黑色节点个数为  $1+count(c)+count(c^{'})$ , 子树【】根节点到子树  $\varepsilon$或 $\zeta$ 之间的黑色节点个数为  $1+count(c)$ 。

   

   

   

   



附录：

Exercise 13.4-1

Show that if node y in RB-DELETE is red, then no black-heights change.

证明：y的原始颜色为红色，得分情形讨论：

case 1 z被彻底删除前，y始终被赋值为 z且z为红色, 该情形下根据性质5和性质4，要被删除的节点z没有内部子节点，删掉z后，z的哨兵子节点tNil替代z, 显然不会影响黑高。

case 2 要被删除节点z有两个内部子节点且z的后继原本为红色， 当z的后继, 即节点y, 为红色时，y的左孩子必然是哨兵tNil(因为y为z的后继)，同样以为性质5和性质4的约束， y的右孩子也必须是哨兵tNil。以此为前提，继续分情形讨论：

​	case 2.1 z的后继为z的右孩子，y替代z, 颜色变为z的颜色，比较删除前以z为根节点的子树和删除后以y为根节点的子树，后者比前者至少了右一个红色节点， 黑高不变。

​    case 2.2 z的后继不为z的右孩子，y的右孩子哨兵tNil替代y后，显然黑高不变。之后 y替代z,且y着色成z的颜色, z的。比较删除前以z为根节点的子树和删除后以y为根节点的子树，后者比前者至少了右一个红色节点， 黑高不变。

综上，如果 RBDelete 中y是红色的，那么没有黑高会发生变化。得证。



程序RBTree的整体c++实现

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
    ~RBTree() { 
        std::cout << "Hi, my job is done. " << "\n";
        int toBeDel = 0;
        auto del =[&toBeDel](RBNode<T>* cur) {
            delete cur;
            cur = nullptr;
            ++toBeDel;
        };
        TravelPreRecursive(root_, del);
        std::cout << "Here is the total num of Deleted nodes : " << toBeDel << std::endl;
        
    }
    // 4.1 recursive version of PreOrder traversal
    template <typename VST>
    void TravelPreRecursive(RBNode<T>* node, VST& visit) {
        if (node == tNil || node == nullptr) return;
        visit(node);  
        TravelPreRecursive(node->lc_, visit);
        TravelPreRecursive(node->rc_, visit); 
    }
    RBNode<T>* TreeMinimum(RBNode<T>* cur) const;
    void LeftRotate(RBNode<T>* x);
    void RightRotate(RBNode<T>* x);
    // insertion
    RBNode<T>* RBInsert(T const& val);
    void RBInsertFixUp(RBNode<T>* z);
    // deletion
    void RBTransplant(RBNode<T>* toBeSubstituted, RBNode<T>* vertex);
    // bool RBDeletion(T const& val);
    void RBDeletion(RBNode<T>* z);
    void RBDeleteFixUp(RBNode<T>* z);
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
RBNode<T>* RBTree<T>::TreeMinimum(RBNode<T>* cur) const {
    while (cur->lc_ != tNil) {
        cur = cur->lc_;       
    }
    return cur;

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
    else toBeSubstituted->parent_->rc_ = vertex;
    // yes, the assignment to v.p happens unconditionally
    vertex->parent_ = toBeSubstituted->parent_;
}
template <typename T>
void RBTree<T>::RBDeletion(RBNode<T>* z) {
    RBNode<T>* y = z;
    RBColor yOriginalColor = y->color_;
    RBNode<T>* x = tNil;
    if (z->lc_ == tNil) {
        x = z->rc_;
        RBTransplant(z, z->rc_); // replace z by its right child
    } else if (z->rc_ == tNil) {
        x = z->lc_;
        RBTransplant(z, z->lc_); // replace z by its left child
    } else {
        y = TreeMinimum(z->rc_); // y is z's successor
        yOriginalColor = y->color_;
        x = y->rc_;
        if (y != z->rc_) {
            RBTransplant(y, y->rc_);
            y->rc_ = z->rc_;
            y->rc_->parent_ = y;
        } else x->parent_ = y;
        RBTransplant(z, y);
        y->lc_ = z->lc_;
        y->lc_->parent_ = y;
        y->color_ = z->color_;
    }
    if (yOriginalColor == RB_BLACK) RBDeleteFixUp(x);
}
template <typename T>
void RBTree<T>::RBDeleteFixUp(RBNode<T>* z) {
    while (x != root_ && x->color_ == RB_BLACK) {
        if (x == x->parent_->lc_) {
            RBNode<T>* w = x->parent_->rc_;
            if (w->color_ == RB_RED) {
                // case 1
                w->color_ = RB_BLACK;
                x->parent_->color_ = RB_RED;
                LeftRotate(x->parent_);
                w = x->parent_->rc_;
            }
            if (w->lc_->color_ == RB_BLACK && w->rc_->color_ == RB_BLACK) {
                // case 2
                w->color_ = RB_RED;
                x = x->parent_;
            } else {
                if ( w->rc_->color_ == RB_BLACK) {
                    // case 3
                    w->lc_->color_ = RB_BLACK;
                    w->color_ = RB_RED;
                    RightRotate(w);
                    w = x->parent_->rc_;
                }
                // case 4
                w->color_ = x->parent_->color_;
                x->parent_->color_ = RB_BLACK;
                w->rc_->color_ = RB_BLACK;
                LeftRotate(x->parent_);
                x = root_;
            }
        } else { // same as above cases, but with "right" and "left" exchanged
            RBNode<T>* w = x->parent_->lc_;
            if (w->color_ == RB_RED) {
                // case 1
                w->color_ = RB_BLACK;
                x->parent_->color_ = RB_RED;
                RightRotate(x->parent_);
                w = x->parent_->lc_;
            }
            if (w->rc_->color_ == RB_BLACK && w->lc_->color_ == RB_BLACK) {
                // case 2
                w->color_ = RB_RED;
                x = x->parent_;
            } else {
                if ( w->rc_->color_ == RB_BLACK) {
                    // case 3
                    w->rc_->color_ = RB_BLACK;
                    w->color_ = RB_RED;
                    LeftRotate(w);
                    w = x->parent_->lc_;
                }
                // case 4
                w->color_ = x->parent_->color_;
                x->parent_->color_ = RB_BLACK;
                w->lc_->color_ = RB_BLACK;
                RightRotate(x->parent_);
                x = root_;
            }
        }
    }
    x->color_ = RB_BLACK;
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

int main()
{

    RBTree<int> rbTree;
    RBNode<int>* node8 = rbTree.RBInsert(8);
    RBNode<int>* node12 = rbTree.RBInsert(12);
    rbTree.RBInsert(5);
    rbTree.RBInsert(3);
    rbTree.RBInsert(7);
    rbTree.RBInsert(13);
    rbTree.RBInsert(10);
    rbTree.RBInsert(11);
    rbTree.RBInsert(9);
    rbTree.RBInsert(16);
    rbTree.Display();
    rbTree.RBDeletion(node12);
    std::cout << "\n";
    std::cout << "\n";
    rbTree.Display();
}
#endif /*BinarySearchTree_h*/
```

运行结果：

```
            L----(3)-RED
      L----(5)-BLK
            R----(7)-RED
(8)-BLK
                  L----(9)-RED
            L----(10)-BLK
                  R----(11)-RED
      R----(12)-RED
            R----(13)-BLK
                  R----(16)-RED


The node with value 12 has been deleted
            L----(3)-RED
      L----(5)-BLK
            R----(7)-RED
(8)-BLK
                  L----(9)-RED
            L----(10)-BLK
                  R----(11)-RED
      R----(13)-RED
            R----(16)-BLK
Hi, my job is done.
Here is the total num of Deleted nodes : 9
```

