# 理解和实现树

到目前为止，我们对数据结构的探索只涉及了线性部分。无论我们使用数组、链表、栈还是队列，所有这些都是线性数据结构。我们已经看到了线性数据结构操作的复杂性，大多数情况下，插入和删除可以以`O(1)`的复杂度执行。然而，搜索有点复杂，并且需要`O(n)`的复杂度。唯一的例外是 PHP 数组，实际上它的工作原理是哈希表，如果索引或键以这种方式管理，可以在`O(1)`中进行搜索。为了解决这个问题，我们可以使用分层数据结构而不是线性数据结构。分层数据可以解决许多线性数据结构无法轻松解决的问题。每当我们谈论家族谱系、组织结构和网络连接图时，实际上我们在谈论分层数据。树是一种表示分层数据的特殊**抽象数据类型**（**ADT**）。与链表不同，链表也是一种 ADT，树是分层的，而链表是线性的。在本章中，我们将探索树的世界。树结构的一个完美例子可以是家族谱系，就像下面的图片：

![](img/Image00041.jpg)

# 树的定义和属性

树是由边连接的节点或顶点的分层集合。树不能有循环，只有边存在于节点和其后代节点或子节点之间。同一父节点的两个子节点之间不能有任何边。每个节点除了顶节点（也称为根节点）外，还可以有一个父节点。每棵树只能有一个根节点。在下图中，**A**是根节点，**B**，**C**和**D**是**A**的子节点。我们还可以说 A 是**B**，**C**和**D**的父节点。**B**，**C**和**D**被称为兄弟姐妹，因为它们是来自同一父节点**A**的子节点：

![](img/Image00042.gif)

没有任何子节点的节点称为叶子。在前面的图表中，**K**，**L**，**F**，**G**，**M**，**I**和**J**都是叶子节点。叶子节点也称为外部节点或终端节点。除了根节点之外，至少有一个子节点的节点称为内部节点。在这里，**B**，**C**，**D**，**E**和**H**是内部节点。在描述树数据结构时，我们使用一些其他常见术语：

+   **后代**：这是一个可以通过重复进行到达父节点的节点。例如，在前面的图表中，**M** 是**C**的后代。

+   **祖先**：这是一个可以通过重复方式从子节点到父节点到达的节点。例如，**B**是**L**的祖先。

+   **度**：特定父节点的子节点总数称为其度。在我们的例子中，**A** 的度为 3，**B** 的度为 1，**C** 的度为 3，**D** 的度为 2。

+   **路径**：从源节点到目标节点的节点和边的序列称为两个节点之间的路径。路径的长度是路径中的节点数。在我们的例子中，**A**到**M**的路径是**A-C-H-M**，路径的长度为 4：

![](img/Image00043.jpg)

+   **节点的高度**：节点的高度由节点与后代节点的最深层之间的边的数量定义。例如，节点**B**的高度为 2。

+   **层级**：层级表示节点的代。如果父节点在第*n*层，其子节点将在*n+1*层。因此，层级由节点与根之间的边的数量加 1 定义。在这里：

+   +   根**A**在**Level 0**

+   **B**，**C**和**D**在**Level 1**

+   **E**，**F**，**G**，**H**，**I**和**J**在**Level 2**

+   **K**，**L**和**M**在**Level 3**

+   **树的高度**：树的高度由其根节点的高度定义。在这里，树的高度为 3。

+   **子树**：在树结构中，每个子节点都递归地形成一个子树。换句话说，树由许多子树组成。例如，**B**与**E**，**K**和**L**形成一个子树，而**E**与**K**和**L**形成一个子树。在前面的例子中，我们已经在左侧用不同的颜色标识了每个子树。我们也可以对**C**和**D**及其子树做同样的事情。

+   **深度**：节点的深度由节点与根节点之间的边的数量确定。例如，在我们的树图中，**H**的深度为 2，**L**的深度为 3。

+   **森林**：森林是零个或多个不相交树的集合。

+   **遍历**：这表示按特定顺序访问节点的过程。我们将在接下来的部分经常使用这个术语。

+   **键**：键是用于搜索目的的节点中的值。

# 使用 PHP 实现树

到目前为止，您已经了解了树数据结构的不同属性。如果我们将树数据结构与现实生活中的例子进行比较，我们可以考虑我们的组织结构或家族谱来表示数据结构。对于组织结构，有一个根节点，可以是公司的 CEO，然后是 CXO 级别的员工，然后是其他级别的员工。在这里，我们不限制特定节点的程度。这意味着一个节点可以有多个子节点。因此，让我们考虑一个节点结构，我们可以定义节点属性、其父节点和其子节点。它可能看起来像这样：

```php
class TreeNode { 

    public $data = NULL; 

    public $children = []; 

    public function __construct(string $data = NULL) { 

      $this->data = $data; 

    } 

    public function addChildren(TreeNode $node) { 

      $this->children[] = $node; 

    } 

} 

```

