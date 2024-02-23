# 探索搜索选项

除了排序，搜索是编程世界中最常用的算法之一。无论是搜索电话簿、电子邮件、数据库还是文件，我们实际上都在执行某种搜索技术来定位我们希望找到的项目。搜索和排序是编程中最重要的两个组成部分。在本章中，您将学习不同的搜索技术以及它们的效率。我们还将学习有关搜索树数据结构的不同搜索方式。

# 线性搜索

执行搜索的最常见方式之一是将每个项目与我们要查找的项目进行比较。这被称为线性搜索或顺序搜索。这是执行搜索的最基本方式。如果我们考虑列表中有*n*个项目，在最坏的情况下，我们必须搜索*n*个项目才能找到特定的项目。我们可以遍历列表或数组来查找项目。让我们考虑以下例子：

```php
function search(array $numbers, int $needle): bool {

    $totalItems = count($numbers);

    for ($i = 0; $i < $totalItems; $i++) {

      if($numbers[$i] === $needle){

        return TRUE;

      }

     }

    return FALSE;

}

```

我们有一个名为`search`的函数，它接受两个参数。一个是数字列表，另一个是我们要在列表中查找的数字。我们运行一个 for 循环来遍历列表中的每个项目，并将它们与我们的项目进行比较。如果找到匹配项，我们返回 true 并且不继续搜索。然而，如果循环结束并且没有找到任何东西，我们在函数定义的末尾返回 false。让我们使用`search`函数来使用以下程序查找一些东西：

```php
$numbers = range(1, 200, 5); 

if (search($numbers, 31)) { 

    echo "Found"; 

} else { 

    echo "Not found"; 

}

```

在这里，我们使用 PHP 的内置函数 range 生成一个随机数组，范围是 1 到 200。每个项目的间隔为 5，如 1、6、11、16 等；然后我们搜索 31，在列表中有 6、11、16、21、26、31 等。然而，如果我们要搜索 32 或 33，那么项目将找不到。因此，对于这种情况，我们的输出将是`Found`。

我们需要记住的一件事是，我们不必担心我们的列表是否按任何特定顺序或特定方式组织。如果我们要查找的项目在第一个位置，那将是最好的结果。最坏的结果可能是最后一个项目或不在列表中的项目。在这两种情况下，我们都必须遍历列表的所有*n*个项目。以下是线性/顺序搜索的复杂性：

| 最佳时间复杂度 | `O(1)` |
| --- | --- |
| 最坏时间复杂度 | `O(n)` |
| 平均时间复杂度 | `O(n)` |
| 空间复杂度（最坏情况） | `O(1)` |

正如我们所看到的，线性搜索的平均或最坏时间复杂度为`O(n)`，这并不会改变我们对项目列表的排序方式。现在，如果数组中的项目按特定顺序排序，那么我们可能不必进行线性搜索，而可以通过选择性或计算性搜索获得更好的结果。最流行和知名的搜索算法是"二分搜索"。是的，这听起来像你在第六章中学到的二分搜索树，*理解和实现树*，但我们甚至可以在不构建二分搜索树的情况下使用这个算法。所以，让我们来探索一下。

# 二分搜索

二分搜索是编程世界中非常流行的搜索算法。在顺序搜索中，我们从开头开始扫描每个项目以找到所需的项目。然而，如果列表已经排序，那么我们就不需要从列表的开头或结尾开始搜索。在二分搜索算法中，我们从列表的中间开始，检查中间的项目是比我们要找的项目小还是大，并决定要走哪条路。这样，我们将列表分成两半，并丢弃一半，就像下面的图片一样：

![](img/Image00063.jpg)

如果我们看前面的图片，我们有一个按升序排序的数字列表。我们想知道项目**7**是否在数组中。由于数组有 17 个项目（0 到 16 索引），我们将首先转到中间索引，对于这个示例来说是第八个索引。现在，第八个索引的值为**14**，大于我们要搜索的值**7**。这意味着如果**7**在这个数组中，它在**14**的左边，因为数字已经排序。因此，我们放弃了从第八个索引到第十六个索引的数组，因为数字不能在数组的那一部分。现在，我们重复相同的过程，并取数组剩余部分的中间部分，即剩余部分的第三个元素。现在，第三个元素的值为**6**，小于**7**。因此，我们要找的项目在剩余部分的第三个元素的右侧，而不是左侧。

