# AVL 树

如果按照排序后的序列插入到二叉查找树中，那么一连串的插入操作将花费二次时间。因为，此时二叉查找树退化为了一个单链表。单链表的插入花费的代价是非常大。

为了不让二叉查找树退化，一个好的解决方案就是让二叉查找树保证某个 **平衡条件** ：任何节点的深度不能过深

## 定义

**AVL 树是带有 _平衡条件_ 的二叉查找树**。这个平衡条件要容易保持，而且必须保证树的深度是 $O(\log N)$

> [!tip] 简单但是严格的平衡条件
> 
> 最简单的平衡条件：所有节点的左右子树具有相同的高度
> 
>+ 如果空子树的高度定义为 $-1$，那么只有具有 $2^k-1$ 个机电的理想平衡树满足这个条件
>  
> 这个平衡条件太严格了，难以使用

> [!tip] 稍微放宽的平衡条件
> 
> 更宽松的平衡条件：每个节点的左子树和右子树的高度差最多为 $1$
> 
> + 空树的高度定义为 $-1$

![[Drawing 2024-07-28 20.19.42.excalidraw|900]]

一棵有 $N$ 个节点的 AVL 树的高度最多为 $1.44 \log(N+1) - 1.328$，实际上的高度只比 $\log N$ 稍微多一些

> [!tip] AVL树的所有操作可以以时间 $O(\log N)$ 执行
> 
> 因此，除去可能的插入外（假设使用懒惰删除），所有的树操作都可以以时间 $O(\log N)$ 执行

插入操作时，需要更新通向根节点路径上的那些节点的平衡信息。插入操作的难点就是插入可能会破坏 AVL 树的平衡条件。例如，在上图描述的 AVL 树中插入 $6$，就会破坏节点 $8$ 的平衡条件

![[Drawing 2024-07-28 20.26.24.excalidraw|900]]

## 旋转

一次插入操作需要包含修正节点平衡条件的操作。事实上，总是可以通过 **_旋转_** 来修改节点的平衡条件

让我们把必须重新平衡的节点叫做 $\alpha$。由于任意任意节点最多两个儿子，因此高度不平衡时，$\alpha$ 点的两棵子树高度差 $2$。显然，这种不平衡会出现下面四种情况

> [!tip] 插入导致的 $4$ 种不平衡
> 情形一：对 $\alpha$ 的左子节点的左子树进行插入。即 **左-左** 情形
> 
> 情形二：对 $\alpha$ 的左子节点的右子树进行插入。即 **左-右** 情形
> 
> 情形三：对 $\alpha$ 的右子节点的左子树进行插入。即 **右-左** 情形
> 
> 情形四：对 $\alpha$ 的右子节点的右子树进行插入。即 **右-右** 情形
> 
> 显然，情形一和情形四是关于 $\alpha$ 对称的；情形二和情形三是关于 $\alpha$ 对称的。理论上，只有两种情形
> + 情形一和情形二称为插入发生在 _外边_ 的情形
> + 情形二和情形三称为插入发生在 _内部_ 的情形

对于插入发生在外边的情形，只需要对树进行一次 **_单旋转_** 即可完成调整。然而，对于插入在内部的情形，需要通过 **_双旋转_** 才能完成。

### 单旋转

下图显示了单旋转如何调整情形一。[[Tldraw 2024-07-28 22.33.54]]，旋转前的图在左边，旋转后的图在右边。

![[Pasted image 20240728231121.png]]

下面我们来看看你具体是如何操作的。节点 $k_2$ 不满足 AVL 平衡条件，因为它的左子树比右子树深 $2$ 层

该图描述的是：在插入之前 $k_2$ 保持平衡，但是插入之后，子树 $X$ 多出一层，导致 $k_2$ 的左子树要比右子树 $Z$ 深 $2$ 层

> [!tip]
> 
> $X$ 和 $Y$ 不可能在同一层。因为那样在插入前就失去平衡
> 
> $Y$ 和 $Z$ 也不能在同一层，因为那样 $k_1$ 就会是在通向 root 的路径上破坏 AVL 平衡条件的第一个节点

为了使树恢复平衡，将 $X$ 上移一层，并把 $Z$ 下移一层。为了满足 AVL 树的特性要求，重新安排节点，形成一棵等价树。上图右侧展示了调整后的树结构

> [!tip] 左-左情形：右旋转
> $k_1$ 变成新的 root。由于 AVL 树是一棵二叉查找树，所以 $k_2 \gt k_1$，于是新树中，$k_2$ 成了 $k_1$ 的右子树。
> 
> $X$ 和 $Z$ 仍然分别是 $k_1$ 的左子树和 $k_2$ 的右子树
> 
> $Y$ 包含的元素大于 $k_1$ 小于 $k_2$ 之间，因此将其放在 $k_2$ 的左子树