如果我们看一下前面的代码，我们可以看到我们为数据和子节点声明了两个公共属性。我们还有一个方法来向特定节点添加子节点。在这里，我们只是将新的子节点追加到数组的末尾。这将使我们有选择地为特定节点添加多个节点作为子节点。由于树是一个递归结构，它将帮助我们递归地构建树，也可以递归地遍历树。

现在，我们有了节点，让我们构建一个树结构，定义树的根节点以及遍历整个树的方法。因此，基本的树结构将如下所示：

```php
class Tree { 

    public $root = NULL; 

    public function __construct(TreeNode $node) { 

      $this->root = $node; 

    } 

    public function traverse(TreeNode $node, int $level = 0) { 

      if ($node) { 

        echo str_repeat("-", $level); 

        echo $node->data . "\n"; 

        foreach ($node->children as $childNode) { 

          $this->traverse($childNode, $level + 1); 

        } 

      } 

    } 

} 

```

前面的代码显示了一个简单的树类，我们可以在其中存储根节点引用，并从任何节点遍历树。在遍历部分，我们正在访问每个子节点，然后立即递归调用遍历方法以获取当前节点的子节点。我们正在传递一个级别，以便在节点名称的开头打印一个破折号（-），这样我们就可以轻松地理解子级数据。

现在让我们创建根节点并将其分配给树作为根。代码将如下所示：

```php
    $ceo = new TreeNode("CEO"); 

    $tree = new Tree($ceo); 

```

在这里，我们创建了第一个节点作为 CEO，然后创建了树，并将 CEO 节点分配为树的根节点。现在是时候从根节点开始扩展我们的树了。由于我们选择了 CEO 的例子，我们现在将在 CEO 下添加 CXO 和其他员工。以下是此代码：

```php
$cto     = new TreeNode("CTO"); 

$cfo     = new TreeNode("CFO"); 

$cmo     = new TreeNode("CMO"); 

$coo     = new TreeNode("COO"); 

$ceo->addChildren($cto); 

$ceo->addChildren($cfo); 

$ceo->addChildren($cmo); 

$ceo->addChildren($coo); 

$seniorArchitect = new TreeNode("Senior Architect"); 

$softwareEngineer = new TreeNode("Software Engineer"); 

$userInterfaceDesigner      = new TreeNode("User Interface Designer"); 

$qualityAssuranceEngineer = new TreeNode("Quality Assurance Engineer"); 

$cto->addChildren($seniorArchitect); 

$seniorArchitect->addChildren($softwareEngineer); 

$cto->addChildren($qualityAssuranceEngineer); 

$cto->addChildren($userInterfaceDesigner); 

$tree->traverse($tree->root); 

```

在这里，我们在开始时创建了四个新节点（CTO、CFO、CMO 和 COO），并将它们分配为 CEO 节点的子节点。然后我们创建了高级架构师，这是软件工程师节点，接着是用户界面设计师和质量保证工程师。我们已经将高级软件工程师节点分配为高级架构师节点的子节点，并将高级架构师分配为 CTO 的子节点，以及用户界面工程师和质量保证工程师。最后一行是从根节点显示树。这将在我们的命令行中输出以下行：

```php
CEO

-CTO

--Senior Architect

---Software Engineer

--Quality Assurance Engineer

--User Interface Designer

-CFO

-CMO

-COO

```

考虑到前面的输出，我们在级别 0 处有`CEO`。`CTO`，`CFO`，`CMO`和`COO`在级别 1 处。`Senior Architect`，`User Interface Designer`和`Quality Assurance Engineer`在级别 2 处，`Software Engineer`在级别 3 处。

我们已经使用 PHP 构建了一个基本的树数据结构。现在，我们将探索我们拥有的不同类型的树。

# 不同类型的树结构

编程世界中存在许多类型的树数据结构。我们将在这里探讨一些最常用的树结构。

# 二叉树

二进制是树结构的最基本形式，其中每个节点最多有两个子节点。子节点称为左节点和右节点。二叉树将如下图所示：

![](img/Image00044.jpg)

# 二叉搜索树

二叉搜索树（BST）是一种特殊类型的二叉树，其中节点以排序的方式存储。它以这样一种方式排序，即在任何给定点，节点值必须大于或等于左子节点值，并且小于右子节点值。每个节点都必须满足此属性，才能将其视为二叉搜索树。由于节点按特定顺序排序，二叉搜索算法可以应用于以对数时间搜索 BST 中的项目。这总是优于线性搜索，它需要**O(n)**时间，我们将在下一章中探讨它。以下是一个二叉搜索树的示例：

![](img/Image00045.jpg)

# 自平衡二叉搜索树