现在，我们将检查数组的第四个元素到第七个元素，中间元素现在指向第五个元素。第五个元素的值为**8**，大于**7**，我们要找的值。因此，我们必须考虑第五个元素的左侧来找到我们要找的项目。这次，我们只剩下两个项目要检查，即第四个和第五个元素。当我们向左移动时，我们将检查第四个元素，我们看到值与我们要找的**7**匹配。如果第四个索引值不是**7**，函数将返回 false，因为没有更多的元素可以检查。如果我们看一下前面图片中的箭头标记，我们可以看到在四步内，我们已经找到了我们要找的值，而在线性搜索函数中，我们需要花 17 步来检查所有 17 个数字，这是最坏情况下的二分搜索，或半间隔搜索，或对数搜索。

正如我们在上一张图片中看到的，我们必须将初始列表分成两半，并继续直到达到一个不能再进一步分割以找到我们的项目的地步。我们可以使用迭代方式或递归方式来执行分割部分。我们将实际上使用两种方式。因此，让我们首先定义迭代方式中的二分搜索的伪代码：

```php
BinarySearch(A : list of sorted items, value) { 

       low = 0 

       high = N

       while (low <= high) { 

    // lowest int value, no fraction 

           mid = (low + high) / 2              

           if (A[mid] > value) 

               high = mid - 1 

           else if (A[mid] < value) 

               low = mid + 1 

           else  

             return true 

       }

       return false 

 }

```

如果我们看一下伪代码，我们可以看到我们根据中间值调整了低和高。如果我们要查找的值大于中间值，我们将调整下界为`mid+1`。如果小于中间值，则将上界设置为`mid-1`。直到下界变大于上界或找到项目为止。如果未找到项目，我们在函数末尾返回 false。现在，让我们使用 PHP 实现伪代码：

```php
function binarySearch(array $numbers, int $needle): bool { 

    $low = 0; 

    $high = count($numbers) - 1; 

    while ($low <= $high) { 

      $mid = (int) (($low + $high) / 2); 

      if ($numbers[$mid] > $needle) { 

          $high = $mid - 1;

      } else if ($numbers[$mid] < $needle) { 

          $low = $mid + 1;

      } else {

          return TRUE;

      }

    }

    return FALSE; 

}

```

在我们的实现中，我们遵循了前一页中的大部分伪代码。现在，让我们运行两次搜索的代码，我们知道一个值在列表中，一个值不在列表中：

```php
$numbers = range(1, 200, 5); 

$number = 31;

if (binarySearch($numbers, $number) !== FALSE) { 

    echo "$number Found \n"; 

} else { 

    echo "$number Not found \n"; 

} 

$number = 500; 

if (binarySearch($numbers, $number) !== FALSE) { 

    echo "$number Found \n"; 

} else { 

    echo "$number Not found \n"; 

} 

```

根据我们之前的线性搜索代码，`31`在列表中，应该显示`Found`。然而，`500`不在列表中，应该显示`Not found`。如果我们运行代码，这是我们在控制台中看到的输出：

```php
31 Found

500 Not found

```

我们现在将为二分搜索编写递归算法，这对我们也很方便。伪代码将要求我们在每次调用函数时发送额外的参数。我们需要在每次递归调用时发送低和高，这是迭代调用中没有做的：

```php
BinarySearch(A : list of sorted items, value, low, high) { 

   if (high < low) 

          return false 

      // lowest int value, no fraction 

           mid = (low + high) / 2   

           if (A[mid] > value) 

               return BinarySearch(A, value, low, mid - 1) 

           else if (A[mid] < value) 

               return BinarySearch(A, value, mid + 1, high)  

     else 

      return TRUE;      

}

```

从前面的伪代码中我们可以看到，现在我们有低和高作为参数，在每次调用中，新值作为参数发送。我们有边界条件，检查低是否大于高。与迭代的代码相比，代码看起来更小更干净。现在，让我们使用 PHP 7 来实现这个：

```php
function binarySearch(array $numbers, int $needle,  

int $low, int $high): bool { 

    if ($high < $low) { 

    return FALSE; 

    } 

    $mid = (int) (($low + $high) / 2); 

    if ($numbers[$mid] > $needle) { 

      return binarySearch($numbers, $needle, $low, $mid - 1); 

    } else if ($numbers[$mid] < $needle) { 

      return binarySearch($numbers, $needle, $mid + 1, $high); 

    } else { 

      return TRUE; 

    } 

}

```

现在，让我们使用以下代码来递归运行这个搜索：

```php
$numbers = range(1, 200, 5); 

$number = 31; 

if (binarySearch($numbers, $number, 0, count($numbers) - 1) !== FALSE) { 

    echo "$number Found \n"; 

} else { 

    echo "$number Not found \n"; 

} 

$number = 500; 

if (binarySearch($numbers, $number, 0, count($numbers) - 1) !== FALSE) { 

    echo "$number Found \n"; 

} else { 

    echo "$number Not found \n"; 

}

```

