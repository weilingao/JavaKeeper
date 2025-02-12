# 程序员心里得有点树——重学数据结构之二叉树

## 前言

> 重学二叉树

## 树

树是一种数据结构，它是由n（n>=0）个有限节点组成一个具有层次关系的集合，存储的是具有“一对多”关系的数据元素的集合。把它叫做“树”是因为它看起来像一棵倒挂的树，也就是说它是根朝上，而叶朝下的

- 除根节点之外的节点被划分为非空集，其中每个节点被称为子树
- 树的节点父子关系，就是姐妹（兄弟）关系
- 在通用树中，一个节点可以具有任意数量的子节点，但它只能有一个父节点
- 下图就是一棵树，节点 `A` 为根节点，而其他节点可以看作是 `A` 的子节点

![tree-demo](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475367871726e6e666a3331393230686f7463762e6a7067.jpeg)



### [基本术语](https://www.bootwiki.com/datastructure/data-structure-tree.html)

- **根节点** ：树中最顶端的节点，根没有父节点（树中至少有一个节点——根）
- **子树**： 如果根节点不为空，则树 `B`，`C` 和 `D` 称为根节点的子树（树中各子树是互不相交的集合）
- **父节点（Parent）**：如果节点拥有子节点，则该节点为子节点的父节点
- **叶节点**：没有子节点的节点，是树的末端节点
- **边（Edge）**：两个节点中间的链接
- **路径**： 连续边的序列称为路径。 在上图所示的树中，节点`E`的路径为`A→B→E`
- **祖先节点**： 节点的祖先是从根到该节点的路径上的任何前节点。根节点没有祖先节点。 在上图所示的树中，节点`F`的祖先是`B`和`A`
- **度**： 节点的度数等于子节点数。 在上图所示的树中，节点`B`的度数为`2`。叶子节点的度数总是`0`，而在完整的二叉树中，每个节点的度数等于`2`
- **高度（Height）/深度（Depth）**：树中层的数量。比如上图中的树有 4 层，则高度为 4
- **级别编号**： 为树的每个节点分配一个级别编号，使得每个节点都存在于高于其父级的一个级别。树的根节点始终是级别`0`。
- **层级（Level）**：根为 Level 0 层，根的子节点为 Level 1 层，以此类推
- 有序树、无序树：如果将树中的各个子树看成是从左到右是有次序的，则称该树是有序树；若不考虑子树的顺序称为无序树
- 森林：m（m>=0）棵互不交互的树的集合。对树中每个结点而言，其子树的集合即为森林

### 基本操作

1. 构造空树（初始化）

2. 销毁树（将树置为空树）

3. 求双亲函数

4. 求孩子节点函数

5. 插入子树