下图展示了单旋转是如何调整情形四 [[Tldraw 2024-07-28 23.23.15]]。这只是情形一的镜像对称

![[Pasted image 20240728233720.png]]

> [!tip] 右-右情形：左旋转
> $k_2$ 变为新的 root。由于 AVL 树是一棵二叉查找树，所以 $k_2 \gt k_1$，于是新树中，$k_1$ 变成了 $k_2$ 的左子树
> 
> $X$ 和 $Z$ 分别是 $k_1$ 的左子树和 $k_2$ 的右子树
> 
> $Y$ 包含的元素大于 $k_1$ 小于 $k_2$ 之间，因此将其放在 $k_1$ 的右子树

### 双旋转

对于情形二和情形三，单旋转是无效的。原因在与子树 $Y$ 太深了，从先前分析中知道：$Y$ 的深度并没有发生任何改变。为了解决这个问题，需要执行双旋转

下图展示了双旋修复情形二 [[Tldraw 2024-07-29 00.30.32]]。

![[Pasted image 20240729011221.png]]

由于子树 $Y$ 已经有一项插入，这就保证了 $Y$ 非空。因此，可以假设它有一个根和两棵子树

 如上图，子树 $Y$ 被拆分成了 $k_2$ 和其子树 $B,C$。恰好树 $B$ 或者树 $C$ 中有一颗比 $D$ 深  $2$ 层，但是不能确定是哪一棵；事实上，这是无关紧要的，这里我们将 $B$ 和 $C$ 都画成比 $D$ 深 $1\frac{1}{2}$

> [!tip]
> 为了重新平衡，$k_3$ 不能再作为根了，单旋右无法调整。因此，只能让 $k_2$ 做根
> 
> $k_1$ 和 $k_3$ 作为 $k_2$ 的左子树和右子树

> [!tip] 左-右情形：先左旋后右旋

下图展示了双旋修复情形三 [[Tldraw 2024-07-29 01.20.30]]。这只是情形二的镜像对称

![[Pasted image 20240729013445.png]]

> [!tip] 右-左情形：先右旋后左旋

### 演示

假设初始的AVL 树开始插入关键字 $3, 2, 1$，然后插入 $4,5,6,7$

插入关键字 $1$ 时，第一个不平衡问题出现了，AVL 特性在根处被破坏

![[Drawing 2024-07-29 10.29.09.excalidraw|900]]

插入 $4$ 没有任何问题，但是插入 $5$ 又会导致 AVL 树在节点 $3$ 处失去平衡

![[Drawing 2024-07-29 10.47.18.excalidraw|900]]

接着插入 $6$ 会导致 AVL 树在节点 $2$ 处失去平衡

![[Drawing 2024-07-29 11.01.48.excalidraw|900]]

接着插入 $7$ 会导致 AVL 树在节点 $5$ 处失去平衡

![[Drawing 2024-07-29 11.14.04.excalidraw|900]]

---

继续插入关键字 $16,15,14,13,12,11,10$，接着插入 $8$，然后在插入 $9$

插入 $16$ 没有任何问题，当插入 $15$ 时会导致 AVL 树在节点 $7$ 处失去平衡

![[Drawing 2024-07-29 15.26.00.excalidraw|900]]

继续插入 $14$，导致 AVL 树在节点 $6$ 处失去平衡

![[Drawing 2024-07-29 15.41.05.excalidraw|900]]

继续插入 $13$，导致 AVL 树在节点 $4$ 处失去平衡

![[Drawing 2024-07-29 16.00.00.excalidraw|900]]

插入 $12$ ，导致 AVL 树在节点 $14$ 处失去平衡

![[Drawing 2024-07-29 16.20.04.excalidraw|900]]

插入 $11$ ，导致 AVL 树在节点 $15$ 失去平衡

![[Drawing 2024-07-29 16.30.23.excalidraw|900]]

继续插入 $10$，导致 AVL 树在节点 $12$ 处失去平衡

![[Drawing 2024-07-29 16.38.52.excalidraw|900]]


插入 $8$ 不会破坏节点的平衡情况。最后插入 $9$ 导致 AVL 树在节点 $10$ 失去平衡

![[Drawing 2024-07-29 16.45.22.excalidraw|1000]]

## 实现

为了将关键字 $X$ 插入到一颗 AVL 树 $T$ 中，我们使用递归方法；即，递归的将 $X$ 插入到 AVL 树的相应子树($T_{L, R}$)中，如果该子树的高度不变，那么插入完成。否则，如果在 AVL 树中出现高度不平衡，那么根据 $X$ 以及 $T$ 和 $T_{L, R}$ 中的关键字做适当的单旋转或双旋转，更新这些高度(并解决好与树的其余部分的连接)，从而完成插入