正如我们从前面的代码中看到的，我们在递归二分搜索的每次调用中发送`0`和`count($numbers)-1`。然后，这个高和低在每次递归调用时根据中间值自动调整。因此，我们已经看到了二分搜索的迭代和递归实现。根据我们的需求，我们可以在程序中使用其中一个。现在，让我们分析二分搜索算法，并找出它为什么比我们的线性或顺序搜索算法更好。

# 二分搜索算法的分析

到目前为止，我们已经看到，对于每次迭代，我们都将列表分成一半，并丢弃一半进行搜索。这使得我们的列表在 1、2 和 3 次迭代后看起来像*n/2*、*n/4*、*n/8*，依此类推。因此，我们可以说，在第 K 次迭代后，将剩下*n/2^k*个项目。我们可以轻松地说，最后一次迭代发生在*n/2^k = 1*时，或者我们可以说，*2^K = n*。因此，从两边取对数得到，*k = log(n)*，这是二分搜索算法的最坏情况运行时间。以下是二分搜索算法的复杂性：

| 最佳时间复杂度 | `O(1)` |
| --- | --- |
| 最坏时间复杂度 | `O(log n)` |
| 平均时间复杂度 | `O(log n)` |
| 空间复杂度（最坏情况） | `O(1)` |

如果我们的数组或列表已经排序，总是更倾向于应用二分搜索以获得更好的性能。现在，无论列表是按升序还是降序排序，都会对我们计算的低和高产生一些影响。到目前为止，我们看到的逻辑是针对升序的。如果数组按降序排序，逻辑将被交换，大于将变成小于，反之亦然。这里需要注意的一点是，二分搜索算法为我们提供了搜索项的索引。然而，可能有一些情况，我们不仅需要知道数字是否存在，还需要找到列表中的第一次出现或最后一次出现。如果我们使用二分搜索算法，它将返回 true 或最大索引号，搜索算法找到数字的地方。然而，这可能不是第一次出现或最后一次出现。为此，我们将稍微修改二分搜索算法，称之为重复二叉搜索树算法。

# 重复二叉搜索树算法

考虑以下图片。我们有一个包含重复项的数组。如果我们尝试从数组中找到**2**的第一次出现，上一节的二分搜索算法将给我们第五个元素。然而，从下面的图片中，我们可以清楚地看到它不是第五个元素；相反，它是第二个元素，这才是正确的答案。因此，我们需要对我们的二分搜索算法进行修改。修改将是重复搜索，直到我们找到第一次出现：

![](img/Image00064.gif)

这是使用迭代方法的修改后的解决方案：

```php
function repetitiveBinarySearch(array $numbers, int $needle): int { 

    $low = 0;

    $high = count($numbers) - 1;

    $firstOccurrence = -1;

    while ($low <= $high) { 

      $mid = (int) (($low + $high) / 2); 

      if ($numbers[$mid] === $needle) { 

          $firstOccurrence = $mid; 

          $high = $mid - 1; 

      } else if ($numbers[$mid] > $needle) { 

          $high = $mid - 1;

      } else {

          $low = $mid + 1;

      } 

    } 

    return $firstOccurrence; 

} 

```

正如我们所看到的，首先我们要检查中间值是否是我们要找的值。如果是真的，那么我们将中间索引分配为第一次出现，并且我们将搜索中间元素的左侧以检查我们要找的数字的任何出现。然后我们继续迭代，直到我们搜索了每个索引（`$low`大于`$high`）。如果没有找到进一步的出现，那么第一次出现的变量将具有我们找到该项的第一个索引的值。如果没有，我们像往常一样返回`-1`。让我们运行以下代码来检查我们的结果是否正确：

```php
$numbers = [1, 2, 2, 2, 2, 2, 2, 2, 2, 3, 3, 3, 3, 3, 4, 4, 5, 5]; 

$number = 2; 

$pos = repetitiveBinarySearch($numbers, $number); 

if ($pos >= 0) { 

    echo "$number Found at position $pos \n"; 

} else { 

    echo "$number Not found \n"; 

} 

$number = 5; 

$pos = repetitiveBinarySearch($numbers, $number); 

if ($pos >= 0) { 

    echo "$number Found at position $pos \n"; 

} else { 

    echo "$number Not found \n"; 

}

```