自平衡二叉搜索树或高度平衡二叉搜索树是一种特殊类型的二叉搜索树，它试图通过自动调整始终保持树的高度或层级数尽可能小。例如，下图显示了左侧的二叉搜索树和右侧的自平衡二叉搜索树：

![](img/Image00046.jpg)

高度平衡的二叉树总是比普通 BST 更好，因为它可以使搜索操作比普通 BST 更快。有不同的自平衡或高度平衡二叉搜索树的实现。其中一些流行的如下：

+   AA 树

+   AVL 树

+   红黑树

+   替罪羊树

+   伸展树

+   2-3 树

+   Treap

我们将在以下章节讨论一些高度平衡树。

# AVL 树

AVL 树是一种自平衡的二叉搜索树，其中一个节点的两个子树的高度最多相差 1。如果高度增加，在任何情况下都会重新平衡以使高度差为 1。这使 AVL 树在不同操作的复杂度上具有对数优势。以下是 AVL 树的示例：

![](img/Image00047.jpg)

# 红黑树

红黑树是一种具有额外属性的自平衡二叉搜索树，即颜色。二叉树中的每个节点存储一位额外的信息，即颜色，可以具有红色或黑色的值。与 AVL 树一样，红黑树也用于实时应用，因为平均和最坏情况的复杂度也是对数的。示例红黑树如下：

![](img/Image00048.jpg)

# B 树

B 树是一种特殊类型的二叉树，它是自平衡的。这与自平衡的二叉搜索树不同。关键区别在于，在 B 树中，我们可以有任意数量的节点作为子节点，而不仅仅是两个。B 树用于大量数据，并主要用于文件系统和数据库。B 树中不同操作的复杂度是对数的。

# N 叉树

N 叉树是一种特殊类型的树，其中一个节点最多可以有 N 个子节点。这也被称为 k 路树或 M 路树。二叉树是 N 叉树，其中 N 的值为 2。

# 理解二叉树

我们经常会对二叉树和二叉搜索树感到困惑。正如我们在定义中所看到的，BST 是一种排序的二叉树。如果它是排序的，那么与普通二叉树相比，我们可以有性能改进。每个二叉树节点最多可以有两个子节点，分别称为左子节点和右子节点。然而，根据二叉树的类型，可以有零个、一个或两个子节点。

我们还可以将二叉树分类为不同的类别：

+   **满二叉树：** 满二叉树是一棵树，每个节点上要么没有子节点，要么有两个子节点。满二叉树也被称为完全二叉树或平衡二叉树。

+   **完美二叉树：** 完美二叉树是一棵二叉树，其中所有内部节点恰好有两个子节点，所有叶子节点的级别或深度相同。

+   **完全二叉树：** 完全二叉树是一棵二叉树，除了最后一层外，所有层都完全填充，所有节点尽可能地靠左。以下图表显示了满二叉树、完全二叉树和完美二叉树：

![](img/Image00049.jpg)

# 实现二叉树

我们现在将创建一个二叉树（不是二叉搜索树）。二叉树中必须具有的关键因素是，我们必须为左孩子节点和右孩子节点保留两个占位符，以及我们想要存储在节点中的数据。二叉节点的简单实现将如下所示：

```php
class BinaryNode { 

    public $data; 

    public $left; 

    public $right; 

    public function __construct(string $data = NULL) { 

      $this->data = $data; 

      $this->left = NULL; 

      $this->right = NULL; 

    } 

    public function addChildren(BinaryNode $left, BinaryNode $right) { 

      $this->left = $left;

      $this->right = $right;

    }

}

```

前面的代码显示，我们有一个带有树属性的类来存储数据，左边和右边。当我们构造一个新节点时，我们将节点值添加到数据属性中，左边和右边保持`NULL`，因为我们不确定是否需要它们。我们还有一个`addChildren`方法来向特定节点添加左孩子和右孩子。

现在，我们将创建一个二叉树类，我们可以在其中定义根节点以及类似于本章早期的基本树实现的遍历方法。两种实现之间的区别在于遍历过程。在我们之前的示例中，我们使用`foreach`来遍历每个子节点，因为我们不知道有多少个节点。由于二叉树中的每个节点最多可以有两个节点，并且它们被命名为左和右，我们只能遍历左节点，然后遍历每个特定节点访问的右节点。更改后的代码将如下所示：

```php
class BinaryTree { 

    public $root = NULL; 

    public function __construct(BinaryNode $node) { 

    $this->root = $node; 

    } 

    public function traverse(BinaryNode $node, int $level    

      = 0) { 

      if ($node) { 

          echo str_repeat("-", $level); 

          echo $node->data . "\n"; 

          if ($node->left) 

            $this->traverse($node->left, $level + 1); 

          if ($node->right) 

            $this->traverse($node->right, $level + 1); 

         } 

    } 

} 

```

