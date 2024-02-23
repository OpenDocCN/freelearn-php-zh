# 将图应用到实际中

图是用于解决各种现实问题的最有趣的数据结构之一。无论是在地图上显示方向，寻找最短路径，规划复杂的网络流量，寻找社交媒体中的个人资料之间的联系或推荐，我们都在处理图数据结构及其相关算法。图给我们提供了解决问题的许多方法，因此它们经常被用来解决复杂问题。因此，我们非常重要的是要理解图以及我们如何在解决方案中使用它们。

# 理解图的属性

图是通过边连接在一起的顶点或节点的集合。这些边可以是有序的或无序的，这意味着边可以有与之相关的方向，也可以是无向的，也称为双向边。我们使用集合*G*与顶点*V*和边*E*的关系来表示图，如下所示：

*G = (V, E)*

![](img/Image00069.jpg)

在前面的图中，我们有五个顶点和六条边：

*V = {A, B, C, D, E}*

*E = {AB, AC, AD, BD, BE, CD, DE}*

如果我们考虑前面的图，A 和 B 之间的连接可以表示为 AB 或 BA，因为我们没有定义连接的方向。图和树数据结构之间的一个重要区别是，图可以形成循环，但树数据结构不能。与树数据结构不同，我们可以从图数据结构中的任何顶点开始。此外，我们可以在任何两个顶点之间有直接的边，而在树中，只有在子节点是父节点的直接后代时，两个节点才能连接。

图有不同的属性和与之相关的关键词。在继续讨论图及其应用之前，我们将探讨这些术语。

# 顶点

图中的每个节点称为一个顶点。通常，顶点表示为一个圆。在我们的图中，节点 A，B，C，D 和 E 是顶点。

# 边

边是两个顶点之间的连接。通常，它由两个顶点之间的线表示。在前面的图中，我们在 A 和 B 之间，A 和 C 之间，A 和 D 之间，B 和 D 之间，C 和 D 之间，B 和 E 之间，以及 D 和 E 之间有边。我们可以表示边为 AB 或（A，B）。边可以有三种类型：

+   **有向边**：如果一条边标有箭头，那么它表示一条有向边。有向边是单向的。箭头的头部是终点，箭头的尾部是起点：

![](img/Image00070.gif)

在前面的图中，我们可以看到 A 有一个指向 B 的有向边，这意味着 A，B 是一条边，但反之不成立（B，A）。因此，这是一个单向边或有向边的例子。

+   无向边：无向边是两个顶点之间没有方向的连接。这意味着边满足双向关系。下图是无向图的一个例子，其中 A 与 B 连接的方式是（A，B）和（B，A）是相同的：

![](img/Image00071.jpg)

+   **加权边**：当一条边携带额外信息，如成本、距离或其他信息时，我们称该边为加权边。这用于许多图算法。在下图中，边（A，B）的权重为 5。根据图的定义，这可以是距离、成本或其他任何东西：

![](img/Image00072.gif)

# 邻接

如果两个顶点之间有一条边，则它们是相邻的。如果顶点 A 和 B 之间有直接的边，则它们被称为相邻。在下图中，我们可以看到顶点 1 和顶点 2 通过边 e1 相连，因此它们被称为相邻。由于顶点 2 与顶点 3 和 4 之间没有边，所以顶点 2 不与顶点 3 和顶点 4 相邻。

# 关联

如果顶点是边的端点之一，则边与顶点相关。此外，如果两条边共享一个顶点，则两条边是相关的。如果考虑下图，我们可以看到边(e1，e2)，(e2，e3)和(e1，e3)共享顶点 1。我们还有边(e3，e4)共享顶点 4，以及边(e2，e4)共享顶点 3。类似地，我们可以说顶点 1 与边 e1，e2 和 e3 相关，顶点 2 与边 e1 相关，顶点 3 与边 e2 和 e4 相关，顶点 4 与边 e3 和 e4 相关：

![](img/Image00073.jpg)

# 入度和出度

特定顶点的入边总数称为该顶点的入度，特定顶点的出边总数称为该顶点的出度。如果考虑下图的有向边，我们可以说顶点 A 的入度为 0，出度为 1，顶点 B 的入度为 2，出度为 1，顶点 C 的入度为 1，出度为 1，顶点 D 的入度为 1，出度为 1，顶点 E 的入度为 1，出度为 2，最后，顶点 F 的入度为 1，出度为 0。

# 路径

路径是从起始顶点到我们试图到达的另一个顶点的顶点和边的序列。在下图中，从 A 到 F 的路径由(A，B)，(B，C)，(C，E)和(E，F)表示：

![](img/Image00074.gif)

# 图的类型

根据它们的绘制或表示方式，有不同类型的图可用。每种类型的图都有不同的行为和用途。我们将重点讨论四种主要类型的图。

# 有向图

如果图只包含有向边，则图称为有向图。有向图也称为有向图或有向网络。下图表示了一个有向图。这里，(A，B)，(B，C)，(C，E)，(E，D)，(E，F)和(D，B)边是有向边。由于边是有向的，边 AB 与边 BA 不同：

![](img/Image00075.gif)

# 无向图

如果图只包含无向边，则图是无向图。换句话说，无向图中的边是双向的。有时，无向图也被称为无向网络。在无向图中，如果顶点 A 连接到顶点 B，则假定(A，B)和(B，A)表示相同的边。下图显示了一个无向图的示例，其中所有边都没有箭头表示方向：

![](img/Image00076.jpg)