现在，我们有一个包含重复值的数组，值为 2、3、4 和 5。我们想搜索数组，并找到值第一次出现的位置或索引。例如，如果我们在一个常规的二分搜索函数中搜索 `2`，它会返回第八个位置，即它找到值 `2` 的位置。在我们的情况下，我们实际上是在寻找第二个索引，它实际上保存了项目 `2` 的第一次出现。我们的函数 `repetitiveBinarySearch` 正是这样做的，我们将返回的位置存储到一个名为 `$pos` 的变量中。如果找到数字，我们将显示输出以及位置。现在，如果我们在控制台中运行前面的代码，我们将得到以下输出：

```php
2 Found at position 1

5 Found at position 16

```

这符合我们的预期结果。因此，我们现在有了一个重复的二分搜索算法，用于查找给定排序列表中项目的第一次和最后一次出现。这可能是一个非常方便的函数来解决许多问题。

到目前为止，从我们的例子和分析来看，我们可以得出结论，二分搜索肯定比线性搜索更快。然而，主要的前提是在应用二分搜索之前对列表进行排序。在未排序的数组中应用二分搜索会导致我们得到不准确的结果。有时候我们会收到一个数组，而我们不确定这个数组是否已经排序。现在，问题是，“在这种情况下，我们应该先对数组进行排序然后应用二分搜索算法吗？还是应该只运行线性搜索算法来找到一个项目？”让我们讨论一下这个问题，这样我们就知道如何处理这种情况。

# 搜索一个未排序的数组 - 我们应该先排序吗？

所以现在，我们处于这样一种情况：我们有一个包含 *n* 个项目的数组，它们没有排序。由于我们知道二分搜索更快，我们决定先对其进行排序，然后使用二分搜索来搜索项目。如果我们这样做，我们必须记住，最好的排序算法的最坏时间复杂度为 `O(nlog n)`，而对于二分搜索，最坏情况的复杂度为 `O(log n)`。因此，如果我们先排序然后应用二分搜索，复杂度将为 `O(n log n)`，因为这是与 `O(log n)` 相比最大的。然而，我们也知道，对于任何线性或顺序搜索（无论是排序还是未排序），最坏的时间复杂度都是 `O(n)`，这比 `O(n log n)` 要好得多。根据 `O(n)` 和 `O(n log n)` 的复杂度比较，我们可以清楚地说，如果数组没有排序，执行线性搜索是一个更好的选择。

让我们考虑另一种情况，我们需要多次搜索一个给定的数组。让我们用 *k* 表示我们想要搜索数组的次数。如果 *k* 为 1，那么我们可以轻松地应用上一段讨论的线性方法。如果 *k* 的值相对于数组的大小 *n* 来说比较小，那么也没问题。然而，如果 *k* 的值接近或大于 *n*，那么我们在这里应用线性方法就会有一些问题。

假设 *k = n*，那么对于 *n* 次搜索，线性搜索的复杂度将为 `O(n²)`。现在，如果我们选择排序然后搜索，即使 *k* 更大，一次排序也只需要 `O(n log n)` 的时间复杂度。然后，每次搜索只需要 `O(log n)`，而 *n* 次搜索的最坏情况复杂度为 `O(n log n)`。如果我们在这里考虑最坏的情况，那么对于排序和搜索 *k* 个项目，我们将得到 `O(n log n)`，这比顺序搜索要好。

因此，我们可以得出结论：如果搜索操作的次数较小，与数组的大小相比，最好不要对数组进行排序，而是执行顺序搜索。然而，如果搜索操作的次数较大，与数组的大小相比，最好先对数组进行排序，然后应用二分搜索。

多年来，二分搜索算法不断发展，并出现了不同的变体。我们可以通过计算决策来选择下一个应该使用的索引，而不是每次选择中间索引。这就是这些变体能够高效工作的原因。现在我们将讨论二分搜索算法的两种变体：插值搜索和指数搜索。

# 插值搜索

在二分搜索算法中，我们总是从数组的中间开始搜索过程。如果数组是均匀分布的，并且我们正在寻找一个可能接近数组末尾的项目，那么从中间开始搜索可能对我们来说并不是一个好选择。在这种情况下，插值搜索可能非常有帮助。插值搜索是对二分搜索算法的改进。插值搜索可能根据搜索关键字的值而转到不同的位置。例如，如果我们正在搜索一个接近数组开头的关键字，它将转到数组的第一部分，而不是从中间开始。位置是使用探测位置计算器方程计算的，如下所示：

```php
pos = low + [ (key-arr[low])*(high-low) / (arr[high]-arr[low]) ]

```