这看起来与本章早期我们所拥有的基本树类非常相似。现在，让我们用一些节点填充二叉树。通常，在任何足球或板球比赛中，我们都有淘汰赛轮次，两支球队互相比赛，赢家继续前进，一直到决赛。我们可以在我们的示例中使用类似的结构作为二叉树。因此，让我们创建一些二叉节点并将它们结构化：

```php
$final = new BinaryNode("Final"); 

$tree = new BinaryTree($final); 

$semiFinal1 = new BinaryNode("Semi Final 1"); 

$semiFinal2 = new BinaryNode("Semi Final 2"); 

$quarterFinal1 = new BinaryNode("Quarter Final 1"); 

$quarterFinal2 = new BinaryNode("Quarter Final 2"); 

$quarterFinal3 = new BinaryNode("Quarter Final 3"); 

$quarterFinal4 = new BinaryNode("Quarter Final 4"); 

$semiFinal1->addChildren($quarterFinal1, $quarterFinal2); 

$semiFinal2->addChildren($quarterFinal3, $quarterFinal4); 

$final->addChildren($semiFinal1, $semiFinal2); 

$tree->traverse($tree->root); 

```

首先，我们创建了一个名为 final 的节点，并将其作为根节点。然后，我们创建了两个半决赛节点和四个四分之一决赛节点。两个半决赛节点分别有两个四分之一决赛节点作为左右子节点。最终节点有两个半决赛节点作为左右子节点。`addChildren`方法正在为节点执行子节点分配工作。在最后一行，我们遍历了树并按层次显示了数据。如果我们在命令行中运行此代码，我们将看到以下输出：

```php
Final

-Semi Final 1

--Quarter Final 1

--Quarter Final 2

-Semi Final 2

--Quarter Final 3

--Quarter Final 4

```

# 使用 PHP 数组创建二叉树

我们可以使用 PHP 数组实现二叉树。由于二叉树最多可以有零到两个子节点，我们可以将最大子节点数设为 2，并构建一个公式来找到给定节点的子节点。让我们从上到下、从左到右为二叉树中的节点编号。因此，根节点将具有编号**0**，左孩子**1**，右孩子**2**，依此类推，直到为每个节点编号，就像以下图表所示：

![](img/Image00050.gif)

我们很容易看到，对于节点**0**，左孩子是**1**，右孩子是**2**。对于节点**1**，左孩子是**3**，右孩子是**4**，依此类推。我们可以很容易地将这个放入一个公式中：

如果*i*是我们的节点编号，那么：

*左节点= 2 X i + 1*

*右节点= 2 X (i + 1)*

现在，让我们使用 PHP 数组创建比赛日程的示例。如果按照我们的讨论进行排名，那么它将如下所示：

```php
    $nodes = []; 

    $nodes[] = "Final"; 

    $nodes[] = "Semi Final 1"; 

    $nodes[] = "Semi Final 2"; 

    $nodes[] = "Quarter Final 1"; 

    $nodes[] = "Quarter Final 2"; 

    $nodes[] = "Quarter Final 3"; 

    $nodes[] = "Quarter Final 4"; 

```

基本上，我们将创建一个带有自动索引的数组，从 0 开始。这个数组将被用作二叉树的表示。现在，我们将修改我们的`BinaryTree`类，使用这个数组而不是我们的节点类，以及左右子节点以及遍历方法。现在，我们将基于节点编号而不是实际节点引用进行遍历：

```php
class BinaryTree { 

    public $nodes = []; 

    public function __construct(Array $nodes) { 

      $this->nodes = $nodes; 

    } 

    public function traverse(int $num = 0, int $level = 0) { 

      if (isset($this->nodes[$num])) { 

          echo str_repeat("-", $level); 

          echo $this->nodes[$num] . "\n"; 

          $this->traverse(2 * $num + 1, $level+1); 

          $this->traverse(2 * ($num + 1), $level+1); 

      } 

    } 

} 

```

从前面的实现中可以看出，遍历部分使用节点位置而不是引用。这个节点位置就是数组索引。因此，我们可以直接访问数组索引并检查它是否为空。如果不为空，我们可以继续使用递归的方式深入。如果我们想使用数组创建二叉树并打印数组值，我们必须编写以下代码：

```php
$tree = new BinaryTree($nodes); 

$tree->traverse(0); 

```

如果我们在命令行中运行此代码，将会看到以下输出：

```php
Final

-Semi Final 1

--Quarter Final 1

--Quarter Final 2

-Semi Final 2

--Quarter Final 3

--Quarter Final 4

```

我们可以使用一个简单的`while`循环来遍历数组并访问每个节点，而不是递归进行。在我们所有的递归示例中，我们会发现如果以迭代的方式使用它们，有些会更有效率。我们也可以直接使用它们，而不是为二叉树创建一个类。

# 理解二叉搜索树