# 加权图

如果图的所有边都是加权边，则图称为加权图。我们将在接下来的部分中详细讨论加权图。加权图可以是有向图或无向图。每条边必须有一个与之关联的值。边的权重总是被称为边的成本。下图表示了一个具有五个顶点和七条边的无向加权图。这里，顶点 1 和 2 之间的边的权重为 2，顶点 1 和 4 之间的边的权重为 5，顶点 4 和 5 之间的边的权重为 58：

![](img/Image00077.jpg)

# 有向无环图（DAG）

无环图是一种没有循环或环路的图。如果我们想从特定节点访问其他节点，我们不会访问任何节点两次。有向无环图，通常称为 DAG，是一个无环的有向图。有向无环图在图算法中有许多用途。有向无环图具有拓扑排序，其中顶点的排序使得每条边的起始端点在排序中出现在边的结束端点之前。以下图表示一个 DAG：

![](img/Image00078.jpg)

乍一看，似乎 B，C，E 和 D 形成一个循环，但仔细观察表明它们并没有形成循环，而我们在有向图部分使用的示例是循环图的完美示例。

# 在 PHP 中表示图

由于图是由顶点和边表示的，我们必须考虑两者来表示图。表示图的方法有几种，但最流行的方法如下：

+   邻接表

+   邻接矩阵

# 邻接表

我们可以使用链表表示图，其中一个数组将用于顶点，每个顶点将有一个链表，表示相邻顶点之间的边。当以邻接表表示时，示例图如下：

![](img/Image00079.jpg)

# 邻接矩阵

在邻接矩阵中，我们使用二维数组表示图，其中每个节点在水平和垂直方向上表示数组索引。如果从 A 到 B 的边是有方向的，则将该数组索引[A][B]标记为 1 以标记连接；否则为 0。如果边是无方向的，则[A][B]和[B][A]都设置为 1。如果图是加权图，则[A][B]或[B][A]将存储权重而不是 1。以下图显示了使用矩阵表示的无向图表示： 

![](img/Image00080.jpg)

这个图显示了矩阵的有向图表示：

![](img/Image00081.jpg)

虽然我们的图表示显示了邻接表和矩阵中数组索引的字母表示，但我们也可以使用数字索引来表示顶点。

# 重新讨论图的 BFS 和 DFS

我们已经看到了如何在树结构中实现广度优先搜索（BFS）和深度优先搜索（DFS）。我们将重新讨论我们的 BFS 和 DFS 用于图。树实现和图实现之间的区别在于，在图实现中，我们可以从任何顶点开始，而在树数据结构中，我们从树的根开始。另一个重要的考虑因素是，我们的图可以有循环，而树中没有循环，因此我们不能重新访问一个节点或顶点，否则会陷入无限循环。我们将使用一个称为图着色的概念，其中我们使用颜色或值来保持不同节点访问的状态，以保持简单。现在让我们编写一些代码来实现图中的 BFS 和 DFS。

# 广度优先搜索

现在我们将实现图的 BFS。考虑以下无向图，首先，我们需要用矩阵或列表表示图。为了简单起见，我们将使用邻接矩阵表示图：

![](img/Image00082.jpg)

前面的邻接图有六个顶点，顶点从 1 到 6 标记（没有 0）。由于我们的顶点编号，我们可以将它们用作数组索引以加快访问速度。我们可以构建图如下：

```php
$graph = []; 

$visited = []; 

$vertexCount = 6; 

for($i = 1;$i<=$vertexCount;$i++) { 

    $graph[$i] = array_fill(1, $vertexCount, 0); 

    $visited[$i] = 0; 

} 

```

在这里，我们有两个数组，一个用于表示实际图形，另一个用于跟踪已访问的节点。我们希望确保我们不会多次访问一个节点，因为这可能会导致无限循环。由于我们的图形有六个顶点，我们将`$vertexCount`保持为`6`。然后，我们将图数组初始化为具有初始值`0`的二维数组。我们将从数组的索引`1`开始。我们还将通过将每个顶点分配给`$visited`数组中的`0`来设置每个顶点为未访问状态。现在，我们将在我们的图形表示中添加边。由于图是无向的，我们需要为每条边设置两个属性。换句话说，我们需要为标记为 1 和 2 的顶点之间的边设置双向边值，因为它们之间共享一条边。以下是先前图形的完整表示的代码：

```php
$graph[1][2] = $graph[2][1] = 1; 

$graph[1][5] = $graph[5][1] = 1; 

$graph[5][2] = $graph[2][5] = 1; 

$graph[5][4] = $graph[4][5] = 1; 

$graph[4][3] = $graph[3][4] = 1; 

$graph[3][2] = $graph[2][3] = 1; 

$graph[6][4] = $graph[4][6] = 1; 

```

因此，我们已经使用邻接矩阵表示了图。现在，让我们为矩阵定义 BFS 算法：

```php
function BFS(array &$graph, int $start, array $visited): SplQueue { 

    $queue = new SplQueue;

    $path = new SplQueue;

    $queue->enqueue($start);

    $visited[$start] = 1;

    while (!$queue->isEmpty()) { 

      $node = $queue->dequeue();

      $path->enqueue($node);

      foreach ($graph[$node] as $key => $vertex) { 

          if (!$visited[$key] && $vertex == 1) { 

          $visited[$key] = 1;

          $queue->enqueue($key);

          }

      }

    }

    return $path;

}

```