正如我们所看到的，我们从通用的 `mid = (low+high)/2` 方程转变为一个更复杂的方程。如果搜索的关键字更接近 `arr[high]`，这个公式将返回一个更高的值，如果关键字更接近 `arr[low]`，则返回一个更低的值。现在，让我们借助我们的二分搜索代码来实现这种搜索方法：

```php
function interpolationSearch(array $arr, int $key): int { 

    $low = 0; 

    $high = count($arr) - 1; 

    while ($arr[$high] != $arr[$low] && $key >= $arr[$low] && 

      $key <= $arr[$high]) { 

    $mid = intval($low + (($key - $arr[$low]) * ($high - $low) 

    / ($arr[$high] - $arr[$low]))); 

      if ($arr[$mid] < $key) 

          $low = $mid + 1; 

      else if ($key < $arr[$mid]) 

          $high = $mid - 1; 

      else 

          return $mid; 

    } 

    if ($key == $arr[$low]) 

      return $low; 

    else

      return -1; 

}

```

在这里，我们以一种不同的方式进行计算。尽管它需要更多的计算步骤，但好处是，如果列表是均匀分布的，那么该算法的平均复杂度为 `O(log (log n))`，这与二分搜索的复杂度 `O(log n)` 相比要好得多。此外，我们必须小心，如果关键字的分布不均匀，插值搜索的性能可能会下降。

现在，我们将探讨另一种二分搜索的变体，称为指数搜索，它可以改进算法。

# 指数搜索

在二分搜索中，我们为给定的关键字搜索整个列表。指数搜索通过决定搜索的下限和上限来改进二分搜索，以便我们不会最终搜索整个列表。它改进了我们需要找到一个元素所需的比较次数。搜索分为以下两个步骤：

1.  我们通过寻找第一个指数 *k*，其中 *2^k* 的值大于搜索项，来确定边界大小。现在，*2^k* 和 *2^(k-1)* 分别成为上限和下限。

1.  对 *2^k* 和 *2^(k-1)* 进行二分搜索算法。

现在让我们使用我们的递归 `binarySearch` 函数来实现指数搜索：

```php
function exponentialSearch(array $arr, int $key): int { 

    $size = count($arr); 

    if ($size == 0) 

      return -1; 

    $bound = 1; 

    while ($bound < $size && $arr[$bound] < $key) { 

      $bound *= 2; 

    } 

    return binarySearch($arr, $key, intval($bound / 2),  

min($bound, $size)); 

}

```

在第一步中，我们需要 *i* 步来确定边界。因此，该算法的复杂度为 `O(log i)`。我们必须记住，这里的 *i* 要远小于 *n*。然后，我们使用 *2^j* 到 *2^(j-1)* 进行二分搜索，其中 *j = log i*。我们知道二分搜索的复杂度为 `O(log n)`，其中 *n* 是列表的大小。然而，由于我们正在进行较小范围的搜索，实际上我们搜索的是 *2 ^(log i) \ - 2 ^(log i) - 1 = 2 ^(log i - 1)* 大小。因此，这个边界的复杂度将是 log *(2 ^(log i - 1) ) = log (i) - 1 = O(log i)*。

因此，指数搜索的复杂性如下：

| 最佳时间复杂度 | `O(1)` |
| --- | --- |
| 最坏时间复杂度 | `O(log i)` |
| 平均时间复杂度 | `O(log i)` |
| 空间复杂度（最坏情况） | `O(1)` |

# 使用哈希表进行搜索

哈希表在搜索操作时可以是非常高效的数据结构。由于哈希表以关联方式存储数据，如果我们知道在哪里查找数据，我们可以很容易地快速获取数据。在哈希表中，每个数据都有一个与之关联的唯一索引。如果我们知道要查看哪个索引，我们可以很容易地找到键。通常，在其他编程语言中，我们必须使用单独的哈希函数来计算哈希索引以存储值。哈希函数旨在为相同的键生成相同的索引，并避免冲突。然而，PHP 的一个伟大特性是 PHP 数组本身就是一个哈希表，在其底层 C 实现中。由于数组是动态的，我们不必担心数组的大小或溢出数组的值。我们需要将值存储在关联数组中，以便我们可以将值与键关联起来。如果是字符串或整数，键可以是值本身。让我们运行一个例子来理解使用哈希表进行搜索：

```php
$arr = [];

$count = rand(10, 30); 

for($i = 0; $i<$count;$i++) {     

    $val = rand(1,500);     

    $arr[$val] = $val;     

} 

$number = 100; 

if(isset($arr[$number])) { 

    echo "$number found "; 

} else { 

    echo "$number not found"; 

}

```