BST 是一种二叉树，它是按照树始终排序的方式构建的。这意味着左孩子节点的值小于或等于父节点的值，右孩子节点的值大于父节点的值。因此，每当我们需要搜索一个值时，要么搜索左边，要么搜索右边。由于它是排序的，我们只需要搜索树的一部分，而不是两部分，这种递归持续进行。由于它的分割性质，搜索变得非常快，我们可以实现对搜索的对数复杂度。例如，如果我们有*n*个节点，我们将搜索前半部分或后半部分的节点。一旦我们在前半部分或后半部分，我们可以再次将其分成两半，这意味着我们的一半现在变成了四分之一，如此循环直到达到最终节点。由于我们不是移动到每个节点进行搜索，因此操作不会花费`O(n)`的复杂度。在下一章中，我们将对二分搜索的复杂性进行分析，并看到为什么二叉搜索树的搜索复杂度是`O(log n)`。与二叉树不同，我们不能在不重建 BST 属性的情况下向树中添加任何节点或删除任何节点。

如果节点**X**有两个孩子，则节点**X**的后继是属于树的最小值，大于**X**的值。换句话说，后继是右子树的最小值。另一方面，前驱是左子树的最大值。现在，我们将更多关注 BST 的不同操作以及执行这些操作时需要考虑的步骤。

以下是 BST 的操作。

# 插入一个新节点

当我们在二叉搜索树中插入一个新节点时，我们必须考虑以下步骤：

1.  创建一个新节点作为叶子节点（没有左孩子或右孩子）。

1.  从根节点开始，并将其设置为当前节点。

1.  如果节点为空，则将新节点作为根。

1.  检查新值是小于当前节点还是大于当前节点。

1.  如果小于，则转到左侧并将左侧设置为当前节点。

1.  如果大于，则转到右侧并将右侧设置为当前节点。

1.  继续*步骤 3*，直到所有节点都被访问并设置了新节点。

# 搜索一个节点

当我们在二叉搜索树中搜索一个新节点时，我们必须考虑以下步骤：

1.  从根节点开始，并将其设置为当前节点。

1.  如果当前节点为空，则返回 false。

1.  如果当前节点的值是搜索值，则返回 true。

1.  检查搜索值是小于当前节点还是大于当前节点。

1.  如果小于，则转到左侧并将左侧设置为当前节点。

1.  如果大于，则转到右侧并将右侧设置为当前节点。

1.  继续*步骤 3*，直到所有节点都被访问。

# 查找最小值

由于二叉搜索树以排序方式存储数据，我们始终可以在左节点中找到较小的数据，在右节点中找到较大的数据。因此，查找最小值将需要我们从根节点开始访问所有左节点，直到找到最左边的节点及其值。以下是查找最小值的步骤：

1.  从根节点开始，并将其设置为当前节点。

1.  如果当前节点为空，则返回 false。

1.  转到左侧并将左侧设置为当前节点。

1.  如果当前节点没有左节点，则转到*步骤 5*；否则，继续*步骤 4*。

1.  继续*步骤 3*，直到所有左节点都被访问。

1.  返回当前节点。

# 查找最大值

以下是查找最大值的步骤：

1.  从根节点开始，并将其设置为当前节点。

1.  如果当前节点为空，则返回 false。

1.  转到右侧并将右侧设置为当前节点。

1.  如果当前节点没有右节点，则转到*步骤 5*；否则，继续*步骤 4*。

1.  继续*步骤 3*，直到所有右节点都被访问。

1.  返回当前节点。

# 删除节点

当我们删除一个节点时，我们必须考虑节点可以是内部节点或叶子节点。如果它是叶子节点，则它没有子节点。但是，如果节点是内部节点，则它可以有一个或两个子节点。在这种情况下，我们需要采取额外的步骤来确保在删除后树的构造是正确的。这就是为什么从 BST 中删除节点始终是一项具有挑战性的工作，与其他操作相比。以下是删除节点时要考虑的事项：

1.  如果节点没有子节点，则使节点为 NULL。

1.  如果节点只有一个子节点，则使子节点取代节点的位置。

1.  如果节点有两个子节点，则找到节点的后继并将其替换为当前节点的位置。删除后继节点。

我们已经讨论了二叉搜索树的大部分可能操作。现在，我们将逐步实现二叉搜索树，从插入、搜索、查找最小和最大值开始，最后是删除操作。让我们开始实现吧。

# 构建二叉搜索树

正如我们所知，一个节点可以有两个子节点，并且本身可以以递归方式表示树。我们将定义我们的节点类更加功能强大，并具有所有必需的功能来查找最大值、最小值、前任和后继。稍后，我们还将为节点添加删除功能。让我们检查 BST 的节点类的以下代码：