我们实现的 BFS 函数接受三个参数：实际图形、起始顶点和空的已访问数组。我们本可以避免第三个参数，并在 BFS 函数内部进行初始化。归根结底，我们可以选择任一种方式来完成这一点。在我们的函数实现中，有两个队列：一个用于保存我们需要访问的节点，另一个用于保存已访问节点的顺序，或者搜索的路径。在函数结束时，我们返回路径队列。

在函数内部，我们首先将起始节点添加到队列中。然后，我们从该节点开始访问其相邻节点。如果节点未被访问并且与当前节点有连接，则将其添加到我们的访问队列中。我们还将当前节点标记为已访问，并将其添加到我们的路径中。现在，我们将使用我们构建的图矩阵和一个访问节点来调用我们的 BFS 函数。以下是执行 BFS 功能的程序：

```php
$path = BFS($graph, 1, $visited); 

while (!$path->isEmpty()) { 

    echo $path->dequeue()."\t"; 

} 

```

从前面的代码片段中可以看出，我们从节点 1 开始搜索。输出将如下所示：

```php
    1       2       5       3       4       6

```

如果我们将`BFS`函数调用的第二个参数从 1 更改为 5 作为起始节点，那么输出将如下所示：

```php
    5       1       2       4       3       6

```

# 深度优先搜索

正如我们在 BFS 中看到的那样，我们也可以为 DFS 定义任何起始顶点。不同之处在于，对于已访问节点的列表，我们将使用堆栈而不是队列。代码的其他部分将类似于我们的 BFS 代码。我们还将使用与 BFS 实现相同的图。我们将实现的 DFS 是迭代的。以下是其代码：

```php
function DFS(array &$graph, int $start, array $visited): SplQueue { 

    $stack = new SplStack; 

    $path = new SplQueue; 

    $stack->push($start); 

    $visited[$start] = 1; 

    while (!$stack->isEmpty()) { 

      $node = $stack->pop(); 

      $path->enqueue($node); 

      foreach ($graph[$node] as $key => $vertex) { 

          if (!$visited[$key] && $vertex == 1) { 

          $visited[$key] = 1; 

          $stack->push($key); 

          } 

      } 

    } 

    return $path; 

} 

```

如前所述，对于 DFS，我们必须使用堆栈而不是队列，因为我们需要从堆栈中获取最后一个顶点，而不是第一个（如果我们使用了队列）。对于路径部分，我们使用队列，以便在显示过程中按顺序显示路径。以下是调用我们的图`$graph`的代码：

```php
$path = DFS($graph, 1, $visited); 

while (!$path->isEmpty()) { 

    echo $path->dequeue()."\t"; 

} 

```

该代码将产生以下输出：

```php
    1       5       4       6       3       2

```

对于上述示例，我们从顶点 1 开始，并首先访问顶点 5，这是顶点 1 的两个相邻顶点中标记为 5 和 2 的顶点之一。现在，顶点 5 有两个标记为 4 和 2 的顶点。顶点 4 将首先被访问，因为它是从顶点 5 出发的第一条边（记住我们从左到右访问节点的方向）。接下来，我们将从顶点 4 访问顶点 6。由于我们无法从顶点 6 继续前进，它将返回到顶点 4 并访问标记为 3 的未访问相邻顶点。当我们到达顶点 3 时，有两个相邻顶点可供访问。它们被标记为顶点 4 和顶点 2。我们之前已经访问了顶点 4，因此无法重新访问它，我们必须从顶点 3 访问顶点 2。由于顶点 2 有三个顶点，分别是顶点 3、5 和 1，它们都已经被访问，因此我们实际上已经完成了 DFS 的实现。

如果我们从一个起始顶点寻找特定的终点顶点，我们可以传递一个额外的参数。在之前的例子中，我们只是获取相邻的顶点并访问它们。对于特定的终点顶点，我们需要在 DFS 算法的迭代过程中将目标顶点与我们访问的每个顶点进行匹配。

# 使用 Kahn 算法进行拓扑排序

假设我们有一些任务要做，每个任务都有一些依赖关系，这意味着在执行实际任务之前，应该先完成依赖的任务。当任务和依赖之间存在相互关系时，问题就出现了。现在，我们需要找到一个合适的顺序来完成这些任务。我们需要一种特殊类型的排序，以便在不违反完成任务的规则的情况下对这些相互关联的任务进行排序。拓扑排序将是解决这类问题的正确选择。在拓扑排序中，从顶点 A 到 B 的有向边 AB 被排序，以便 A 始终在排序中位于 B 之前。这将适用于所有的顶点和边。应用拓扑排序的另一个重要因素是图必须是一个 DAG。任何 DAG 都至少有一个拓扑排序。大多数情况下，对于给定的图，可能存在多个拓扑排序。有两种流行的算法可用于拓扑排序：Kahn 算法和 DFS 方法。我们将在这里讨论 Kahn 算法，因为我们在本书中已经多次讨论了 DFS。

Kahn 算法有以下步骤来从 DAG 中找到拓扑排序：

1.  计算每个顶点的入度（入边），并将所有入度为 0 的顶点放入队列中。还要将访问节点的计数初始化为 0。

1.  从队列中移除一个顶点，并对其执行以下操作：

1. 将访问节点计数加 1。

2. 将所有相邻顶点的入度减 1。

3. 如果相邻顶点的入度变为 0，则将其添加到队列中。

1.  重复*步骤 2*，直到队列为空。

1.  如果访问节点的计数与节点的计数不同，则给定 DAG 的拓扑排序是不可能的。