我们刚刚构建了一个简单的随机关联数组，其中值和键是相同的。由于我们使用的是 PHP 数组，尽管值可以在 1 到 500 的范围内，实际数组大小可以是 10 到 30 之间的任何值。如果是在其他语言中，我们将构建一个大小为 501 的数组来容纳这个值作为键。这就是为什么要使用哈希函数来计算索引。如果需要的话，我们也可以使用 PHP 的内置哈希函数：

```php
string hash(string $algo ,string $data [,bool $raw_output = false ])

```

第一个参数采用我们想要用于哈希的算法类型。我们可以选择 md5、sha1、sha256、crc32 等。每个算法都会产生一个固定长度的哈希输出，我们可以将其用作哈希表的键。

如果我们看一下我们的搜索部分，我们可以看到我们实际上是直接检查相关的索引。这使得我们的搜索复杂度为`O(1)`。在 PHP 中，使用哈希表进行快速搜索可能是有益的，即使不使用哈希函数。但是，如果需要的话，我们总是可以使用哈希函数。

到目前为止，我们已经涵盖了基于数组和线性结构的搜索。现在我们将把重点转移到层次化数据结构搜索，比如搜索树和图。虽然我们还没有讨论图（我们将在下一章讨论），但我们将把重点放在树搜索上，这也可以应用于图搜索。

# 树搜索

搜索层次化数据的最佳方法之一是创建搜索树。在第六章中，*理解和实现树*，我们看到了如何构建二叉搜索树并提高搜索效率。我们还发现了遍历树的不同方法。现在，我们将探索两种最流行的搜索树结构的方式，通常称为广度优先搜索（BFS）和深度优先搜索（DFS）。

# 广度优先搜索

在树结构中，根节点连接到其子节点，每个子节点都可以表示为一棵树。我们在第六章中已经看到了这一点，*理解和实现树*。在广度优先搜索中，通常称为 BFS，我们从一个节点（通常是根节点）开始，首先访问所有相邻或邻居节点，然后再访问其他邻居节点。换句话说，我们必须逐层移动，而我们应用 BFS。由于我们逐层搜索，这种技术被称为广度优先搜索。在下面的树结构中，我们可以使用 BFS：

![](img/Image00065.jpg)

对于这棵树，BFS 将按照以下节点进行：![](img/Image00066.jpg)

BFS 的伪代码如下：

```php
procedure BFS(Node root)  

  Q := empty queue 

  Q.enqueue(root); 

  while(Q != empty) 

     u := Q.dequeue() 

     for each node w that is childnode of u 

        Q.enqueue(w) 

     end for each 

  end while 

end procedure

```

我们可以看到我们保留了一个队列来跟踪我们需要访问的节点。我们可以保留另一个队列来保存访问的顺序，并将其返回以显示访问顺序。现在，我们将使用 PHP 7 来实现 BFS。

# 实现广度优先搜索

到目前为止，我们还没有详细介绍图，因此我们将严格将 BFS 和 DFS 的实现保留在树结构中。此外，我们将使用我们在第六章中看到的通用树结构，*理解和实现树*，（甚至不是二叉树）。我们将使用相同的`TreeNode`类来定义我们的节点和与子节点的关系。因此，现在让我们定义具有 BFS 功能的`Tree`类：

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

class Tree { 

    public $root = NULL; 

    public function __construct(TreeNode $node) { 

      $this->root = $node; 

    } 

    public function BFS(TreeNode $node): SplQueue { 

      $queue = new SplQueue; 

      $visited = new SplQueue; 

      $queue->enqueue($node); 

      while (!$queue->isEmpty()) { 

          $current = $queue->dequeue(); 

          $visited->enqueue($current); 

          foreach ($current->children as $child) { 

            $queue->enqueue($child); 

          } 

      } 

    return $visited; 

    }

}

```

我们在树类内部实现了 BFS 方法。我们以根节点作为广度优先搜索的起点。在这里，我们有两个队列：一个用于保存我们需要访问的节点，另一个用于我们已经访问的节点。我们还在方法的最后返回了访问的队列。现在让我们模仿一下我们在本节开头看到的树。我们想要像图中显示的树一样放置数据，并检查 BFS 是否实际返回我们期望的模式：![](img/Image00066.jpg)：

```php
    $root = new TreeNode("8"); 

    $tree = new Tree($root); 

    $node1 = new TreeNode("3"); 

    $node2 = new TreeNode("10"); 

    $root->addChildren($node1); 

    $root->addChildren($node2); 

    $node3 = new TreeNode("1"); 

    $node4 = new TreeNode("6"); 

    $node5 = new TreeNode("14"); 

    $node1->addChildren($node3); 

    $node1->addChildren($node4); 

    $node2->addChildren($node5); 

    $node6 = new TreeNode("4"); 

    $node7 = new TreeNode("7"); 

    $node8 = new TreeNode("13"); 

    $node4->addChildren($node6); 

    $node4->addChildren($node7); 

    $node5->addChildren($node8); 

    $visited = $tree->BFS($tree->root); 

    while (!$visited->isEmpty()) { 

      echo $visited->dequeue()->data . "\n"; 

    } 