```php
class Node { 

    public $data; 

    public $left; 

    public $right; 

    public function __construct(int $data = NULL) { 

       $this->data = $data; 

       $this->left = NULL; 

       $this->right = NULL; 

    } 

    public function min() { 

       $node = $this; 

       while($node->left) { 

         $node = $node->left; 

       } 

         return $node; 

    } 

    public function max() { 

         $node = $this; 

         while($node->right) { 

            $node = $node->right; 

         } 

         return $node; 

    } 

    public function successor() { 

         $node = $this; 

         if($node->right) 

               return $node->right->min(); 

         else 

               return NULL; 

    } 

    public function predecessor() { 

         $node = $this; 

         if($node->left) 

               return $node->left->max(); 

         else 

               return NULL;

    }

}

```

节点类看起来很简单，并且与我们在前一节中定义的步骤相匹配。每个新节点都是叶子节点，因此在创建时没有左节点或右节点。由于我们知道可以在节点的左侧找到较小的值以找到最小值，因此我们正在到达最左边的节点和最右边的节点以获取最大值。对于后继，我们正在从给定节点的右子树中找到节点的最小值，并且对于前任部分，我们正在从左子树中找到节点的最大值。

现在，我们需要一个 BST 结构来在树中添加新节点，以便我们可以遵循插入原则：

```php
class BST { 

    public $root = NULL; 

    public function __construct(int $data) { 

         $this->root = new Node($data); 

    } 

    public function isEmpty(): bool { 

         return $this->root === NULL; 

    } 

    public function insert(int $data) { 

         if($this->isEmpty()) { 

               $node = new Node($data); 

               $this->root = $node; 

               return $node; 

         }  

    $node = $this->root; 

    while($node) { 

      if($data > $node->data) { 

          if($node->right) { 

            $node = $node->right; 

          } else { 

            $node->right = new Node($data); 

            $node = $node->right; 

            break; 

          } 

      } elseif($data < $node->data) { 

          if($node->left) { 

            $node = $node->left; 

          } else { 

            $node->left = new Node($data); 

            $node = $node->left; 

            break; 

          } 

      } else { 

            break; 

      } 

    } 

    return $node; 

    } 

    public function traverse(Node $node) { 

      if ($node) { 

          if ($node->left) 

            $this->traverse($node->left); 

          echo $node->data . "\n"; 

          if ($node->right)

            $this->traverse($node->right);

      }

    }

}

```

如果我们看前面的代码，我们只有一个 BST 类的属性，它将标记根节点。在构建 BST 对象时，我们传递一个单个值，该值将用作树的根。`isEmpty`方法检查树是否为空。`insert`方法允许我们在树中添加新节点。逻辑检查值是否大于或小于根节点，并遵循 BST 的原则将新节点插入正确的位置。如果值已经插入，我们将忽略它并避免添加到树中。

我们还有一个`traverse`方法来遍历节点并以有序格式查看数据（首先左侧，然后是节点，然后是右侧节点的值）。它有一个指定的名称，我们将在下一节中探讨。现在，让我们准备一个样本代码来使用 BST 类，并添加一些数字，然后检查这些数字是否以正确的方式存储。如果 BST 有效，则遍历将显示一个有序的数字列表，无论我们如何插入它们：

```php
$tree = new BST(10); 

$tree->insert(12); 

$tree->insert(6); 

$tree->insert(3); 

$tree->insert(8); 

$tree->insert(15); 

$tree->insert(13); 

$tree->insert(36); 

$tree->traverse($tree->root);

```

如果我们看一下前面的代码，`10`是我们的根节点，然后我们随机添加了新节点。最后，我们调用了遍历方法来显示节点以及它们在二叉搜索树中的存储方式。以下是前面代码的输出：

```php
3

6

8

10

12

13

15

36

```

实际树在视觉上看起来是这样的，与 BST 实现所期望的完全一样：

![](img/Image00051.jpg)

现在，我们将在我们的 BST 类中添加搜索部分。我们想要找出值是否存在于树中。如果值不在我们的 BST 中，它将返回 false，否则返回节点。这是简单的搜索功能：

```php
public function search(int $data) { 

  if ($this->isEmpty()) { 

      return FALSE; 

  } 

  $node = $this->root; 

  while ($node) { 

      if ($data > $node->data) { 

        $node = $node->right; 

      } elseif ($data < $node->data) { 

        $node = $node->left; 

      } else { 

        break; 

      } 

  } 

  return $node; 

}

```

在前面的代码中，我们可以看到我们正在从节点中搜索树中的值，并迭代地跟随树的左侧或右侧。如果没有找到具有该值的节点，则返回节点的叶子节点，即`NULL`。我们可以这样测试代码：

```php
echo $tree->search(14) ? "Found" : "Not Found";

echo "\n";

echo $tree->search(36) ? "Found" : "Not Found";

```

这将产生以下输出。由于`14`不在我们的列表中，它将显示`Not Found`，而对于`36`，它将显示`Found`：

```php
Not Found

Found

```