让我们考虑以下图。这是一个 DAG 的完美例子。现在，我们想使用拓扑排序和 Kahn 算法对其进行排序：

![](img/Image00083.jpg)

现在让我们使用邻接矩阵来表示这个图，就像我们之前为其他图所做的那样。矩阵将如下所示：

```php
$graph = [ 

    [0, 0, 0, 0, 1], 

    [1, 0, 0, 1, 0], 

    [0, 1, 0, 1, 0], 

    [0, 0, 0, 0, 0], 

    [0, 0, 0, 0, 0], 

];

```

现在，我们将按照我们定义的步骤实现 Kahn 算法。以下是它的实现：

```php
function topologicalSort(array $matrix): SplQueue { 

    $order = new SplQueue; 

    $queue = new SplQueue; 

    $size = count($matrix); 

    $incoming = array_fill(0, $size, 0); 

    for ($i = 0; $i < $size; $i++) { 

      for ($j = 0; $j < $size; $j++) { 

          if ($matrix[$j][$i]) { 

          $incoming[$i] ++; 

          } 

      } 

      if ($incoming[$i] == 0) { 

          $queue->enqueue($i); 

      } 

    } 

    while (!$queue->isEmpty()) { 

      $node = $queue->dequeue(); 

      for ($i = 0; $i < $size; $i++) { 

          if ($matrix[$node][$i] == 1) { 

            $matrix[$node][$i] = 0; 

            $incoming[$i] --; 

            if ($incoming[$i] == 0) { 

                $queue->enqueue($i); 

            } 

          } 

      } 

      $order->enqueue($node); 

    } 

    if ($order->count() != $size) // cycle detected 

      return new SplQueue; 

    return $order; 

} 

```

从前面的实现中可以看出，我们实际上考虑了我们提到的 Kahn 算法的每一步。我们首先找到了顶点的入度，并将入度为 0 的顶点放入了队列中。然后，我们检查了队列的每个节点，并减少了相邻顶点的入度，并再次将任何入度为 0 的相邻顶点添加到队列中。最后，我们返回了排序后的队列，或者如果有序顶点的计数与实际顶点的计数不匹配，则返回一个空队列。现在，我们可以调用该函数来返回排序后的顶点列表作为队列。以下是执行此操作的代码：

```php
$sorted = topologicalSort($graph);

while (!$sorted->isEmpty()) {

    echo $sorted->dequeue() . "\t";

} 

```

现在，这将遍历队列中的每个元素并将它们打印出来。输出将如下所示：

```php
    2       1       0 

      3       4

```

输出符合我们的期望。从之前的图表中可以看出，顶点 **2** 直接连接到顶点 **1** 和顶点 **3** ，顶点 **1** 直接连接到顶点 **0** 和顶点 **3** 。由于顶点 **2** 没有入边，我们将从顶点 **2** 开始进行拓扑排序。顶点 **1** 有一个入边，顶点 **3** 有两个入边，所以在顶点 **2** 之后，我们将按照算法访问顶点 **1** 。相同的原则将带我们到顶点 **0** ，然后是顶点 **3** ，最后是顶点 **4** 。我们还必须记住对于给定的图，可能存在多个拓扑排序。Kahn 算法的复杂度是 **O** (*V+E* )，其中 **V** 是顶点的数量，**E** 是边的数量。

# 使用 Floyd-Warshall 算法的最短路径

披萨外卖公司的常见情景是尽快送达披萨。图算法可以帮助我们在这种情况下。Floyd-Warshall 算法是一种非常常见的算法，用于找到从 u 到 v 的最短路径，使用所有顶点对(u, v)。最短路径表示两个相互连接的节点之间的最短可能距离。用于计算最短路径的图必须是加权图。在某些情况下，权重也可以是负数。该算法非常简单，也是最容易实现的之一。它在这里显示：

```php
for i:= 1 to n do 

  for j:= 1 to n do 

     dis[i][j] = w[i][j] 

for k:= 1 to n do 

   for i:= 1 to n do 

      for j:= 1 to n do 

         sum := dis[i][k] + dis[k][j] 

         if (sum < dis[i][j]) 

              dis[i][j] := sum 

```

首先，我们将每个权重复制到一个成本或距离矩阵中。然后，我们遍历每个顶点，并计算从顶点 `i` 经过顶点 `k` 到达顶点 `j` 的成本或距离。如果距离或成本小于顶点 `i` 到顶点 `j` 的直接路径，我们选择路径 `i` 到 `k` 到 `j` 而不是直接路径 `i` 到 `j` 。让我们考虑以下图表：

![](img/Image00084.gif)

在这里，我们可以看到一个带有每条边权重的无向图。现在，如果我们寻找从 **A** 到 **E** 的最短路径，那么我们有以下选项：

+   **A** 到 **E** 通过 **B** 的距离为 **20**

+   **A** 到 **E** 通过 **D** 的距离为 **25**

+   **A** 到 **E** 通过 **D** 和 **B** 的距离为 **20**

+   **A** 到 **E** 通过 **B** 和 **D** 的距离为 **35**

因此，我们可以看到最小距离是 **20** 。现在，让我们以数值表示顶点，以编程方式实现这一点。我们将使用 0、1、2、3 和 4 代替 A、B、C、D 和 E。现在，让我们用邻接矩阵格式表示之前的图：

```php
$totalVertices = 5; 

$graph = []; 

for ($i = 0; $i < $totalVertices; $i++) { 

    for ($j = 0; $j < $totalVertices; $j++) { 

      $graph[$i][$j] = $i == $j ? 0 : PHP_INT_MAX; 

    }

}

```