关于高度信息的存储如果处理不好将带来效率问题

> [!tip] 存储高度信息的方案
> 
> 由于真正需要的是左右子树的高度差，应该保证它很小。如果保存左右子树的高度差，可以使用能两个二进制位(代表 `+1, 0, -1`)表示这个差
> 
> 然而，这样会让程序损失简明性。不如直接存储每个节点的高度来的简单。所以，采用存储节点的高度

### 类型定义

在每个节点中使用 `height` 保存它高度

```go
package avl

type node struct {
	value  int   // 值
	left   *node // 左子树
	right  *node // 右子树
	height int   // 当前节点的高度
}

type avlTree struct {
	root *node
	len  int
}

// Len() 返回树中存在的元素个数
func (at *avlTree) Len() int {
	return at.len
}

func New() *avlTree {
	return &avlTree{
		root: nil,
		len:  0,
	}
}
```

### Insert

`Insert` 操作比较容易。只是一些列函数调用就能完成插入

```go
func insert(root *node, value int) *node {

	if root == nil {
		root = &node{
			value: value,
		}
	} else if value < root.value {
		// 当期节点大于 value，在左子节点上插入
		root.left = insert(root.left, value)
		// 插入导致不平衡
		if root.left.height-root.right.height == 2 {
			if value < root.left.value {
				// 左子节点的左子树
				root = signalRotateWithLeft(root)
			} else {
				// 左子节点的右子树
				root = doubleRotateWithLeft(root)
			}
		}
	} else if value > root.value {
		// 当前节点小于 value，在右子节点上插入
		root.right = insert(root.right, value)
		if root.right.height-root.left.height == 2 {
			if value > root.right.value {
				// 右子节点的右子树插入
				root = signalRotateWithRight(root)
			} else {
				// 右子节点的左子树插入
				root = doubleRotateWithRight(root)
			}
		}
	}

	//更新高度
	root.height = max(root.left.height, root.right.height) + 1
	return root
}

func max(x, y int) int {
	if x > y {
		return x
	}
	return y
}

func (at *avlTree) Insert(value int) *avlTree {
	at.root = insert(at.root, value)
	at.len++
	return at
}
```

#### 单旋

```go
// 右旋
func signalRotateWithLeft(k2 *node) *node {
	/*
		K2					K1
		/\					/\
	   /  \				   /  \
	  K1   Z              X    K2
	  /\                  |    /\
	 /  \				  |	  /  \
	X    Y					 Y	  Z
	|
	|
	*/

	k1 := k2.left

	k2.left = k1.right

	k1.right = k2
	k2.height = max(k2.left.height, k2.right.height) + 1
	k1.height = max(k1.left.height, k1.right.height) + 1
	return k1
}

// 左旋
func signalRotateWithRight(k1 *node) *node {
	/*
		K1						K2
		/\						/\
	   /  \					   /  \
	  X   K2                  K1   Z
		  /\                  /\   |
		 /  \                /  \  |
		Y    Z              X   Y
			 |
			 |
	*/

	k2 := k1.right

	k1.right = k2.left

	k2.left = k1

	k2.height = max(k2.left.height, k2.right.height) + 1
	k1.height = max(k1.left.height, k1.right.height) + 1
	return k2
}
```

#### 双旋

```go
// 左旋-右旋
func doubleRotateWithLeft(k3 *node) *node {
	/*
		k3					k3      				 K2
		/\					/\					    /  \
	   /  \				   /  \                    /    \
	  k1   D  先左旋      K2   D 在右旋           K1     K3
	  /\   		  		  /\					 / \     / \
	 /  \			     /  \					/   \   /   \
	A	K2	            K1   C                 A     B C    D
		/\				/\
	   /  \		   	   /  \
	  B    C          A    B
	现在 K1 左旋，然后在 k3 右旋
	*/
	k3.left = signalRotateWithRight(k3.left)
	k3 = signalRotateWithLeft(k3)
	return k3
}

// 右旋-左旋
func doubleRotateWithRight(k1 *node) *node {
	/*
		K1     		   		K1						 K2
	   /  \				   /  \					    /  \
	  /    \ 			  /    \			       /    \
	 A     K3  先右旋  	 A      K2  后左旋        K1      K3
		  /  \    			   /  \              / \     /\
		 /	  \				  /	   \            /   \   /  \
		K2	   D             B		K3         A     B  C   D
	   /  \						   /  \
	  /    \                      /    \
	 B		C                    C     D
	*/
	k1.right = signalRotateWithLeft(k1.right)
	k1 = signalRotateWithRight(k1)
	return k1
}
```

### Delete

如果删除的不多，建议使用懒惰删除。