```

我们在这里通过创建节点并将它们附加到根和其他节点来构建整个树结构。一旦树完成，我们就调用`BFS`方法来找到遍历的完整序列。最后的`while`循环打印了我们访问的节点序列。以下是前面代码的输出：

```php
8

3

10

1

6

14

4

7

13

```

我们已经收到了我们期望的结果。现在，如果我们想搜索以查找节点是否存在，我们可以为我们的`$current`节点值添加一个简单的条件检查。如果匹配，那么我们可以返回访问的队列。

BFS 的最坏复杂度为**O**(*|V| + |E*),其中*V*是顶点或节点的数量，*E*是节点之间的边或连接的数量。对于空间复杂度，最坏情况是**O**(*|V|*)。

图的 BFS 类似，但有一点不同。由于图可能是循环的（可以创建循环），我们需要确保我们不会一遍又一遍地访问相同的节点以创建无限循环。为了避免重新访问图节点，我们必须跟踪我们已经访问的节点。为了标记已访问的节点，我们可以使用队列，或使用图着色算法。我们将在下一章中探讨这一点。

# 深度优先搜索

深度优先搜索，或 DFS，是一种搜索技术，我们从一个节点开始搜索，并尽可能深入到目标节点通过分支。DFS 不同于 BFS，我们尝试更深入地挖掘而不是首先扩散。DFS 垂直增长，并在到达分支的末端时回溯，并移动到下一个可用的相邻节点，直到搜索结束。我们可以从上一节中取相同的树图像，如下所示：

![](img/Image00067.jpg)

如果我们在这里应用 DFS，遍历将是![](img/Image00068.jpg)。我们从根开始，然后访问第一个子节点，即**3**。然而，与 BFS 不同，我们将探索**3**的子节点，并重复此过程，直到达到分支的底部。在 BFS 中，我们采用了迭代方法。对于 DFS，我们将采用递归方法。现在让我们为 DFS 编写伪代码：

```php
procedure DFS(Node current)       

     for each node v that is childnode of current  

        DFS(v) 

     end for each 

end procedure 

```

# 实现深度优先搜索

DFS 的伪代码看起来很简单。为了跟踪节点访问的顺序，我们需要使用一个队列，它将跟踪我们`Tree`类内部的节点。以下是我们带有递归 DFS 的`Tree`类的实现：

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

class Tree { 

    public $root = NULL; 

    public $visited; 

    public function __construct(TreeNode $node) { 

      $this->root = $node; 

      $this->visited = new SplQueue; 

    } 

    public function DFS(TreeNode $node) { 

      $this->visited->enqueue($node); 

      if($node->children){ 

          foreach ($node->children as $child) { 

        $this->DFS($child); 

          } 

      } 

    }

}

```

正如我们所看到的，我们在树类中添加了一个额外的属性`$visited`来跟踪访问的节点。当我们调用`DFS`方法时，我们将节点添加到队列中。现在，如果我们使用上一节中的相同树结构，只需添加 DFS 调用并获取访问部分，它将如下所示：

```php
try { 

    $root = new TreeNode("8"); 

    $tree = new Tree($root); 

    $node1 = new TreeNode("3"); 

    $node2 = new TreeNode("10"); 

    $root->addChildren($node1); 

    $root->addChildren($node2); 

    $node3 = new TreeNode("1"); 

    $node4 = new TreeNode("6"); 

    $node5 = new TreeNode("14"); 

    $node1->addChildren($node3); 

    $node1->addChildren($node4); 

    $node2->addChildren($node5); 

    $node6 = new TreeNode("4"); 

    $node7 = new TreeNode("7"); 

    $node8 = new TreeNode("13"); 

    $node4->addChildren($node6); 

    $node4->addChildren($node7); 

    $node5->addChildren($node8); 

    $tree->DFS($tree->root); 

    $visited = $tree->visited; 

    while (!$visited->isEmpty()) { 

      echo $visited->dequeue()->data . "\n"; 

    } 

} catch (Exception $e) { 

    echo $e->getMessage(); 

}

```

由于 DFS 不返回任何内容，我们使用类属性`visited`来获取队列，以便我们可以显示访问节点的序列。如果我们在控制台中运行此程序，将会得到以下输出：