现在，我们将进入编码中最复杂的部分，即删除节点。我们需要实现节点可以有零个、一个或两个子节点的每种情况。以下图像显示了我们需要满足的删除节点的三个条件，并确保在操作后二叉搜索树仍然是二叉搜索树。当处理具有两个子节点的节点时，我们需要小心。因为我们需要在节点之间来回移动，我们需要知道当前节点的父节点是哪个节点。因此，我们需要添加一个额外的属性来跟踪任何节点的父节点：

![](img/Image00052.jpg)

这是我们要添加到`Node`类的代码更改：

```php
    public $data;

    public $left;

    public $right;

    public $parent;

    public function __construct(int $data = NULL, Node $parent = NULL)   

     {

      $this->data = $data; 

      $this->parent = $parent; 

      $this->left = NULL; 

      $this->right = NULL; 

     }

```

此代码块现在还将新创建的节点与其直接父节点建立父子关系。我们还希望将我们的删除功能与单个节点关联起来，以便我们可以找到一个节点，然后只需使用`delete`方法将其删除。以下是删除功能的代码：

```php
public function delete() { 

    $node = $this; 

    if (!$node->left && !$node->right) { 

        if ($node->parent->left === $node) { 

          $node->parent->left = NULL; 

        } else { 

          $node->parent->right = NULL; 

        } 

    } elseif ($node->left && $node->right) { 

        $successor = $node->successor(); 

        $node->data = $successor->data; 

        $successor->delete(); 

    } elseif ($node->left) { 

        if ($node->parent->left === $node) { 

          $node->parent->left = $node->left; 

          $node->left->parent = $node->parent->left; 

        } else { 

          $node->parent->right = $node->left; 

          $node->left->parent = $node->parent->right; 

        } 

        $node->left = NULL; 

    } elseif ($node->right) { 

        if ($node->parent->left === $node) { 

          $node->parent->left = $node->right; 

          $node->right->parent = $node->parent->left; 

        } else { 

          $node->parent->right = $node->right; 

          $node->right->parent = $node->parent->right; 

        } 

        $node->right = NULL; 

    }

}

```

第一个条件检查节点是否是叶子节点。如果节点是叶子节点，那么我们只需使父节点删除子节点的引用（左侧或右侧）。这样，节点将与树断开连接，满足了我们零个子节点的第一个条件。

接下来的条件实际上检查了我们的第三个条件，即节点有两个子节点的情况。在这种情况下，我们获取节点的后继节点，将后继节点的值分配给节点本身，并删除后继节点。这只是从后继节点复制数据。

接下来的两个条件检查节点是否有单个子节点，就像我们之前的*Case 2*图表所示。由于节点只有一个子节点，它可以是左子节点或右子节点。因此，条件检查单个子节点是否是节点的左子节点。如果是，我们需要根据节点本身与其父节点的位置，将左子节点指向节点的父节点左侧或右侧引用。右子节点也适用相同的规则。在这里，右子节点引用设置为其父节点的左侧或右侧子节点，而不是基于节点位置的引用。

由于我们已经更新了我们的节点类，我们需要对我们的 BST 类进行一些更改，以便插入和删除节点。插入代码将如下所示：

```php
function insert(int $data)

 {

    if ($this->isEmpty()) {

          $node = new Node($data);

          $this->root = $node;

          return $node;

    }

    $node = $this->root;

    while ($node) {

          if ($data > $node->data) {

                if ($node->right) {

                      $node = $node->right;

                }

                else {

                      $node->right = new Node($data, $node);

                      $node = $node->right;

                      break;

                }

          }

          elseif ($data < $node->data) {

                if ($node->left) {

                      $node = $node->left;

                }

                else {

                      $node->left = new Node($data, $node);

                      $node = $node->left;

                      break;

                }

          }

          else {

                break;

    }

 }

    return $node;

 }

```

代码看起来与我们之前使用的代码类似，只有一个小改变。现在，当我们创建一个新节点时，我们会发送当前��点的引用。这个当前节点将被用作新节点的父节点。`new Node($data, $node)`代码实际上就是这样做的。

对于删除一个节点，我们可以先进行搜索，然后使用节点类中的`delete`方法删除搜索到的节点。因此，`remove`函数本身将会非常小，就像这里的代码一样：

```php
public function remove(int $data) {

    $node = $this->search($data);

    if ($node) $node->delete();

 }

```

如代码所示，我们首先搜索数据。如果节点存在，我们将使用`delete`方法将其移除。现在，让我们运行我们之前的例子，使用`remove`调用，看看它是否有效：

```php
   $tree->remove(15);

   $tree->traverse($tree->root);

```

我们只是从我们的树中移除`15`，然后从根节点遍历树。我们现在将看到以下输出：

```php
3

6

8

10

12

13

36

```