6. 遍历操作

   ......

   [![img](https://camo.githubusercontent.com/8a33cd5c67ca4961456946bac93a996b6f4db0ac0d24c429b6947f15108ed17d/68747470733a2f2f69303170696363646e2e736f676f7563646e2e636f6d2f31633466386664396464333336313061)](https://camo.githubusercontent.com/8a33cd5c67ca4961456946bac93a996b6f4db0ac0d24c429b6947f15108ed17d/68747470733a2f2f69303170696363646e2e736f676f7563646e2e636f6d2f31633466386664396464333336313061)

> 其他都好理解，主要回顾下几种遍历操作，也是面试常客

#### 遍历

遍历的含义就是把树的所有节点（Node）按照**某种顺序**访问一遍。包括**前序**，**中序**，**后续**，**广度优先**（队列），**深度优先**（栈）5 种遍历方法

| 遍历方法 | 顺序                     | 示意图                                                       | 顺序     | 应用                                                         |
| -------- | ------------------------ | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| 前序     | **根 ➜ 左 ➜ 右**         | ![img](https://camo.githubusercontent.com/0104158eb34d17b00348299bde73588e961a54e4a51fc51df50c4e4a3e9f9147/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316768346574647862626e6a333067613066726162662e6a7067) | 12457836 | 想在节点上直接执行操作（或输出结果）使用先序                 |
| 中序     | **左 ➜ 根 ➜ 右**         | [![img](https://camo.githubusercontent.com/5552a974bd7d5979f1e64fb5594f0cc8386eed15ee82fc46c518900139a5cf3d/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316768346635756d6263396a3330686630667261626c2e6a7067)](https://camo.githubusercontent.com/5552a974bd7d5979f1e64fb5594f0cc8386eed15ee82fc46c518900139a5cf3d/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316768346635756d6263396a3330686630667261626c2e6a7067) | 42758136 | 在**二分搜索树**中，中序遍历的顺序符合从小到大（或从大到小）顺序的 要输出排序好的结果使用中序 |
| 后序     | **左 ➜ 右 ➜ 根**         | ![img](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676834666268616673376a3330686e306676676e322e6a7067.jpeg) | 47852631 | 后续遍历的特点是在执行操作时，肯定**已经遍历过该节点的左右子节点** 适用于进行破坏性操作 比如删除所有节点，比如判断树中是否存在相同子树 |
| 广度优先 | **层序，横向访问**       | ![img](https://camo.githubusercontent.com/7e40e2efd626bdf594db5a3d879a82dba1c9faf0988e1a40323240ad574e5500/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676834666664786d71756a333067613066727134372e6a7067) | 12345678 | 当**树的高度非常高**（非常瘦） 使用广度优先剑节省空间        |
| 深度优先 | **纵向，探底到叶子节点** | ![img](https://camo.githubusercontent.com/2e5073e5b981ef95503fefe796247b7ade50586a85326cfa99e79d30d22f31bd/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676834666f67797737656a333067613066727439772e6a7067) | 12457836 | 当**每个节点的子节点非常多**（非常胖），使用深度优先遍历节省空间 （访问顺序和入栈顺序相关，相当于先序遍历） |

> 之所以叫前序、中序、后序遍历，是因为根节点在前、中、后

> 数据结构中有很多树的结构，其中包括二叉树、二叉搜索树、2-3树、红黑树等等。本文中对数据结构中常见的几种树的概念和用途进行了汇总，不求严格精准，但求简单易懂。

## 二叉树

二叉树是每个节点最多有两个子树的树结构

- 二叉树中不存在度大于 2 的结点
- 左子树和右子树是有顺序的，次序不能任意颠倒

根据二叉树的定义和特点，可以将二叉树分为五种不同的形态，如下图所示

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c793167647565757a673234696a33316f3030646b7134722e6a7067.jpeg)

### 二叉树的性质

- 在非空二叉树中，二叉树的第 i 层最多有 2^(i-1) 个结点（i>=1）；
- 深度为 k 的二叉树最多有 2^k – 1 个结点，最少有 k 个结点（k>=1）；
- 对于任意一棵非空二叉树如果其叶结点数为 n0，而度为 2 的非叶结点总数为 n2，则 n0=n2+1；
- 具有 n (n>=0) 个结点的完全二叉树的深度为 log2(n) +1；
- 任意一棵二叉树，其节点个数等于分支个数加 1，即 n=B+1

### 两个特别的二叉树

- 满二叉树：如果二叉树中除了叶子结点，每个结点的度都为 2，则此二叉树称为满二叉树，也叫严格二叉树
  - 二叉树中第 i 层的节点数为 2^(n-1) 个。
  - 深度为 k 的满二叉树必有 2^(k-1) 个节点 ，叶子数为 2^(k-1)
  - 满二叉树中不存在度为 1 的节点，每一个分支点中都两棵深度相同的子树，且叶子节点都在最底层
  - 具有 n 个节点的满二叉树的深度为 log2(n+1)
- **完全二叉树**：若设二叉树的深度为 h，除第 h 层外，其它各层 (1～(h-1)层) 的结点数都达到最大个数，第 h 层所有的结点都连续集中在最左边，这就是完全二叉树。

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316764756238786a6f39656a333139643069323431372e6a7067.jpeg)

**满二叉树一定是一颗棵完全二叉树，但完全二叉树不一定是满二叉树。**

- 其实还有一种更特殊的二叉树：**斜树**，顾名思义，就是斜着长的，分为左斜树和右斜树。（线性表结构可以理解为是树的一种极其特殊的表现形式）

### 常见的存储方法

二叉树的存储结构有两种，分别为顺序存储和链式存储。

#### 二叉树的顺序存储结构

二叉树的顺序存储，指的是使用顺序表（数组）存储二叉树。需要注意的是，顺序存储只适用于完全二叉树。换句话说，只有完全二叉树才可以使用顺序表存储。因此，如果我们想顺序存储普通二叉树，需要提前将普通二叉树转化为完全二叉树。

完全二叉树的顺序存储，仅需从根节点开始，按照层次依次将树中节点存储到数组即可。

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475386964617536726a33307479306d636468752e6a7067.jpeg)

普通二叉树转完全二叉树，只需给二叉树额外添加一些节点，将其"拼凑"成完全二叉树即可。

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475387062367561366a3330776f306d633736382e6a7067.jpeg)

#### 二叉树的链式存储结构

并不是每个二叉树都是完全二叉树，普通二叉树使用顺序表存储或多或少会存在空间浪费的现象，所以就有了链式存储结构。

二叉树的链式存储结构就是用链表来表示一棵二叉树，即用链来指示着元素的逻辑关系。通常有下面两种形式。

##### 二叉链表存储

链表中每个结点由三个域组成，除了数据域外，还有两个指针域，分别用来给出该结点左孩子和右孩子所在的链结点的存储地址。

其中，data 域存放某结点的数据信息；lchild 与 rchild 分别存放指向左孩子和右孩子的指针，当左孩子或右孩子不存在时，相应指针域值为空（用符号 ∧ 或 NULL 表示）。

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475623767656f6d396a33316530306a6f37366f2e6a7067.jpeg)

##### 三叉链表存储

为了方便找到父节点，可以在上述结点结构中增加一个指针域，指向结点的父结点。利用此结点结构得到的二叉树存储结构称为三叉链表。

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316764756238336d6430386a33316b79306c636164392e6a7067.jpeg)

### 二叉树的基本操作

二叉树的遍历方式主要有：先序遍历、中序遍历、后序遍历、层次遍历。

关于应用部分，选择遍历方法的基本的原则：**更快的访问到你想访问的节点**。先序会先访问根节点，后序会先访问叶子节点

> coding 部分，下一篇结合 leetcode 常见的二叉树算法题，再一并说下二叉树的建立、递归等操作

------

## 二叉查找树

**二叉查找树定义**：又称为二叉排序树（Binary Sort Tree）或二叉搜索树。二叉排序树要么是一棵空树，要么是具有如下性质的二叉树：

1. 若它的左子树不为空，则左子树上所有结点的值均小于它的根结点的值
2. 若它的右子树不为空，则右子树上所有结点的值均大于它的根结点的值
3. 左、右子树也分别为二叉排序树
4. 没有键值相等的节点

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475686674657968386a3331366b306a716469792e6a7067.jpeg)

### 性质

二叉查找树是一个递归的数据结构。对二叉查找树进行中序遍历，即可得到有序的数列。

### 时间复杂度