```php
8

3

1

6

4

7

10

14

13

```

结果符合预期。如果我们需要 DFS 的迭代解决方案，我们必须记住，我们需要使用堆栈而不是队列来跟踪下一个要访问的节点。然而，由于堆栈遵循 LIFO 原则，对于我们提到的图像，输出将与我们最初的想法不同。以下是使用迭代方法的实现：

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

class Tree { 

    public $root = NULL; 

    public function __construct(TreeNode $node) { 

      $this->root = $node; 

    }

    public function DFS(TreeNode $node): SplQueue { 

      $stack = new SplStack;

      $visited = new SplQueue;

      $stack->push($node);

      while (!$stack->isEmpty()) { 

          $current = $stack->pop(); 

          $visited->enqueue($current); 

          foreach ($current->children as $child) { 

            $stack->push($child); 

          } 

      } 

      return $visited; 

    }

}

try {

    $root = new TreeNode("8"); 

    $tree = new Tree($root); 

    $node1 = new TreeNode("3"); 

    $node2 = new TreeNode("10"); 

    $root->addChildren($node1); 

    $root->addChildren($node2); 

    $node3 = new TreeNode("1"); 

    $node4 = new TreeNode("6"); 

    $node5 = new TreeNode("14"); 

    $node1->addChildren($node3); 

    $node1->addChildren($node4); 

    $node2->addChildren($node5); 

    $node6 = new TreeNode("4"); 

    $node7 = new TreeNode("7"); 

    $node8 = new TreeNode("13"); 

    $node4->addChildren($node6); 

    $node4->addChildren($node7); 

    $node5->addChildren($node8); 

    $visited = $tree->DFS($tree->root); 

    while (!$visited->isEmpty()) { 

      echo $visited->dequeue()->data . "\n"; 

    } 

} catch (Exception $e) { 

    echo $e->getMessage(); 

}

```

它看起来与我们的迭代 BFS 算法非常相似。主要区别在于使用堆栈数据结构而不是队列数据结构来存储已访问的节点。这也会对输出产生影响。前面的代码将产生输出`8 → 10 → 14 → 13 → 3 → 6 → 7 → 4 → 1`。这与上一节中显示的先前输出不同。由于我们使用堆栈，输出实际上是正确的。我们使用堆栈来推入特定节点的子节点。对于我们的根节点，其值为**8**，我们有值为**3**的第一个子节点。它被推入堆栈，然后，根的第二个子节点的值为**10**，也被推入堆栈。由于值**10**是最后被推入的，它将首先出现，遵循堆栈的 LIFO 原则。因此，如果我们使用堆栈，顺序始终将从最后的分支开始到第一个分支。然而，如果我们想要保持节点的顺序从左到右，那么我们需要在 DFS 代码中进行一些小的调整。以下是带有更改的代码块：

```php
public function DFS(TreeNode $node): SplQueue { 

  $stack = new SplStack; 

  $visited = new SplQueue;

  $stack->push($node); 

  while (!$stack->isEmpty()) { 

      $current = $stack->pop(); 

      $visited->enqueue($current); 

      $current->children = array_reverse($current->children); 

      foreach ($current->children as $child) { 

        $stack->push($child); 

      } 

    } 

    return $visited;

}

```

与上一个代码块的唯一区别是，在访问特定节点的子节点之前，我们添加了以下行：

```php
$current->children = array_reverse($current->children);

```

由于堆栈执行后进先出（LIFO）的操作，通过反转，我们确保首先访问第一个节点，因为我们已经反转了顺序。实际上，它将简单地作为队列工作。这将产生 DFS 部分所示的期望顺序。如果我们有一棵二叉树，那么我们可以很容易地做到这一点，而不需要任何反转，因为我们可以选择先推入右子节点，然后再推入左子节点以先弹出左子节点。

DFS 的最坏复杂度为**O**（*|V| + |E|*），其中*V*是顶点或节点的数量，*E*是节点之间的边或连接的数量。对于空间复杂度，最坏情况是**O**（*|V|*），这与 BFS 类似。

# 摘要

在本章中，我们讨论了不同的搜索算法及其复杂性。您学会了如何通过哈希表来改进搜索，以获得恒定的时间结果。我们还探讨了 BFS 和 DFS，这两种是层次数据搜索中最重要的方法之一。我们将使用类似的概念来探索下一章中即将探讨的图数据结构。图算法对于解决许多问题至关重要，并且在编程世界中被广泛使用。让我们继续探讨另一个有趣的主题 - 图。