在这里，我们采取了不同的方法，并将所有边初始化为 PHP 整数的最大值。这样做的原因是确保非边的值为 0 不会影响算法逻辑，因为我们正在寻找最小值。现在，我们需要像之前的图表中显示的那样向图中添加权重：

```php
$graph[0][1] = $graph[1][0] = 10;

$graph[2][1] = $graph[1][2] = 5;

$graph[0][3] = $graph[3][0] = 5;

$graph[3][1] = $graph[1][3] = 5;

$graph[4][1] = $graph[1][4] = 10;

$graph[3][4] = $graph[4][3] = 20;

```

由于这是一个无向图，我们给两条边分配相同的值。如果是有向图，我们只能为每个权重制作一次输入。现在，是时候实现 Floyd-Warshall 算法，以找到任意一对节点的最短路径。这是我们对该函数的实现：

```php
function floydWarshall(array $graph): array {

    $dist = [];

    $dist = $graph;

    $size = count($dist);

    for ($k = 0; $k < $size; $k++)

      for ($i = 0; $i < $size; $i++)

          for ($j = 0; $j < $size; $j++)

        $dist[$i][$j] = min($dist[$i][$j],

    $dist[$i][$k] + $dist[$k][$j]);

    return $dist;

} 

```

正如我们之前提到的，实现非常简单。我们有三个内部循环来计算最小距离，并且在函数结束时返回距离数组。现在，让我们调用这个函数并检查我们的预期结果是否匹配：

```php
$distance = floydWarshall($graph); 

echo "Shortest distance between A to E is:" . $distance[0][4] . "\n"; 

echo "Shortest distance between D to C is:" . $distance[3][2] . "\n"; 

```

以下是代码的输出：

```php
Shortest distance between A to E is:20

Shortest distance between D to C is:10

```

如果我们检查之前的图表，我们可以看到 **D** 和 **C** 之间的最短距离实际上是 **10** ，路径是 D → B → C (5+5)，这是所有可能路线中的最短距离 (D → A → B → C (20)，或 D → E → B → C (35))。

Floyd-Warshall 算法的复杂度为 **O** (*V3* )，其中 **V** 是图中顶点的数量。现在我们将探讨另一个以找到单源最短路径而闻名的算法。

# 使用 Dijkstra 算法的单源最短路径

我们可以很容易地使用 Floyd-Warshall 算法找到最短路径，但我们无法得到从节点 X 到 Y 的实际路径。这是因为 Floyd-Warshall 算法计算距离或成本，不存储最小成本的实际路径。例如，使用 Google 地图，我们总是可以找到从任何给定位置到目的地的路线。Google 地图可以显示最佳路线，关于距离、旅行时间或其他因素。这是单源最短路径算法使用的完美例子。有许多算法可以找到单源最短路径问题的解决方案；然而，Dijkstra 最短路径算法是最流行的。有许多实现 Dijkstra 算法的方法，例如使用斐波那契堆、最小堆、优先队列等。每种实现都有其自身的优势，关于 Dijkstra 解决方案的性能和改进。让我们来看一下算法的伪代码：

```php
   function Dijkstra(Graph, source):

      create vertex set Q

      for each vertex v in Graph:   

          dist[v] := INFINITY

          prev[v] := UNDEFINED          

          add v to Q         

      dist[source] := 0           

      while Q is not empty:

          u := vertex in Q with min dist[u]

          remove u from Q

          for each neighbor v of u:

              alt := dist[u] + length(u, v)

              if alt < dist[v]:   

                  dist[v] := alt

                  prev[v] := u

      return dist[], prev[]

```

现在，我们将使用优先队列来实现算法。首先，让我们选择一个图来实现算法。我们可以选择以下无向加权图。它有六个节点，节点和顶点之间有许多连接。首先，我们需要用邻接矩阵表示以下图：

![](img/Image00085.jpg)

从前面的图表中可以看出，我们的顶点用字母**A**到**F**标记，因此我们将使用顶点名称作为 PHP 关联数组中的键：

```php
$graph = [

    'A' => ['B' => 3, 'C' => 5, 'D' => 9],

    'B' => ['A' => 3, 'C' => 3, 'D' => 4, 'E' => 7],

    'C' => ['A' => 5, 'B' => 3, 'D' => 2, 'E' => 6, 'F' => 3],

    'D' => ['A' => 9, 'B' => 4, 'C' => 2, 'E' => 2, 'F' => 2],

    'E' => ['B' => 7, 'C' => 6, 'D' => 2, 'F' => 5],

    'F' => ['C' => 3, 'D' => 2, 'E' => 5],

];

```

现在，我们将使用优先队列来实现 Dijkstra 算法。我们将使用我们为上一个图表创建的邻接矩阵来找到从源顶点到目标顶点的路径。我们的 Dijkstra 算法将返回一个数组，其中包括两个节点之间的最小距离和所遵循的路径。我们将路径返回为一个栈，以便我们可以按相反顺序获取实际路径。以下是实现：