它和二分查找一样，插入和查找的时间复杂度均为 $O(log2n)$，但是在最坏的情况下仍然会有 $O(n)$ 的时间复杂度。原因在于插入和删除元素的时候，树没有保持平衡。我们追求的是在最坏的情况下仍然有较好的时间复杂度，这就是平衡查找树设计的初衷。

**二叉查找树的高度决定了二叉查找树的查找效率。**

### 基本操作

在实行基本操作之前，我们需要先定义一下基本数据类型：

```java
class TreeNode<E extends Comparable<E>>{
    private E data;
    private TreeNode<E> left;
    private TreeNode<E> right;
    TreeNode(E theData){
        data = theData;
        left = null;
        right = null;
    }
}
public class BinarySearchTree<E extends Comparable<E>>{
    private TreeNode<E> root = null;
}
```

#### 1. 树的遍历：

假设我们需要遍历树中所有节点，这里有许多递归方法可以实现：

**1.中序遍历：当到达某个节点时，先访问左子节点，再输出该节点，最后访问右子节点。**

```java
public void inOrder(TreeNode<E> cursor){
    if(cursor == null) return;
    inOrder(cursor.getLeft());
    System.out.println(cursor.getData());
    inOrder(cursor.getRight());
}
```

**2. 前序遍历：当到达某个节点时，先输出该节点，再访问左子节点，最后访问右子节点。**

```java
public void preOrder(TreeNode<E> cursor){
    if(cursor == null) return;
    System.out.println(cursor.getData());
    inOrder(cursor.getLeft());
    inOrder(cursor.getRight());
}
```

**3. 后序遍历：当到达某个节点时，先访问左子节点，再访问右子节点，最后输出该节点。**

```java
public void postOrder(TreeNode<E> cursor){
    if(cursor == null) return;
    inOrder(cursor.getLeft());
    inOrder(cursor.getRight());
    System.out.println(cursor.getData());
}
```

#### 2. 树的搜索：

树的搜索和树的遍历差不多，就是在遍历的时候只搜索不输出就可以了（类比有序数组的搜索）

![图片来源：penjee.com](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475686c787475617567333063693061697462732e676966.gif)

```java
public boolean searchNode(TreeNode<E> node){
    TreeNode<E> currentNode = root;
    while(true){
        if(currentNode == null){
            return false;
        }
        if(currentNode.getData().compareTo(node.getData()) == 0){
            return true;
        }else if(currentNode.getData().compareTo(node.getData()) < 0){
            currentNode = currentNode.getLeft();
        }else{
            currentNode = currentNode.getRight();
        }
    }
}
```

#### 3. 节点插入：

步骤：

1. 递归地去查找该二叉树，找到应该插入的节点
2. 若当前的二叉查找树为空，则插入的元素为根节点
3. 若插入的元素值小于根节点值，则将元素插入到左子树中
4. 若插入的元素值不小于根节点值，则将元素插入到右子树中

![图片来源：penjee.com](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475686d366736317667333062343037336a76382e676966.gif)

```java
public void insertNode(TreeNode<E> node){
    TreeNode<E> currentNode = root;
    if(currentNode == null){
        root = node;
        return;
    }else{
        while(true){
            if(node.getData().compareTo(currentNode.getData()) < 0){
                if(currentNode.getLeft() == null){
                    break;
                }else{
                    currentNode = currentNode.getLeft();
                }
            }else if(node.getData().compareTo(currentNode.getData()) > 0){
            
                if(currentNode.getRight() == null){
                    break;
                }else{
                    currentNode = currentNode.getRight();
                }
            }
        }   
    }
    if(node.getData().compareTo(currentNode.getData()) < 0){
        currentNode.setLeft(node);
    }else if(node.getData().compareTo(currentNode.getData()) > 0){
        currentNode.setRight(node);
    }
}
```

#### 4. 节点删除：

首先需要搜索该节点，然后可以分为以下四种情况进行讨论：

- 如果找不到该节点，那么什么都不用做

- 如果被移除的元素在叶节点(no children)：那么直接移除该节点，并且将父节点原本指向该位置改为 null (如果是根节点，那就不用修改父节点指向位置)

- 如果删除的元素只有一个儿子(one child)：那么也很简单，直接删除该节点，并且将父节点原本指向的位置改为该儿子 (如果是根节点，那么该儿子成为新的根节点)

  例如：要在树中删除元素 20