我们可以看到 15 不再是我们 BST 的一部分了。这样，我们可以移除任何节点，如果我们使用相同的方法进行遍历，我们将会看到一个排序的列表。如果我们看我们之前的输出，我们可以看到输出是按升序显示的。这其中有一个原因，我们将在下一个主题-不同的树遍历方式中探讨。

您可以在[`btv.melezinek.cz/binary-search-tree.html`](http://btv.melezinek.cz/binary-search-tree.html)找到一个用于可视化二叉搜索树操作的好工具。这对于学习者来说是一个很好的开始，可以通过可视化的方式理解不同的操作。

# 树的遍历

树的遍历是指我们访问给定树中的每个节点的方式。根据我们进行遍历的方式，我们可以遵循三种不同的遍历方式。这些遍历在许多不同的方面都非常重要。表达式求值的波兰表示法转换就是使用树遍历的最流行的例子之一。

# 中序

中序树遍历首先访问左节点，然后是根节点，然后是右节点。对于每个节点，这将递归地继续进行。左节点存储的值比根节点值小，右节点存储的值比根节点大。因此，当我们应用中序遍历时，我们得到一个排序的列表。这就是为什么到目前为止，我们的二叉树遍历显示的是一个排序的数字列表。这种遍历部分实际上就是中序树遍历的例子。中序树遍历遵循以下原则：

1.  通过递归调用中序函数来遍历左子树。

1.  显示根（或当前节点）的数据部分。

1.  递归调用中序函数来遍历右子树。

![](img/Image00053.jpg)

前面的树将显示 A、B、C、D、E、F、G、H 和 I 作为输出，因为它是按照中序遍历进行遍历的。

# 前序

在前序遍历中，首先访问根节点，然后是左节点，然后是右节点。前序遍历的原则如下：

1.  显示根（或当前节点）的数据部分。

1.  通过递归调用前序函数来遍历左子树。

1.  通过递归调用前序函数来遍历右子树。

![](img/Image00054.jpg)

前面的树将以 F、B、A、D、C、E、G、I 和 H 作为输出，因为它是按照前序遍历进行遍历的。

# 后序

在后序遍历中，最后访问根节点。首先访问左节点，然后是右节点。后序遍历的原则如下：

1.  通过递归调用后序函数来遍历左子树。

1.  通过递归调用后序函数来遍历右子树。

1.  显示根（或当前节点）的数据部分。

![](img/Image00055.jpg)

前序遍历将以 A、C、E、D、B、H、I、G 和 F 作为输出，因为它是按照后序遍历进行遍历的。

现在，让我们在我们的 BST 类中实现遍历逻辑：

```php
public function traverse(Node $node, string $type="in-order") { 

switch($type) {        

    case "in-order": 

      $this->inOrder($node); 

    break; 

    case "pre-order": 

      $this->preOrder($node); 

    break; 

    case "post-order": 

      $this->postOrder($node); 

    break;       

}      

} 

public function preOrder(Node $node) { 

  if ($node) { 

      echo $node->data . " "; 

      if ($node->left) $this->traverse($node->left); 

      if ($node->right) $this->traverse($node->right); 

  }      

} 

public function inOrder(Node $node) { 

  if ($node) {           

      if ($node->left) $this->traverse($node->left); 

      echo $node->data . " "; 

      if ($node->right) $this->traverse($node->right); 

  } 

} 

public function postOrder(Node $node) { 

  if ($node) {           

      if ($node->left) $this->traverse($node->left); 

      if ($node->right) $this->traverse($node->right); 

      echo $node->data . " "; 

  } 

} 

```

现在，如果我们对我们之前的二叉搜索树运行三种不同的遍历方法，这里是运行遍历部分的代码：

```php
   $tree->traverse($tree->root, 'pre-order');

   echo "\n";

   $tree->traverse($tree->root, 'in-order');

   echo "\n";

   $tree->traverse($tree->root, 'post-order');

```

这将在我们的命令行中产生以下输出：

```php
10 3 6 8 12 13 15 36

3 6 8 10 12 13 15 36

3 6 8 12 13 15 36 10

```

# 不同树数据结构的复杂性

到目前为止，我们已经看到了不同的树类型及其操作。不可能逐一介绍每种树类型及其不同的操作，因为这将超出本书的范围。我们希望对其他树结构及其操作复杂性有一个最基本的了解。下面是一个包含不同类型树的平均和最坏情况下操作复杂度以及空间的图表。根据我们的需求，我们可能需要选择不同的树结构：

![](img/Image00056.jpg)

# 总结

在本章中，我们详细讨论了非线性数据结构。您了解到树是分层数据结构，有不同的树类型、操作和复杂性。我们还看到了如何定义二叉搜索树。这对于实现不同的搜索技术和数据存储将非常有用。在下一章中，我们将把重点从数据结构转移到算法上。我们将专注于第一类算法--排序算法。