```php
function Dijkstra(array $graph, string $source,string $target):array{ 

    $dist = []; 

    $pred = []; 

    $Queue = new SplPriorityQueue(); 

    foreach ($graph as $v => $adj) { 

      $dist[$v] = PHP_INT_MAX; 

      $pred[$v] = null; 

      $Queue->insert($v, min($adj)); 

    } 

    $dist[$source] = 0; 

    while (!$Queue->isEmpty()) { 

      $u = $Queue->extract(); 

      if (!empty($graph[$u])) { 

          foreach ($graph[$u] as $v => $cost) { 

           if ($dist[$u] + $cost < $dist[$v]) { 

            $dist[$v] = $dist[$u] + $cost; 

            $pred[$v] = $u; 

        } 

          } 

      } 

    } 

    $S = new SplStack();

    $u = $target; 

    $distance = 0;

    while (isset($pred[$u]) && $pred[$u]) {

      $S->push($u);

      $distance += $graph[$u][$pred[$u]];

      $u = $pred[$u]; 

    } 

    if ($S->isEmpty()) { 

      return ["distance" => 0, "path" => $S]; 

    } else {

      $S->push($source);

      return ["distance" => $distance, "path" => $S]; 

    }

}

```

从前面的实现中可以看出，首先，我们创建了两个数组来存储距离和前任，以及优先队列。然后，我们将每个顶点设置为 PHP 的最大整数（`PHP_INT_MAX`）值（伪代码中的 INFINITY）和前任为`NULL`。我们还取了所有相邻节点的最小值并将它们存储在队列中。循环结束后，我们将源节点的距离设置为`0`。然后我们检查队列中的每个节点，并检查最近的邻居以找到最小路径。如果使用`if ($dist[$u] + $cost < $dist[$v])`找到了路径，我们将其分配给该顶点。

然后我们创建了一个名为`$s`的栈来存储路径。我们从目标顶点开始，访问相邻的顶点以到达源顶点。当我们通过相邻的顶点移动时，我们还计算了通过访问这些顶点所覆盖的距离。由于我们的函数返回了距离和路径，我们构造了一个数组来返回给定图、源和目标的距离和路径。如果没有路径存在，我们将返回距离为 0，并返回一个空栈作为输出。现在，我们将写几行代码来使用图`$graph`和函数`Dijkstra`来检查我们的实现：

```php
$source = "A"; 

$target = "F"; 

$result = Dijkstra($graph, $source, $target); 

extract($result); 

echo "Distance from $source to $target is $distance \n"; 

echo "Path to follow : "; 

while (!$path->isEmpty()) { 

    echo $path->pop() . "\t"; 

} 

```

如果我们运行这段代码，它将在命令行中输出以下内容：

```php
Distance from A to F is 8

Path to follow : A      C       F

```

输出看起来完全正确，从图表中我们可以看到从**A**到**F**的最短路径是通过**C**，最短距离是*5 + 3 = 8*。

Dijkstra 算法的运行复杂度为**O**(*V2*)。由于我们使用了最小优先队列，运行时复杂度为**O**(*E + V log V*)。

# 使用 Bellman-Ford 算法找到最短路径

尽管 Dijkstra 算法是最流行和高效的用于找到单源最短路径的算法，但它没有解决一个问题。如果图中有一个负循环，Dijkstra 算法无法检测到负循环，因此它无法工作。负循环是一个循环，其中所有边的总和为负。如果一个图包含一个负循环，那么找到最短路径将是不可能的，因此在寻找最短路径时解决这个问题是很重要的。这就是为什么我们使用 Bellman-Ford 算法，尽管它比 Dijkstra 算法慢。以下是 Bellman-Ford 算法寻找最短路径的算法伪代码：

```php
function BellmanFord(list vertices, list edges, vertex source) 

  // This implementation takes a vertex source 

  // and fills distance array with shortest-path information 

  // Step 1: initialize graph 

  for each vertex v in vertices: 

    if v is source 

      distance[v] := 0 

    else 

      distance[v] := infinity 

  // Step 2: relax edges repeatedly 

  for i from 1 to size(vertices)-1: 

    for each edge (u, v) with weight w in edges: 

      if distance[u] + w < distance[v]: 

        distance[v] := distance[u] + w 

  // Step 3: check for negative-weight cycles 

    for each edge (u, v) with weight w in edges: 

        if distance[u] + w < distance[v]: 

      error "Graph contains a negative-weight cycle" 

```

我们可以看到 Bellman-Ford 算法在寻找节点之间的最短路径时也考虑了边和顶点。这被称为松弛过程，在 Dijkstra 算法中也使用。图算法中的松弛过程是指如果通过*V*的路径包括*V*，则更新与顶点*V*连接的所有顶点的成本。简而言之，松弛过程试图通过另一个顶点降低到达一个顶点的成本。现在，我们将为我们在 Dijkstra 算法中使用的相同图实现这个算法。唯一的区别是这里我们将为我们的节点和顶点使用数字标签：

![](img/Image00086.jpg)

现在是时候以邻接矩阵格式表示图了。以下是 PHP 中的矩阵：

```php
$graph = [ 

    0 => [0, 3, 5, 9, 0, 0], 

    1 => [3, 0, 3, 4, 7, 0], 

    2 => [5, 3, 0, 2, 6, 3], 

    3 => [9, 4, 2, 0, 2, 2], 

    4 => [0, 7, 6, 2, 0, 5], 

    5 => [0, 0, 3, 2, 5, 0] 

]; 

```

以前，我们使用值 0 表示两个顶点之间没有边。如果我们在这里做同样的事情，那么在松弛过程中，取两条边中的最小值，其中一条代表 0，将始终产生 0，这实际上意味着两个顶点之间没有连接。因此，我们必须选择一个更大的数字来表示不存在的边。我们可以使用 PHP 的`MAX_INT_VALUE`常量来表示这些边，以便这些不存在的边不被考虑。这可以成为我们新的图表示：