![img](https://camo.githubusercontent.com/5ee19e7c662a5ca1afcc587f37a8869072afdfca2176960ddaa4c9ef8140e760/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c793167647569317663333773673330677a30357a6a73772e676966)

- 如果删除的元素有两个儿子，那么可以取左子树中最大元素或者右子树中最小元素进行替换，然后将最大元素最小元素原位置置空

  例如：要在树中删除元素 15

![](https://camo.githubusercontent.com/f5c03694805a1537f87758951017e1ba75f33f0250393a6a571ec200b67950e1/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475693277766b3461673330677a30357a6162722e676966)

- 有序数组转为二叉查找树

![图片来源：penjee.com](https://camo.githubusercontent.com/510f4796320d1381ef161d7cdceafdace3ce6e6a6e2c70affd9c65d69fc39bef/68747470733a2f2f626c6f672e70656e6a65652e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031352f31322f6f7074696d616c2d62696e6172792d7365617263682d747265652d66726f6d2d736f727465642d61727261792e676966)

- 将二叉树转为有序数组

![图片来源：penjee.com](https://camo.githubusercontent.com/665c1d5eb211cfb69a8209a52f79bdb95b0fc94742c83b7e63ceeac27e349b72/68747470733a2f2f626c6f672e70656e6a65652e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031352f31312f62696e6172792d7365617263682d747265652d646567656e65726174696e672d64656d6f2d616e696d6174696f6e2e676966)

------

## 平衡二叉树

二叉搜索树虽然在插入和删除时的效率都有所提升，但是如果原序列有序时，比如 {3,4,5,6,7}，这个时候构造二叉树搜索树就变成了斜树，二叉树退化成单链表，搜索效率降低到 $O(n)$，查找数字 7 的话，需要找 5 次。这又说明了**二叉查找树的高度决定了二叉查找树的查找效率**。

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c7931676475696d617735356a6a33303838303874676d302e6a7067.jpeg)

为了解决这一问题，两位科学家大爷，G. M. Adelson-Velsky 和 E. M. Landis 又发明了平衡二叉树，从他两名字中提取出了 AVL，所以平衡二叉树又叫 **AVL 树**。

二叉搜索树的查找效率取决于树的高度，所以保持树的高度最小，就可保证树的查找效率，如下保持左右平衡，像不像天平？

![](https://cdn.jsdelivr.net/gh/Jstarfish/picBed/datastrucutre/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316764756b76707a3779316a3330386f3037373734702e6a7067.jpeg)

![](https://camo.githubusercontent.com/9d7104e88f2a6f400d8aed0bf35cf045e48e69b60304967f59c4bed7c493fcd0/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316764756b75366179766e6a3330386f3037353074352e6a7067)

**定义**：

平衡二叉查找树，简称平衡二叉树，指的是左子树上的所有节点的值都比根节点的值小，而右子树上的所有节点的值都比根节点的值大，且左子树与右子树的高度差最大为1。因此，平衡二叉树满足所有二叉排序（搜索）树的性质：

- 可以是空树
- 假如不是空树，**任何一个结点的左子树与右子树都是平衡二叉树**，并且高度之差的绝对值不超过 1

**平衡因子**：

某节点的左子树与右子树的高度(深度)差即为该节点的平衡因子（BF,Balance Factor），平衡二叉树中不存在平衡因子大于 1 的节点。在一棵平衡二叉树中，节点的平衡因子只能取 0 、1 或者 -1 ，分别对应着左右子树等高，左子树比较高，右子树比较高。

在AVL中任何节点的两个儿子子树的高度最大差别为1，所以它也被称为高度平衡树，n个结点的AVL树最大深度约1.44log2n。查找、插入和删除在平均和最坏情况下都是O(logn)。增加和删除可能需要通过一次或多次树旋转来重新平衡这个树。**这个方案很好的解决了二叉查找树退化成链表的问题，把插入，查找，删除的时间复杂度最好情况和最坏情况都维持在O(logN)。但是频繁旋转会使插入和删除牺牲掉O(logN)左右的时间，不过相对二叉查找树来说，时间上稳定了很多。**

平衡二叉树（AVL树）在符合二叉查找树的条件下，还满足任何节点的两个子树的高度最大差为1。下面的两张图片，左边是AVL树，它的任何节点的两个子树的高度差<=1；右边的不是AVL树，其根节点的左子树高度为3，而右子树高度为1； [![索引](https://camo.githubusercontent.com/8e50f7d9828f1e438edcb97b04cb051542d3ad7fa588dd3220aeeb14eed3010d/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323033353534363633)](https://camo.githubusercontent.com/8e50f7d9828f1e438edcb97b04cb051542d3ad7fa588dd3220aeeb14eed3010d/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323033353534363633)

如果在AVL树中进行插入或删除节点，可能导致AVL树失去平衡，这种失去平衡的二叉树可以概括为四种姿态：LL（左左）、RR（右右）、LR（左右）、RL（右左）。它们的示意图如下： [![索引](https://camo.githubusercontent.com/b67a94873e0ca508363d711117242d0f7f295086fd764f2097f400f189c5ccdb/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323033363438313438)](https://camo.githubusercontent.com/b67a94873e0ca508363d711117242d0f7f295086fd764f2097f400f189c5ccdb/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323033363438313438)

这四种失去平衡的姿态都有各自的定义： LL：LeftLeft，也称“左左”。插入或删除一个节点后，根节点的左孩子（Left Child）的左孩子（Left Child）还有非空节点，导致根节点的左子树高度比右子树高度高2，AVL树失去平衡。

RR：RightRight，也称“右右”。插入或删除一个节点后，根节点的右孩子（Right Child）的右孩子（Right Child）还有非空节点，导致根节点的右子树高度比左子树高度高2，AVL树失去平衡。

LR：LeftRight，也称“左右”。插入或删除一个节点后，根节点的左孩子（Left Child）的右孩子（Right Child）还有非空节点，导致根节点的左子树高度比右子树高度高2，AVL树失去平衡。

RL：RightLeft，也称“右左”。插入或删除一个节点后，根节点的右孩子（Right Child）的左孩子（Left Child）还有非空节点，导致根节点的右子树高度比左子树高度高2，AVL树失去平衡。

AVL树失去平衡之后，可以通过旋转使其恢复平衡。下面分别介绍四种失去平衡的情况下对应的旋转方法。

LL的旋转。LL失去平衡的情况下，可以通过一次旋转让AVL树恢复平衡。步骤如下：

1. 将根节点的左孩子作为新根节点。
2. 将新根节点的右孩子作为原根节点的左孩子。
3. 将原根节点作为新根节点的右孩子。

LL旋转示意图如下： [![索引](https://camo.githubusercontent.com/6850de62dc5ddb0126755ce28bdd117752132cf9749fa8d527068bc5d9af46de/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034313133393934)](https://camo.githubusercontent.com/6850de62dc5ddb0126755ce28bdd117752132cf9749fa8d527068bc5d9af46de/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034313133393934)

RR的旋转：RR失去平衡的情况下，旋转方法与LL旋转对称，步骤如下：

1. 将根节点的右孩子作为新根节点。
2. 将新根节点的左孩子作为原根节点的右孩子。
3. 将原根节点作为新根节点的左孩子。

RR旋转示意图如下： [![索引](https://camo.githubusercontent.com/910e0d2207881d121608d1efb0b1121732bb8915f5a6a38f8803f1b162d6a0fd/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034323037393633)](https://camo.githubusercontent.com/910e0d2207881d121608d1efb0b1121732bb8915f5a6a38f8803f1b162d6a0fd/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034323037393633)

LR的旋转：LR失去平衡的情况下，需要进行两次旋转，步骤如下：

1. 围绕根节点的左孩子进行RR旋转。
2. 围绕根节点进行LL旋转。

LR的旋转示意图如下： [![索引](https://camo.githubusercontent.com/ef203ace5934366c04ddc854bd7f7a57670f549f8c0b12f28ab5096993259887/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034323537333639)](https://camo.githubusercontent.com/ef203ace5934366c04ddc854bd7f7a57670f549f8c0b12f28ab5096993259887/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034323537333639)

RL的旋转：RL失去平衡的情况下也需要进行两次旋转，旋转方法与LR旋转对称，步骤如下：

1. 围绕根节点的右孩子进行LL旋转。
2. 围绕根节点进行RR旋转。

RL的旋转示意图如下： [![索引](https://camo.githubusercontent.com/36a25db63c2608975c3c40ea85e63bfca51e6601f236721489f07be41bce3684/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034333331303733)](https://camo.githubusercontent.com/36a25db63c2608975c3c40ea85e63bfca51e6601f236721489f07be41bce3684/68747470733a2f2f696d672d626c6f672e6373646e2e6e65742f3230313630323032323034333331303733)

】

### AVL树插入时的失衡与调整

平衡二叉树大部分操作和二叉查找树类似，主要不同在于插入删除的时候平衡二叉树的平衡可能被改变，当平衡因子的绝对值大于1的时候，我们就需要对其进行旋转来保持平衡。

### AVL树的四种插入节点方式

假设一颗 AVL 树的某个节点为 A，有四种操作会使 A 的左右子树高度差大于 1，从而破坏了原有 AVL 树的平衡性：

1. 在 A 的左儿子的左子树进行一次插入
2. 对 A 的左儿子的右子树进行一次插入
3. 对 A 的右儿子的左子树进行一次插入
4. 对 A 的右儿子的右子树进行一次插入

![](https://camo.githubusercontent.com/ff977c44979a77ea9ed8f8cdcc0d5ca529bf0196333b0b02321ba845e613340d/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6c79316764756c67686a7130366a33316673306a676469342e6a7067)

情形 1 和情形 4 是关于 A 的镜像对称，情形 2 和情形 3 也是关于 A 的镜像对称，因此理论上看只有两种情况，但编程的角度看还是四种情形。

第一种情况是插入发生在“外边”的情形（左左或右右），该情况可以通过一次单旋转完成调整；第二种情况是插入发生在“内部”的情形（左右或右左），这种情况比较复杂，需要通过双旋转来调整。

单旋转又有左旋和右旋之分

#### 左旋

情景 1 对 A 的左二子的左子树插入节点，只需要一次右旋即可。

https://www.zhihu.com/search?type=content&q=平衡二叉树

https://www.cxyxiaowu.com/1696.html

https://www.cnblogs.com/zhangbaochong/p/5164994.html

### AVL树的四种删除节点方式

AVL 树和二叉查找树的删除操作情况一致，都分为四种情况：

1. 删除叶子节点
2. 删除的节点只有左子树
3. 删除的节点只有右子树
4. 删除的节点既有左子树又有右子树

只不过 AVL 树在删除节点后需要重新检查平衡性并修正，同时，删除操作与插入操作后的平衡修正区别在于，插入操作后只需要对插入栈中的弹出的第一个非平衡节点进行修正，而删除操作需要修正栈中的所有非平衡节点。

删除操作的大致步骤如下：

- 以前三种情况为基础尝试删除节点，并将访问节点入栈。
- 如果尝试删除成功，则依次检查栈顶节点的平衡状态，遇到非平衡节点，即进行旋转平衡，直到栈空。
- 如果尝试删除失败，证明是第四种情况。这时先找到被删除节点的右子树最小节点并删除它，将访问节点继续入栈。
- 再依次检查栈顶节点的平衡状态和修正直到栈空。

对于删除操作造成的非平衡状态的修正，可以这样理解：对左或者右子树的删除操作相当于对右或者左子树的插入操作，然后再对应上插入的四种情况选择相应的旋转就好了。

## 红黑树

**红黑树的定义：**红黑树是一种自平衡二叉查找树，是在计算机科学中用到的一种数据结构，典型的用途是实现关联数组。它是在1972年由鲁道夫·贝尔发明的，称之为"对称二叉B树"，它现代的名字是在 Leo J. Guibas 和 Robert Sedgewick 于1978年写的一篇论文中获得的。**它是复杂的，但它的操作有着良好的最坏情况运行时间，并且在实践中是高效的: 它可以在O(logn)时间内做查找，插入和删除，这里的n是树中元素的数目。**

红黑树和AVL树一样都对插入时间、删除时间和查找时间提供了最好可能的最坏情况担保。这不只是使它们在时间敏感的应用如实时应用（real time application）中有价值，而且使它们有在提供最坏情况担保的其他数据结构中作为建造板块的价值；例如，在计算几何中使用的很多数据结构都可以基于红黑树。此外，红黑树还是2-3-4树的一种等同，它们的思想是一样的，只不过红黑树是2-3-4树用二叉树的形式表示的。

**红黑树的性质：**

红黑树是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。在二叉查找树强制的一般要求以外，对于任何有效的红黑树我们增加了如下的额外要求:

- 每个节点要么是黑色，要么是红色
- 根节点是黑色
- 所有叶子都是黑色（叶子是NIL节点）
- 每个红色节点必须有两个黑色的子节点(从每个叶子到根的所有路径上不能有两个连续的红色节点)
- **任意一结点到每个叶子结点的简单路径都包含数量相同的黑结点**

下面是一个具体的红黑树的图例：

[![An example of a red-black tree](https://camo.githubusercontent.com/5db2fdaae07fbac460e1ef727d9b890ce292cbcb33783efece8fc2fad30006a8/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f7468756d622f362f36362f5265642d626c61636b5f747265655f6578616d706c652e7376672f34353070782d5265642d626c61636b5f747265655f6578616d706c652e7376672e706e67)](https://camo.githubusercontent.com/5db2fdaae07fbac460e1ef727d9b890ce292cbcb33783efece8fc2fad30006a8/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f7468756d622f362f36362f5265642d626c61636b5f747265655f6578616d706c652e7376672f34353070782d5265642d626c61636b5f747265655f6578616d706c652e7376672e706e67)

这些约束确保了红黑树的关键特性: 从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。结果是这个树大致上是平衡的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

要知道为什么这些性质确保了这个结果，注意到性质4导致了路径不能有两个毗连的红色节点就足够了。最短的可能路径都是黑色节点，最长的可能路径有交替的红色和黑色节点。因为根据性质5所有最长的路径都有相同数目的黑色节点，这就表明了没有路径能多于任何其他路径的两倍长。

**红黑树的自平衡操作：**

因为每一个红黑树也是一个特化的二叉查找树，因此红黑树上的只读操作与普通二叉查找树上的只读操作相同。然而，在红黑树上进行插入操作和删除操作会导致不再符合红黑树的性质。恢复红黑树的性质需要少量(O(logn))的颜色变更(实际是非常快速的)和不超过三次树旋转(对于插入操作是两次)。虽然插入和删除很复杂，但操作时间仍可以保持为O(logn) 次。

**我们首先以二叉查找树的方法增加节点并标记它为红色。如果设为黑色，就会导致根到叶子的路径上有一条路上，多一个额外的黑节点，这个是很难调整的（违背性质5）。**但是设为红色节点后，可能会导致出现两个连续红色节点的冲突，那么可以通过颜色调换（color flips）和树旋转来调整。下面要进行什么操作取决于其他临近节点的颜色。同人类的家族树中一样，我们将使用术语叔父节点来指一个节点的父节点的兄弟节点。注意:

- 性质1和性质3总是保持着。
- 性质4只在增加红色节点、重绘黑色节点为红色，或做旋转时受到威胁。
- 性质5只在增加黑色节点、重绘红色节点为黑色，或做旋转时受到威胁。

　　**插入操作：**

　　假设，将要插入的节点标为**N**，N的父节点标为**P**，N的祖父节点标为**G**，N的叔父节点标为**U**。在图中展示的任何颜色要么是由它所处情形这些所作的假定，要么是假定所暗含的。

　　**情形1:** 该树为空树，直接插入根结点的位置，违反性质1，把节点颜色由红改为黑即可。

　　**情形2:** 插入节点N的父节点P为黑色，不违反任何性质，无需做任何修改。在这种情形下，树仍是有效的。性质5也未受到威胁，尽管新节点N有两个黑色叶子子节点；但由于新节点N是红色，通过它的每个子节点的路径就都有同通过它所取代的黑色的叶子的路径同样数目的黑色节点，所以依然满足这个性质。

　　注： 情形1很简单，情形2中P为黑色，一切安然无事，但P为红就不一样了，下边是P为红的各种情况，也是真正难懂的地方。

　　**情形3:** 如果父节点P和叔父节点U二者都是红色，(此时新插入节点N做为P的左子节点或右子节点都属于情形3,这里右图仅显示N做为P左子的情形)则我们可以将它们两个重绘为黑色并重绘祖父节点G为红色(用来保持性质4)。现在我们的新节点N有了一个黑色的父节点P。因为通过父节点P或叔父节点U的任何路径都必定通过祖父节点G，在这些路径上的黑节点数目没有改变。但是，红色的祖父节点G的父节点也有可能是红色的，这就违反了性质4。为了解决这个问题，我们在祖父节点G上递归地进行上述情形的整个过程（把G当成是新加入的节点进行各种情形的检查）。比如，G为根节点，那我们就直接将G变为黑色（情形1）；如果G不是根节点，而它的父节点为黑色，那符合所有的性质，直接插入即可（情形2）；如果G不是根节点，而它的父节点为红色，则递归上述过程（情形3）。

[![img](https://camo.githubusercontent.com/4d3f371b7365cc94d8c9a881f1d750b8d0b72a22b17a4eba9648714370531e8c/68747470733a2f2f7069633030322e636e626c6f67732e636f6d2f696d616765732f323031312f3333303731302f323031313132303131363432353235312e706e67)](https://camo.githubusercontent.com/4d3f371b7365cc94d8c9a881f1d750b8d0b72a22b17a4eba9648714370531e8c/68747470733a2f2f7069633030322e636e626c6f67732e636f6d2f696d616765732f323031312f3333303731302f323031313132303131363432353235312e706e67)

　　**情形4:** 父节点P是红色而叔父节点U是黑色或缺少，新节点N是其父节点的左子节点，而父节点P又是其父节点G的左子节点。在这种情形下，我们进行针对祖父节点G的一次右旋转; 在旋转产生的树中，以前的父节点P现在是新节点N和以前的祖父节点G的父节点。我们知道以前的祖父节点G是黑色，否则父节点P就不可能是红色(如果P和G都是红色就违反了性质4，所以G必须是黑色)。我们切换以前的父节点P和祖父节点G的颜色，结果的树满足性质4。性质5也仍然保持满足，因为通过这三个节点中任何一个的所有路径以前都通过祖父节点G，现在它们都通过以前的父节点P。在各自的情形下，这都是三个节点中唯一的黑色节点。

[![情形5 示意图](https://camo.githubusercontent.com/6589a9c75d8e686270a6f2e981c1dd7ea274ce0c68064861e205ee259dc0a164/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f362f36362f5265642d626c61636b5f747265655f696e736572745f636173655f352e706e67)](https://camo.githubusercontent.com/6589a9c75d8e686270a6f2e981c1dd7ea274ce0c68064861e205ee259dc0a164/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f362f36362f5265642d626c61636b5f747265655f696e736572745f636173655f352e706e67)

　　**情形5:** 父节点P是红色而叔父节点U是黑色或缺少，并且新节点N是其父节点P的右子节点而父节点P又是其父节点的左子节点。在这种情形下，我们进行一次左旋转调换新节点和其父节点的角色; 接着，我们按**情形4**处理以前的父节点P以解决仍然失效的性质4。注意这个改变会导致某些路径通过它们以前不通过的新节点N（比如图中1号叶子节点）或不通过节点P（比如图中3号叶子节点），但由于这两个节点都是红色的，所以性质5仍有效。

[![情形4 示意图](https://camo.githubusercontent.com/b3607bd07118241f8f16f2cc07603423bb3bfcae73c203434d0633a4bd3cdd60/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f352f35362f5265642d626c61636b5f747265655f696e736572745f636173655f342e706e67)](https://camo.githubusercontent.com/b3607bd07118241f8f16f2cc07603423bb3bfcae73c203434d0633a4bd3cdd60/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f352f35362f5265642d626c61636b5f747265655f696e736572745f636173655f342e706e67)

　　**注: 插入实际上是原地算法，因为上述所有调用都使用了尾部递归。**

　　**删除操作：**

　　**如果需要删除的节点有两个儿子，那么问题可以被转化成删除另一个只有一个儿子的节点的问题**。对于二叉查找树，在删除带有两个非叶子儿子的节点的时候，我们找到要么在它的左子树中的最大元素、要么在它的右子树中的最小元素，并把它的值转移到要删除的节点中。我们接着删除我们从中复制出值的那个节点，它必定有少于两个非叶子的儿子。因为只是复制了一个值，不违反任何性质，这就把问题简化为如何删除最多有一个儿子的节点的问题。它不关心这个节点是最初要删除的节点还是我们从中复制出值的那个节点。

　　**我们只需要讨论删除只有一个儿子的节点**(如果它两个儿子都为空，即均为叶子，我们任意将其中一个看作它的儿子)。如果我们删除一个红色节点（此时该节点的儿子将都为叶子节点），它的父亲和儿子一定是黑色的。所以我们可以简单的用它的黑色儿子替换它，并不会破坏性质3和性质4。通过被删除节点的所有路径只是少了一个红色节点，这样可以继续保证性质5。另一种简单情况是在被删除节点是黑色而它的儿子是红色的时候。如果只是去除这个黑色节点，用它的红色儿子顶替上来的话，会破坏性质5，但是如果我们重绘它的儿子为黑色，则曾经通过它的所有路径将通过它的黑色儿子，这样可以继续保持性质5。

　　**需要进一步讨论的是在要删除的节点和它的儿子二者都是黑色的时候**，这是一种复杂的情况。我们首先把要删除的节点替换为它的儿子。出于方便，称呼这个儿子为**N**(在新的位置上)，称呼它的兄弟(它父亲的另一个儿子)为**S**。在下面的示意图中，我们还是使用**P**称呼N的父亲，**SL**称呼S的左儿子，**SR**称呼S的右儿子。

　　如果N和它初始的父亲是黑色，则删除它的父亲导致通过N的路径都比不通过它的路径少了一个黑色节点。因为这违反了性质5，树需要被重新平衡。有几种情形需要考虑:

　　**情形1:** N是新的根。在这种情形下，我们就做完了。我们从所有路径去除了一个黑色节点，而新根是黑色的，所以性质都保持着。

　　**注意**: 在情形2、5和6下，我们假定N是它父亲的左儿子。如果它是右儿子，则在这些情形下的左和右应当对调。

　　**情形2:** S是红色。在这种情形下我们在N的父亲上做左旋转，把红色兄弟转换成N的祖父，我们接着对调N的父亲和祖父的颜色。完成这两个操作后，尽管所有路径上黑色节点的数目没有改变，但现在N有了一个黑色的兄弟和一个红色的父亲（它的新兄弟是黑色因为它是红色S的一个儿子），所以我们可以接下去按**情形4**、**情形5**或**情形6**来处理。

[![情形2 示意图](https://camo.githubusercontent.com/e91996abee214a19db6edb37d13288a5bcb97b3decd3c51e96a55b139c965da7/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33392f5265642d626c61636b5f747265655f64656c6574655f636173655f322e706e67)](https://camo.githubusercontent.com/e91996abee214a19db6edb37d13288a5bcb97b3decd3c51e96a55b139c965da7/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33392f5265642d626c61636b5f747265655f64656c6574655f636173655f322e706e67)

　　**情形3:** N的父亲、S和S的儿子都是黑色的。在这种情形下，我们简单的重绘S为红色。结果是通过S的所有路径，它们就是以前*不*通过N的那些路径，都少了一个黑色节点。因为删除N的初始的父亲使通过N的所有路径少了一个黑色节点，这使事情都平衡了起来。但是，通过P的所有路径现在比不通过P的路径少了一个黑色节点，所以仍然违反性质5。要修正这个问题，我们要从**情形1**开始，在P上做重新平衡处理。

[![情形3 示意图](https://camo.githubusercontent.com/11d59ddec9bba2b6f6fd119e7c3a3504ab8a14a99fbf48c7f75c5016f0bd8fd7/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f632f63372f5265642d626c61636b5f747265655f64656c6574655f636173655f332e706e67)](https://camo.githubusercontent.com/11d59ddec9bba2b6f6fd119e7c3a3504ab8a14a99fbf48c7f75c5016f0bd8fd7/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f632f63372f5265642d626c61636b5f747265655f64656c6574655f636173655f332e706e67)

　　**情形4:** S和S的儿子都是黑色，但是N的父亲是红色。在这种情形下，我们简单的交换N的兄弟和父亲的颜色。这不影响不通过N的路径的黑色节点的数目，但是它在通过N的路径上对黑色节点数目增加了一，添补了在这些路径上删除的黑色节点。

[![情形4 示意图](https://camo.githubusercontent.com/6b648caabf6e7caf7b156f24482634f5e77bbd1c565a77b0be67f0af6eeaa350/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f642f64372f5265642d626c61636b5f747265655f64656c6574655f636173655f342e706e67)](https://camo.githubusercontent.com/6b648caabf6e7caf7b156f24482634f5e77bbd1c565a77b0be67f0af6eeaa350/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f642f64372f5265642d626c61636b5f747265655f64656c6574655f636173655f342e706e67)

　　**情形5:** S是黑色，S的左儿子是红色，S的右儿子是黑色，而N是它父亲的左儿子。在这种情形下我们在S上做右旋转，这样S的左儿子成为S的父亲和N的新兄弟。我们接着交换S和它的新父亲的颜色。所有路径仍有同样数目的黑色节点，但是现在N有了一个黑色兄弟，他的右儿子是红色的，所以我们进入了**情形6**。N和它的父亲都不受这个变换的影响。

[![情形5 示意图](https://camo.githubusercontent.com/ddae9cf5c9133ed4f91d7146251d39f9837dfe9dd2342eb2c8973fea3fe0694e/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33302f5265642d626c61636b5f747265655f64656c6574655f636173655f352e706e67)](https://camo.githubusercontent.com/ddae9cf5c9133ed4f91d7146251d39f9837dfe9dd2342eb2c8973fea3fe0694e/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33302f5265642d626c61636b5f747265655f64656c6574655f636173655f352e706e67)

　　**情形6:** S是黑色，S的右儿子是红色，而N是它父亲的左儿子。在这种情形下我们在N的父亲上做左旋转，这样S成为N的父亲（P）和S的右儿子的父亲。我们接着交换N的父亲和S的颜色，并使S的右儿子为黑色。子树在它的根上的仍是同样的颜色，所以性质3没有被违反。但是，N现在增加了一个黑色祖先: 要么N的父亲变成黑色，要么它是黑色而S被增加为一个黑色祖父。所以，通过N的路径都增加了一个黑色节点。

　　此时，如果一个路径不通过N，则有两种可能性:

- 它通过N的新兄弟。那么它以前和现在都必定通过S和N的父亲，而它们只是交换了颜色。所以路径保持了同样数目的黑色节点。
- 它通过N的新叔父，S的右儿子。那么它以前通过S、S的父亲和S的右儿子，但是现在只通过S，它被假定为它以前的父亲的颜色，和S的右儿子，它被从红色改变为黑色。合成效果是这个路径通过了同样数目的黑色节点。

　　在任何情况下，在这些路径上的黑色节点数目都没有改变。所以我们恢复了性质4。在示意图中的白色节点可以是红色或黑色，但是在变换前后都必须指定相同的颜色。

[![情形6 示意图](https://camo.githubusercontent.com/c9e3fda5572806f8c15531bcb0dd91fee28010f68bfdf9a1a7c7869964d81cd3/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33312f5265642d626c61636b5f747265655f64656c6574655f636173655f362e706e67)](https://camo.githubusercontent.com/c9e3fda5572806f8c15531bcb0dd91fee28010f68bfdf9a1a7c7869964d81cd3/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f332f33312f5265642d626c61636b5f747265655f64656c6574655f636173655f362e706e67)

## 哈夫曼树

> “不假，最近有没有 Java 学习资源？”
>
> “有的，我压缩后发到微信群里哈”（加入 JavaKeeper 交流群各种学习资料哈）

我们都有过对文件进行压缩、解压的操作，压缩而不出错是怎么做到的呢？压缩文本的时候其实是对文本进行了一次重新编码，减少了不必要的空间，而哈夫曼编码算是一种最基本的压缩编码方式。

发明这种压缩编码方式的数学家叫哈夫曼，所以就把他在编码中用到的特殊的二叉树称之为哈弗曼树。

## leetcode

## 参考与感谢

https://www.cnblogs.com/maybe2030/p/4732377.html

[www.bootwiki.com](http://www.bootwiki.com/)

https://www.jianshu.com/p/45661b029292

https://charlesliuyx.github.io/2018/10/22/[直观算法]树的基本操作/

https://www.jianshu.com/p/45661b029292

https://www.cnblogs.com/gaochundong/p/binary_search_tree.html

https://blog.csdn.net/yin767833376/article/details/81511377