```php
define("I", PHP_INT_MAX); 

$graph = [ 

    0 => [I, 3, 5, 9, I, I], 

    1 => [3, I, 3, 4, 7, I], 

    2 => [5, 3, I, 2, 6, 3], 

    3 => [9, 4, 2, I, 2, 2], 

    4 => [I, 7, 6, 2, I, 5], 

    5 => [I, I, 3, 2, 5, I] 

]; 

```

现在，让我们为 Bellman-Ford 算法编写实现。我们将使用在伪代码中定义的相同方法：

```php
function bellmanFord(array $graph, int $source): array { 

    $dist = []; 

    $len = count($graph); 

    foreach ($graph as $v => $adj) { 

      $dist[$v] = PHP_INT_MAX; 

    } 

    $dist[$source] = 0; 

    for ($k = 0; $k < $len - 1; $k++) { 

      for ($i = 0; $i < $len; $i++) { 

          for ($j = 0; $j < $len; $j++) { 

            if ($dist[$i] > $dist[$j] + $graph[$j][$i]) { 

            $dist[$i] = $dist[$j] + $graph[$j][$i]; 

        } 

          } 

      } 

    } 

    for ($i = 0; $i < $len; $i++) { 

      for ($j = 0; $j < $len; $j++) { 

          if ($dist[$i] > $dist[$j] + $graph[$j][$i]) { 

           echo 'The graph contains a negative-weight cycle!'; 

           return []; 

          } 

      } 

        } 

    return $dist; 

} 

```

与 Dijkstra 算法不同的是，我们不是在跟踪前任。我们在松弛过程中考虑距离。由于我们在 PHP 中使用整数的最大值，它自动取消了选择值为 0 的不存在边作为最小路径的可能性。实现的最后部分检测给定图中的任何负循环，并在这种情况下返回一个空数组：

```php
$source = 0; 

$distances = bellmanFord($graph, $source); 

foreach($distances as $target => $distance) { 

    echo "distance from $source to $target is $distance \n"; 

} 

```

这将产生以下输出，显示了从我们的源节点到其他节点的最短路径距离：

```php
distance from 0 to 0 is 0

distance from 0 to 1 is 3

distance from 0 to 2 is 5

distance from 0 to 3 is 7

distance from 0 to 4 is 9

distance from 0 to 5 is 8

```

Bellman-Ford 算法的运行时间复杂度为**O**(*V*, *E*)。

# 理解最小生成树（MST）

假设我们正在设计一个新的办公园区，其中有多栋建筑相互连接。如果我们考虑每栋建筑之间的互联性，将需要大量的电缆。然而，如果我们能够通过一种共同的连接方式将所有建筑物连接起来，其中每栋建筑物只与其他建筑物通过一个连接相连，那么这个解决方案将减少冗余和成本。如果我们把我们的建筑看作顶点，建筑之间的连接看作边，我们可以使用这种方法构建一个图。我们试图解决的问题也被称为**最小生成树**或**MST**。考虑以下图。我们有 10 个顶点和 21 条边。然而，我们可以用只有九条边（黑线）连接所有 10 个顶点。这将使我们的成本或距离保持在最低水平：

![](img/Image00087.jpg)

有几种算法可以用来从给定的图中找到最小生成树。最流行的两种是 Prim 算法和 Kruskal 算法。我们将在接下来的部分探讨这两种算法。

# 实现 Prim 生成树算法

Prim 算法用于寻找最小生成树依赖于贪婪方法。贪婪方法被定义为一种算法范例，其中我们尝试通过考虑每个阶段的局部最优解来找到全局最优解。我们将在第十一章中探讨贪婪算法，*使用高级技术解决问题*。在贪婪方法中，算法创建边的子集，并找出子集中成本最低的边。这个边的子集将包括所有顶点。它从任意位置开始，并通过选择顶点之间最便宜的可能连接来逐个顶点地扩展树。让我们考虑以下图：

![](img/Image00088.jpg)

现在，我们将应用 Prim 算法的一个非常基本的版本，以获得最小生成树以及边的最小成本或权重。图将看起来像这样，作为邻接矩阵：

```php
$G = [ 

    [0, 3, 1, 6, 0, 0], 

    [3, 0, 5, 0, 3, 0], 

    [1, 5, 0, 5, 6, 4], 

    [6, 0, 5, 0, 0, 2], 

    [0, 3, 6, 0, 0, 6], 

    [0, 0, 4, 2, 6, 0] 

]; 

```

现在，我们将实现 Prim 最小生成树的算法。我们假设我们将从顶点 0 开始找出整个生成树，因此我们只需将图的邻接矩阵传递给函数，它将显示生成树的连接边以及最小成本：

```php
function primMST(array $graph) { 

    $parent = [];   // Array to store the MST 

    $key = [];     // used to pick minimum weight edge         

    $visited = [];   // set of vertices not yet included in MST 

    $len = count($graph); 

    // Initialize all keys as MAX 

    for ($i = 0; $i < $len; $i++) { 

      $key[$i] = PHP_INT_MAX; 

      $visited[$i] = false; 

    } 

    $key[0] = 0; 

    $parent[0] = -1; 

    // The MST will have V vertices 

    for ($count = 0; $count < $len - 1; $count++) { 

  // Pick the minimum key vertex 

  $minValue = PHP_INT_MAX; 

  $minIndex = -1; 

  foreach (array_keys($graph) as $v) { 

      if ($visited[$v] == false && $key[$v] < $minValue) { 

        $minValue = $key[$v]; 

        $minIndex = $v; 

      } 

  } 

  $u = $minIndex; 

  // Add the picked vertex to the MST Set 

  $visited[$u] = true; 

  for ($v = 0; $v < $len; $v++) { 

      if ($graph[$u][$v] != 0 && $visited[$v] == false && 

        $graph[$u][$v] < $key[$v]) { 

          $parent[$v] = $u; 

          $key[$v] = $graph[$u][$v]; 

      } 

  } 

    } 

    // Print MST 

    echo "Edge\tWeight\n"; 

    $minimumCost = 0; 

    for ($i = 1; $i < $len; $i++) { 

      echo $parent[$i] . " - " . $i . "\t" . $graph[$i][$parent[$i]] 

         "\n"; 

      $minimumCost += $graph[$i][$parent[$i]]; 

    } 

    echo "Minimum cost: $minimumCost \n"; 

} 

```

现在，如果我们用我们的图$G$调用函数`primMST`，则以下将是算法构建的输出和最小生成树：

```php
Edge    Weight

0 - 1   3

0 - 2   1

5 - 3   2

1 - 4   3

2 - 5   4

Minimum cost: 13

```

![](img/Image00089.jpg)

还有其他实现 Prim 算法的方法，如使用斐波那契堆、优先队列等。这与 Dijkstra 算法寻找最短路径非常相似。我们的实现具有**O**(*V²*)的时间复杂度。使用二叉堆和斐波那契堆，我们可以显著降低复杂度。

# Kruskal 算法的生成树

另一个用于寻找最小生成树的流行算法是 Kruskal 算法。它类似于 Prim 算法，并使用贪婪方法来找到解决方案。以下是我们需要实现 Kruskal 算法的步骤：

1.  创建一个森林**T**（一组树），图中的每个顶点都是一个单独的树。

1.  创建一个包含图中所有边的集合**S**。

1.  当**S**非空且**T**尚未跨越时：

1\. 从**S**中移除权重最小的边。

2\. 如果该边连接两棵不同的树，则将其添加到森林中，将两棵树合并成一棵树；否则，丢弃该边。

我们将使用与 Prim 算法相同的图。以下是 Kruskal 算法的实现：

```php
function Kruskal(array $graph): array { 

    $len = count($graph); 

    $tree = []; 

    $set = []; 

    foreach ($graph as $k => $adj) { 

    $set[$k] = [$k]; 

    } 

    $edges = []; 

    for ($i = 0; $i < $len; $i++) { 

      for ($j = 0; $j < $i; $j++) { 

        if ($graph[$i][$j]) { 

          $edges[$i . ',' . $j] = $graph[$i][$j]; 

        } 

    } 

    } 

    asort($edges); 

    foreach ($edges as $k => $w) { 

    list($i, $j) = explode(',', $k); 

    $iSet = findSet($set, $i); 

    $jSet = findSet($set, $j); 

    if ($iSet != $jSet) { 

        $tree[] = ["from" => $i, "to" => $j, 

    "cost" => $graph[$i][$j]]; 

        unionSet($set, $iSet, $jSet); 

    } 

    } 

    return $tree; 

} 

function findSet(array &$set, int $index) { 

    foreach ($set as $k => $v) { 

      if (in_array($index, $v)) { 

        return $k; 

      } 

    } 

    return false; 

} 

function unionSet(array &$set, int $i, int $j) { 

    $a = $set[$i]; 

    $b = $set[$j]; 

    unset($set[$i], $set[$j]); 

    $set[] = array_merge($a, $b); 

} 

```

正如我们所看到的，我们有两个单独的函数——`unionSet`和`findSet`——来执行两个不相交集合的并操作，以及找出一个数字是否存在于集合中。现在，让我们用我们构建的图运行程序：

```php
$graph = [ 

    [0, 3, 1, 6, 0, 0], 

    [3, 0, 5, 0, 3, 0], 

    [1, 5, 0, 5, 6, 4], 

    [6, 0, 5, 0, 0, 2], 

    [0, 3, 6, 0, 0, 6], 

    [0, 0, 4, 2, 6, 0] 

]; 

$mst = Kruskal($graph); 

$minimumCost = 0; 

foreach($mst as $v) { 

    echo "From {$v['from']} to {$v['to']} cost is {$v['cost']} \n"; 

    $minimumCost += $v['cost']; 

} 

echo "Minimum cost: $minimumCost \n"; 

```

这将产生以下输出，与我们从 Prim 算法得到的输出类似：

```php
From 2 to 0 cost is 1

From 5 to 3 cost is 2

From 1 to 0 cost is 3

From 4 to 1 cost is 3

From 5 to 2 cost is 4

Minimum cost: 13

```

Kruskal 算法的复杂度是**O**(*E log V*），这比通用的 Prim 算法实现更好。

# 总结

在本章中，我们讨论了不同的图算法及其操作。图在解决各种问题时非常方便。我们已经看到，对于相同的图，我们可以应用不同的算法并获得不同的性能。我们必须仔细选择要应用的算法，这取决于问题的性质。由于某些限制，本书中我们略过了许多其他图的主题。有一些主题，如图着色、二分匹配和流问题，应该在适用的地方进行研究和应用。在下一章中，我们将把重点转移到本书的最后一个数据结构主题，称为堆，学习堆数据结构的不同用法。
