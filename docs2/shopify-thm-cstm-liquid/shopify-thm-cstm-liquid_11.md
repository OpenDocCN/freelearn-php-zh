# 第八章：*第八章*：探索 Shopify Ajax API

在前面的章节中，我们学习了 Shopify 和 Liquid 的基础知识，这为我们未来的开发提供了坚实的基础。在为我们的未来学习打下适当的基础后，我们学习了 Liquid 核心的工作原理。通过了解对象、标签和过滤器，我们学会了如何使用相对简单且不显眼的特性集创建复杂的功能。最后，我们学习了如何使用各种输入类型设置以及`sections`和`blocks`属性，在商店中创建易于配置的元素。

在这些章节中，我们学习了如何使用静态内容创建元素。*然而，如果我们想动态更新页面内容呢？*这正是 Shopify Ajax API 发挥作用的地方。在本章的最后，我们将探讨 Shopify Ajax API，解释其要求和限制，以及可能的用例。

在本章中，我们将涵盖以下主题：

+   Shopify Ajax API 简介

+   使用 POST 请求更新购物车会话

+   使用 GET 请求获取数据

完成本章后，我们将了解 Shopify Ajax API 是什么，以及我们可以发起的类型请求，例如获取产品信息、将产品添加到购物车，甚至读取购物车当前的内容。此外，我们将通过改进一些我们的先前项目来学习 Shopify Ajax API 的典型用例。

最后，我们将学习如何根据 Shopify 的算法自动生成推荐产品列表，并将其转换为通常由店主要求的预测搜索功能。

# 技术要求

尽管我们将解释每个主题并配合相应的图形展示，但由于**Shopify**是一个托管服务，我们需要网络连接来跟随本章中概述的步骤。

本章的代码可在 GitHub 上找到：[`github.com/PacktPublishing/Shopify-Theme-Customization-with-Liquid/tree/main/Chapter08`](https://github.com/PacktPublishing/Shopify-Theme-Customization-with-Liquid/tree/main/Chapter08)。

尽管本章将包含每个主题的多个真实示例和用例，但我们仍需要具备基本的 Ajax 理解和知识，以便彻底理解本章内容。

注意，我们只会展示与 Shopify API 相关的示例和项目，而不是通用的 Ajax。有关 Ajax 的详细信息，我们可以参考[`www.w3schools.com/js/js_ajax_intro.asp`](https://www.w3schools.com/js/js_ajax_intro.asp)，它提供了 Ajax 的出色介绍。

本章的“代码实战”视频可在此找到：[`bit.ly/2VUZ7Qp`](https://bit.ly/2VUZ7Qp)

# Shopify Ajax API 简介

**Ajax**，或**异步 JavaScript 和 XML**，是一种我们可以用来与服务器交换少量数据并更新任何页面的部分内容的方法，而无需重新加载整个页面。*那么，Shopify Ajax API 究竟是什么呢？*

Shopify Ajax API 是一个 REST API 端点，通过该端点我们可以发送请求来读取或更新某些信息。例如，我们可以使用**GET**请求来读取产品或甚至当前的购物车数据，或者我们可以使用**POST**请求来更新当前的内容会话。

Shopify Ajax 是一个未认证的 API，这意味着它不需要任何令牌或 API 密钥即可访问商店信息。Shopify 还为我们提供了一个名为**Shopify Admin API**的认证 API，应用程序和服务使用该 API 与 Shopify 服务器通信。

通过 Shopify API，我们可以访问我们大部分的商店数据，其响应将返回 JSON 格式的数据，尽管我们无法读取客户和订单数据或更新任何商店数据——我们只能使用 Shopify Admin API 来完成这些操作。

注意，Shopify 对 Ajax API 有一定的速率限制，以防止滥用（向 Shopify 服务器发送无限数量的请求）。其中一项限制是最大输入数组大小限制，目前限制为 250。假设我们正在查找有关超过 1,000 个产品的集合中所有产品的信息。由于我们每个查询最多只能限制为 250 个产品，因此我们将不得不使用多个查询来实现这一点。

小贴士：

为了保持简洁并切中要点，我们在此不会提及所有速率限制。有关 Ajax API 速率限制的更多信息，请参阅[`shopify.dev/api/usage/rate-limits`](https://shopify.dev/api/usage/rate-limits)。

现在我们已经熟悉了需要了解的 Shopify Ajax API 基础知识，我们可以从实际的角度了解更多关于 Ajax API 的内容。

# 使用 POST 请求更新购物车会话

此前，我们提到可以使用 POST 请求来更新当前的购物车会话。根据我们想要执行的操作类型，我们可以将 POST 请求与以下购物车端点配对：

+   `/cart/add.js`

+   `/cart/update.js`

+   `/cart/change.js`

+   `/cart/clear.js`

虽然这听起来可能微不足道，但它是当今电子商务商店的一个基本方面，我们期望在不刷新整个页面的情况下执行操作。

## `/cart/add.js`端点

如其名称所示，`/cart/add.js`端点允许我们添加一个或多个产品变体到购物车，而无需刷新购物车。要执行此操作，我们需要创建一个名为`items`的数组，其中包含一个对象，该对象包含以下两个键：

+   `id`键的值应包含我们要添加到购物车的变体 ID 的数字类型值。

+   `quantity`键的值应包含我们要添加到购物车的数量的数字类型值。

如果我们需要包含多个变体，我们可以在`items`数组内部简单地追加多个对象：

```php
items: [
  {
    id: 40065085407386
  },
  {
    id: 40065085603994,
    quantity: 5
  }
]
```

在前面的例子中，我们可以看到一个包含两个对象的数组，这些对象包含一组不同的变体`id`和`quantity`键。然而，请注意，第一个对象不包含`quantity`键。这是因为`quantity`键完全是可选的，如果我们没有包含它，它假定`quantity`的值等于`1`。

让我们看看我们如何在现实生活中的例子中使用它：

1.  如您所回忆的，在*第四章*，*使用对象深入液态核心*，以及后来在*第五章*，*使用过滤器深入液态核心*中，我们通过向集合模板添加额外的集合来开展了一个`自定义集合`项目。然而，当前的功能是，如果我们点击通过*第四章*，*使用对象深入液态核心*，和*第五章*，*使用过滤器深入液态核心*开发的`自定义集合`表单，我们可以在`Snippet`目录下的`collection-form.liquid`中找到`自定义集合`表单：

    ```php
    {% if product.compare_at_price != blank %}
    <div class="custom-collection--item">
      <a href="{{ product.url }}">
        <img src="img/{{ product | img_url: "300x300" }}"/>
        <p class="h4 custom-collection--title">{{ 
            product.title }}</p>
        <p class="custom-collection--price">
          {{ product.price | money }}
          {% assign discount-price =            product.compare_at_price |                   minus: product.price %}
          <span>Save {{ discount-price | money }}</span>
        </p>
        <span class="custom-collection--sale-badge">{{       product.compare_at_price | minus: product.price |         times: 100 | divided_by: product.compare_at_price     }}%</span>
      </a>
      {% form "product", product %}
    <input type="hidden" name="id" value="{{ 
            product.first_available_variant.id }}" />
        <input type="submit" value="Add to Cart"/>
      {% endform %}
    </div>
    {% endif %}
    ```

1.  如我们所见，我们已创建的集合表单已经包含了我们需要的两个必要元素：提交按钮和存储在隐藏输入中的变体`id`。为了更直接的导航，让我们首先将一个名为`collection-submit`的新类分配给提交按钮：

    ```php
    {% form "product", product %}
      <input type="hidden" name="id" value="{{     product.first_available_variant.id }}" />
      <input type="submit" class="collection-submit"     value="Add to Cart"/>
    {% endform %}
    ```

1.  在适当的选择器到位后，我们现在可以在提交按钮上使用`addEventListener`来捕获点击事件并将对象传递给我们将要创建的函数：

    ```php
    const addSelector = document.querySelectorAll   (".collection-submit");
    if (addSelector.length) {
      for (let i = 0; i < addSelector.length; i++) {
        addSelector[i].addEventListener('click', function(e) { 
          e.preventDefault();
          addCart(this);
        });
      }
    }
    ```

1.  在前面的例子中，我们创建了一个`addSelector`常量来捕获点击事件。使用`preventDefault()`，我们取消了任何当前事件的流程，并将点击元素的`object`传递给`addCart`函数。现在，让我们看看如何创建`addCart`函数：

    ```php
    const addCart = (el) => {
      let formData = {
        'items': [
          {
            id: el.previousElementSibling.value
          }
        ]
      };
    }
    ```

    我们首先创建了一个带有`el`参数的箭头函数，我们将传递之前点击元素的`object`。在`addCart`函数内部，我们创建了一个局部变量，在其中我们分配了一个数组。这个数组包含一个对象，该对象包含我们想要添加到购物车的变量的`id`属性。

1.  考虑到我们之前将点击的对象传递给了箭头函数，我们使用了`previousElementSibling`来选择正确的输入元素并相应地返回其值。现在我们已经准备好了所有必要的资产，我们只需要使用`fetch`请求将数据`POST`到 Shopify 服务器并更新当前的购物车会话：

    ```php
    const addCart = (el) => {
      let formData = {
        'items': [
          {
            id: el.previousElementSibling.value
          }
        ]
      };
      fetch('/cart/add.js', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(formData)
      })
      .then(success => {
        console.log("Success:", success);
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    }
    ```

有了这个，我们已经成功创建了一个完全功能的 Ajax API POST 请求，允许我们在不重新加载页面的情况下向当前的购物车会话添加任意数量的项目。此外，我们还包含了 `then()` 和 `catch()` 方法，以便在控制台日志中返回 `success` 和 `error` 消息。

我们还学会了如何通过 `/cart/add.js` 端点将特定数量的产品添加到当前的购物车会话中。*然而，如果我们想要将特定产品的某些行项目属性携带到购物车中，那会怎样呢？*

我们可以通过简单地包含一个额外的参数 `properties` 来轻松解决这个问题，它接受一个键值类型的对象：

```php
let formData = {
  'items': [
    {
      id: el.previousElementSibling.value,
      properties: {
        'Engraving message': 'Learning Liquid is fantastic!'
      }
    }
  ]
};
```

我们应该设置键，使其等于行项目输入的名称或行项目属性的第一个部分，其中值应该等于从输入或行项目属性的第二个部分检索到的值。假设我们需要回忆行项目属性是如何工作的。在这种情况下，我们可以回顾 *第四章*，*使用对象深入液态核心*，其中在 *产品定制* 子主题中，位于 *使用全局对象* 部分中，我们解释了行项目属性是如何工作的。

如果我们需要传递一个仅在管理员订单部分可见的隐藏行项目，我们需要在键名后附加一个 *下划线*：

```php
let formData = {
  'items': [
    {
      id: el.previousElementSibling.value,
      properties: {
        '_Engraving message': 'Learning Liquid is fantastic!'
      }
    }
  ]
};
```

如果我们决定使用 `jQuery`，我们可以使代码更加紧凑：

```php
jQuery.post('/cart/add.js', {
  items: [
    {
      quantity: 1,
      id: 40065085407386,
      properties: {
        '_Engraving message': 'Learning Liquid is fantastic!'
      }
    }
  ]
});
```

然而，我们应该检查我们正在工作的主题是否已经包含一个 `jQuery` 库。否则，我们应该避免向主题引入新的库。

通过涵盖 `JavaScript` 和 `jQuery` 解决方案，我们现在可以确信我们将能够使用我们的技能来生成必要的 Ajax API 代码。*但是，如果我们不小心添加了比所需数量多得多的数量，我们需要更新产品的数量怎么办？*

## `/cart/update.js` 端点

正如其名称所暗示的，`/cart/update.js` 端点允许我们更新当前购物车会话中的行项目值。

虽然 `/cart/update.js` 与 `/cart/update.js` 的工作方式类似，但有一些明显的区别。例如，在 `/cart/add.js` 中，当我们处理多个变体时，我们必须创建一个单独的对象，而与 `/cart/update.js` 不同，我们只需要创建一个单一的对象：

```php
updates: {
    40065085407386: 5,
    40065085603994: 3
}
```

注意，在这里我们使用的是一组键值，而不是两组，其中键由变体 ID 表示，`quantity` 值代表键值。此外，我们现在使用的是更新，而不是项目。让我们创建一个函数来帮助我们测试我们的新知识：

```php
const updateCart = (el) => {
  let formData = {
    updates: {
      [el.previousElementSibling.value]: 5
    }
  };
  fetch('/cart/update.js', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(formData)
  })
  .then(success => {
    console.log("Success:", success);
  })
  .catch((error) => {
    console.error('Error:', error);
  });
}
```

正如我们所见，更新购物车会话的通用代码与添加产品购物车相似，这并不令人惊讶。除了更新购物车的内容外，`/cart/update.js` 还允许我们将产品添加到当前的购物车会话中。

通过使用 `/cart/update.js`，我们可以通过使用变体 ID 来识别我们想要更新的变体来轻松更新购物车中每个项目的数量。*但如果我们想要更新的变体不在购物车中呢？* 这就是 `/cart/update.js` 交替函数触发的地方，通过将产品变体添加到购物车中并选择数量来更新当前的购物车会话。

例如，在之前的 `updateCart` 函数中，我们将 `quantity` 值设置为静态值 `5`。无论我们调用前面的函数多少次，购物车中任何变体的总数量永远不会超过 `5`。因此，我们建议始终使用 `/cart/update.js` 来更新现有的购物车项目，并使用 `/cart/add.js` 来添加额外的项目到购物车。

这样，我们就已经学会了如何在当前的购物车会话中更新行项目。然而，正如你可能从 *第四章* 中回忆起来的，“使用对象深入液态核心”，在 *产品定制* 子主题中，位于 *使用全局对象* 部分中，我们了解到可以使用行项目实现不同类型的定制。因此，如果它们的定制不同，这将把相同的产品变体排序到不同的行中。

虽然这些产品可能位于不同的行上，但它们都将具有相同的变体 ID。*那么，如果我们运行* `/cart/update.js` *来更新三个不同行上的特定变体会发生什么？*

`/cart/update.js` 端点将成功执行其操作。然而，由于它不知道我们想要更新哪个行项目，它只会更新匹配变体 ID 的行项目的第一个出现，然后停止。它不会更新具有相同变体 ID 的任何其他出现。*但如果我们想要更新特定的行项目而不是第一个出现呢？*

## `/cart/change.js` 端点

`/cart/change.js` 端点与 `/cart/update.js` 端点类似，因为它允许我们在当前的购物车会话中更新行项目。然而，有两个关键的区别：我们一次只能修改一个行项目，而且（更重要的是）我们可以指定我们想要更改的确切行项目。

与 `/cart/add.js` 端点类似，`/cart/change.js` 端点也使用一个包含两个键值对的对象——一个用于识别行项目，另一个用于分配所需的数量：

```php
{
  'id': 40065085407386,
  'quantity': 7
}
```

在使用 `id` 和变体 ID 来识别行项目时，这不会引起任何错误，但这并不能解决我们的问题，因为我们购物车中可能会有多个具有相同变体 ID 的行项目。为了解决这个问题，我们可以使用 `line` 属性来识别我们想要更改的具体行项目：

```php
{
  'line': 3,
  'quantity': 7
}
```

`line`值基于当前购物车会话中行项的索引位置，其中基本值从`1`开始。例如，如果我们购物车中有四个项目，并且我们正在尝试更新第三个位置的行项，我们可以将`line`值设置为`3`，正如我们之前的例子。请注意，`/cart/change.js`端点最常见的用途是轻松更新购物页面中每个行项的数量。

执行以下步骤以成功实现`/cart/update.js`端点：

1.  正如我们之前提到的，要成功使用`/cart/update.js`端点，我们需要两样东西：`quantity`值，我们可以快速从我们修改的输入值中返回它，以及行项的当前位置。为了确定行项的位置，我们可以使用 JavaScript 的`indexOf()`方法。或者，如果我们想在 Liquid 的`for`循环中设置`quantity`输入的`data`属性并设置其值为`forloop.index`，我们可以采用第二种方法。在这里，我们将使用第二种方法来添加`forloop.index`作为`data`属性：

    ```php
    <input type="number" name="quantity" value="0" data-  quantityItem="{{ forloop.index }}"/>
    ```

1.  在确保我们已放置所有必要的属性后，我们只需要使用`addEventListener`来检测输入上的`change`事件，然后将对象传递给`changeCart()`箭头函数：

    ```php
    const changeCart = (el) => {
      let formData = {
        line: el.dataset.quantityItem,
        quantity: el.value
      };
      fetch('/cart/change.js', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(formData)
      })
      .then(success => {
        console.log("Success:", success);
      })
      .catch((error) => {
        console.error('Error:', error);
      });
    }
    ```

`changeCart()`箭头函数与我们之前创建的函数类似。唯一的区别是，现在我们正在使用`/cart/change.js`端点，而不是使用静态值作为键值对。相反，我们从之前传递的对象中获取这两个值。

虽然我们可以使用`/cart/update.js`和`/cart/change.js`通过将`quantity`值设置为`0`来简单地从购物车中移除项目，但我们必须手动调整每个`line`项的数量到`0`。*但如果我们想要一个简单的方法，通过单次点击就能清空整个购物车呢？*

## `/cart/clear.js`端点

与之前的端点相比，`/cart/clear.js`端点非常简单易用，因为它不接受任何参数。我们只需提交一个带有`/cart/clear.js`的 POST 请求，购物车就会自动清空所有现有项目：

```php
const clearCart = () => {
  fetch('/cart/clear.js', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    }
  })
  .then(success => {
    console.log("Success:", success);
  })
  .catch((error) => {
    console.error('Error:', error);
  });
}
```

注意，如果我们要在购物车中运行前面的代码，我们会成功清除购物车中的所有项目。然而，我们仍然需要刷新购物车页面才能看到变化，因为尽管我们已经从当前的购物车会话中清除了所有项目，但我们并没有从实际的 DOM 中移除这些项目。我们可以在`success`函数中实现一个短的`while`语句，并移除所有行项目元素以解决这个问题：

```php
const cartItems = document.querySelector("[data-cart-line-  items]");
while (cartItems.firstChild) {
  cartItems.removeChild(cartItems.firstChild);
}
```

使用前面的代码示例，我们已经成功从当前的购物车会话和 DOM 中移除了所有项目。请注意，还有许多其他需要处理的微调方面，例如清除价格、移除购物车表格，并显示一条消息，说明购物车为空。然而，为了使这本书简洁明了，我们不会深入探讨这一点，但你可以自由（并且建议）根据需要升级前面的代码，因为这将只会对你有益。

到目前为止，我们已经学习了如何使用 `/cart/add.js` 端点向当前购物车会话添加产品，使用 `/cart/update.js` 和 `/cart/change.js` 更新现有行项目，以及如何使用 `/cart/clear.js` 端点清除当前购物车会话。然而，正如我们所看到的，虽然我们可以轻松添加、更新或甚至清除当前购物车会话中的项目，但我们仍然需要重新加载页面才能看到特定的结果，例如更新位于页眉中 *购物车* 图标附近的项目 `counter` 或更新项目数量时的行项目价格。

当我们这样做时，我们可以快速简单地通过添加到购物车中的产品数量来增加项目 `counter`。实现这一点的更直接的方法是使用与 Shopify Ajax API 结合的 GET 请求，这将允许我们从 Shopify 服务器检索所有类型的数据，包括当前购物车会话中的产品数量。

# 使用 GET 请求检索数据

正如我们之前提到的，使用 GET 请求，我们可以从 Shopify 服务器获取所有类型的数据，除了客户和订单信息，这些信息只能通过认证的 Shopify Admin API 访问。根据我们想要执行的操作类型，我们可以将 GET 请求与以下端点配对：

+   `/cart.js`

+   `/products/{product-handle}.js`

+   `/recommendations/products.json`

+   `/search/suggest.json`

GET 请求是一种相当强大的方法，我们将经常与 POST 请求结合使用，在更改当前购物车会话后检索数据。然而，我们也可以使用 GET 请求来检索和创建复杂的功能，正如我们即将学习的。

## `/cart.js` 端点

如其名称所示，`/cart.js` 端点允许我们访问当前的购物车会话并检索有关购物车以及购物车中产品的所有信息。我们可以使用它来动态更新购物车页面，甚至为商店创建购物车抽屉，从而显著改善购买流程。让我们看看：

1.  我们可以使用以下 `fetch` 方法检索有关购物车的信息：

    ```php
    fetch('/cart.js')
      .then(response => response.json())
      .then(data => {
        console.log(data);
    });
    ```

    注意，成功的 GET 请求的响应是一个 JSON 对象。以下示例显示了使用前面的代码获取购物车数据时我们将收到的响应：

    ```php
    items array object, which we have minified to keep everything concise.
    ```

1.  现在我们已经学会了如何检索当前购物车会话信息，我们可以将其与之前我们工作的 `/cart/add.js` 的 `POST` 请求结合起来，并确保每次我们向购物车添加新产品时购物车计数器都能正确更新：

    ```php
    fetch('/cart/add.js', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(formData)
    })
    .then(success => {
      console.log("Success:", success);
      fetch('/cart.js')
      .then(response => response.json())
      .then(data => {
        document.querySelector("[data-cart-        count]").innerHTML = data.item_count;
      });
    })
    .catch((error) => {
      console.error('Error:', error);
    });
    ```

    我们现在拥有从当前购物车会话中检索不同类型信息所需的所有知识。然而，请注意，在购物车响应中价格值是纯字符串，没有货币格式。

1.  例如，假设我们想在更新产品数量时更新购物页面的总价格。首先，我们将使用 `fetch` 方法检索总价格值：

    ```php
    fetch('/cart.js')
      .then(response => response.json())
      .then(data => {
        console.log(data.total_price);
    });
    ```

    虽然我们成功检索到了价格，但我们收到的只是一个未格式化的字符串值，这对我们来说并不那么有用：

    ```php
    78594
    ```

1.  解决这个问题最简单的方法是查看主题开发者如何在主题中定义货币格式化辅助函数。我们通常可以在主题的 `master js` 文件中找到它。在我们的例子中，这将是一个 `theme.js` 文件。

    在确定我们需要的关键词后，我们只需将格式应用到我们想要格式化的值上：

    ```php
    fetch('/cart.js')
      .then(response => response.json())
      .then(data => {
        console.log(theme.Currency.formatMoney(        cart.total_price, theme.moneyFormat));
    });
    ```

注意，我们在前面的示例中使用过的格式在大多数情况下无需修改即可正常工作。然而，我们可能需要根据特定的主题进行调整——这完全取决于主题开发者如何定义该函数。

之前，我们学习了如何检索当前购物车会话的信息以及购物车中任何产品的数据。*然而，如果我们想更直接地检索产品信息呢？*

## `/products/{product-handle}.js` 端点

`/products/{product-handle}.js` 端点是一个简单的端点，我们可以将其与 GET 请求结合使用，轻松检索商店中任何产品的信息。同样，与 `/cart.js` 端点一样，`/products/{product-handle}.js` 相对容易使用，因为它只需要我们包含我们想要检索数据的产品的处理程序：

```php
const getProduct = (handle) => {
  fetch(`/products/${handle}.js`)
  .then(response => response.json())
  .then(product => {
    console.log(product.id);
  });
}
```

从前面的示例中我们将收到的返回值将包括产品 ID，我们将在下面的示例中使用它。

这个端点最常用的用途是在创建点击功能时，例如快速查看功能，我们需要动态加载大量产品信息以避免 DOM 杂乱无章并减慢商店速度。

## `/recommendations/products.json` 端点

`/recommendations/products.json` 端点允许我们根据 Shopify 算法检索关于所选产品的推荐产品列表的 JSON 对象，我们可以使用这些对象构建动态推荐部分。

通过这个端点，我们可以使用三个参数，其中一个是必需的，而其他两个是可选的：

+   `product_id`参数是一个必填参数，其值应设置为我们要检索推荐列表的产品 ID。请注意，产品 ID 与变体 ID 不同，它们是两个不同的属性，我们可以通过`product`对象来获取。

+   `limit`参数是一个可选参数，允许我们选择每次请求应接收的最大推荐产品数量。由于 Shopify 的限制，我们每次请求不能检索超过 10 个推荐产品。如果我们没有设置`limit`参数，这将默认值。

+   最后但同样重要的是`section_id`参数，虽然它是可选的，但这个参数相当有趣，因为它允许我们更改我们将接收的响应类型。通过将部分 ID 作为`section_id`参数值包含，我们可以选择我们想要渲染推荐产品的父元素。更重要的是，我们还可以将 JSON 响应更改为 HTML 字符串，然后我们可以将其与`recommendations`对象结合使用，以动态输出推荐产品。

现在我们已经熟悉了所有可以与`/recommendations/products.json`端点一起使用的属性，是时候看看它们在实际中的应用了。

在下面的示例中，我们使用了一个`fetch`请求，配合`/recommendations/products.json`端点，生成一个 JSON 对象列表并在控制台日志中输出：

```php
const productRecommendations = (productId, limit) => {
fetch(`/recommendations/products.json?product_id=  ${productId}&limit=${limit}`)
  .then(response => response.json())
  .then(products => {
    console.log(products);
  });
}
```

如我们所见，检索推荐产品的 JSON 对象相当简单，因为我们现在需要做的只是将产品 ID 传递给`productId`参数。如您所回忆，`limit`参数是可选的，如果不包含，将默认为最大值`10`。现在，让我们看看如何包含`section_id`并学习如何检索 HTML 字符串。

在我们能够修改`fetch`请求以实现这一点之前，我们需要做一些具体的准备。我们首先需要做的是在`Sections`目录中创建一个新的部分。在我们的例子中，我们将命名为`recommended-products`。

由于我们在产品页面模板上已经有了推荐产品部分，现在只是创建一个新的部分来了解它是如何工作的，所以让我们将这个新部分包含在`theme.liquid`布局文件的底部，正好在`</body>`标签之上。现在我们已经创建了部分文件并成功包含它，我们必须熟悉`recommendations`对象。

如其名所示，`recommendations`对象允许我们从产品推荐列表中检索产品。然而，这个特定的对象仅与`/recommendations/products`端点结合使用。

如我们所见，`recommendations`对象相对简单易用，因为它只包含三个属性：

+   `performed`属性返回一个布尔值，取决于我们是否将`recommendations`对象放置在通过组合`recommendations`端点和必要的参数而渲染内容的部分中。

+   `products_count`属性为我们提供了推荐列表中产品数量的数值。

+   最后但同样重要的是，`products`属性允许我们检索推荐产品对象的数组。我们可以将`products`属性与`for`标签结合使用，以提供与我们在*第五章*，“使用过滤器深入液态核心”的`Custom collection`项目中相同的方式提供输出。

让我们回到我们创建的`recommended-products`部分文件，并使用`recommendations`对象输出推荐产品数组：

```php
<div class="product-recommendations">
  {% if recommendations.performed %}
    {% if recommendations.products_count > 0 %}
      {% for product in recommendations.products %}
        <div class="product">
          <a href="{{ product.url }}">
            <img class="product__img" src="img/{{             product.featured_image | img_url: '300x300' }}" 
            alt="{{ product.featured_image.alt }}" />
            <p class="product__title">{{ product.title }}</p>
            <p class="product__price">{{ product.price |                 money}}</p>
          </a>
        </div>
      {% endfor %}
    {% endif %}
  {% endif %}
</div>
```

这样，我们就为未来的推荐列表创建了一个合适的布局。然而，如果我们预览主题上的结果，我们会注意到该部分在`product-recommendations`div 内没有渲染任何内容。正如我们之前提到的，`recommendations`对象仅与`recommendations`端点结合使用，所以让我们看看如何使用端点生成必要的 HTML 字符串以输出推荐产品列表。

为了实现这一点，我们需要对我们的之前的`fetch`请求做一些调整：

1.  我们需要做的第一件事是为`productRecommendations`函数包含额外的参数，我们将传递部分 ID 值。此外，我们还需要将`section_id`参数及其值包含到 fetch URL 中。

1.  第二个且更为重要的步骤是移除 fetch URL 中的`.json`。否则，我们将无法检索到 JSON HTML 字符串。

1.  最后但同样重要的是，我们需要在第一个`then`方法中将`.json()`替换为`.text()`。

到目前为止，我们已经拥有了检索 JSON HTML 字符串所需的所有必要元素。让我们通过在控制台日志中调用产品来测试它：

```php
const productRecommendations = (productId, limit,   sectionId) => {
fetch(`/recommendations/products?product_id=${productId}  &limit=${limit}&section_id=${sectionId}`)
  .then(response => response.text())
  .then(products => {
    if (products.length > 0) {
      console.log(products);
    }
  });
}
```

然而，在我们能够测试之前，我们需要将三个值传递给我们的函数：

+   对于`productId`，我们可以使用我们在学习`/products/{product-handle}.js`端点时检索到的产品 ID 值。或者，我们可以在任何产品模板中使用`product.id`并复制我们收到的值。

+   对于`limit`，我们可以使用任何不超过`10`的数字值，这是我们可以作为响应接收的最大产品数量。

+   对于`sectionId`，我们应该包含一个等于我们想要在内部显示推荐产品的部分名称的字符串值。在我们的例子中，值是`recommended-products`。

以下是将所有三个值传递给我们的函数的示例：

```php
productRecommendations(6796616663194, 3, "recommended-  products");
```

如果我们要预览我们的重复主题并检查之前`fetch`函数中的控制台日志，我们会看到我们已经成功检索到了推荐产品的 JSON HTML 字符串值。

现在我们已经确认一切正常，我们唯一要做的就是使用检索到的值并输出推荐产品列表：

```php
const productRecommendations = (productId, limit,   sectionId) => {
fetch(`/recommendations/products?product_id=${productId}  &limit=${limit}&section_id=${sectionId}`)
  .then(response => response.text())
  .then(products => {
    if (products.length > 0) {
      document.querySelector(".product-          recommendations").innerHTML = products;
    }
  });
}
```

有了这些，我们已经成功学习了如何根据我们在`recommended-products`部分之前定义的布局渲染推荐产品列表。此外，产品列表将根据 Shopify 的算法自动更新。

虽然有一个推荐的产品列表是一个很好的功能，用于查找类似的产品，但我们仍然需要导航到特定的产品，即使如此，我们也不能确定我们会收到我们需要的确切结果。为了帮助我们，我们可以使用预测搜索功能。

## `/search/suggest.json`端点

如其名所示，`/search/suggest.json`端点允许我们创建一个预测搜索，该搜索将自动为我们提供一个与我们的查询部分或完全匹配的产品列表。

除了允许我们在产品上使用预测搜索外，我们还可以根据我们包含的参数类型搜索集合、页面甚至文章。`/search/suggest.json`允许我们使用七种不同的参数类型。然而，为了保持简洁并切中要点，我们只会介绍使预测搜索功能正常工作所需的最重要的一些参数：

+   我们列表中的第一个参数是`q`参数，这是一个必填的字符串类型参数，其值应等于搜索查询。

+   `type`参数允许我们指定我们想要接收的结果类型。我们可以包含以下以逗号分隔的值：`product`、`page`、`article`和`collection`。`type`参数也是必填的。

+   `limit`参数是一个可选的整数参数，允许我们设置每次请求应接收的结果数量。请注意，如果我们不包括`limit`属性，它默认为`10`，这是我们每次请求可以接收的最大结果数量。

+   `resources`属性是一个必填的哈希类型参数，它根据`type`和`limit`字段请求查询的资源结果。

在以下示例中，我们使用了一个`fetch`请求，配合`/search/suggest.json`端点，生成一个与我们的搜索查询匹配的 JSON 对象列表，并在控制台日志中输出：

```php
const predictiveSearch = (query, limit, type) => {
fetch(`/search/suggest.json?q=${query}&resources[type]=  ${type}&resources[limit]=${limit}`)
  .then(response => response.json())
  .then(suggestions => {
    const productSuggestions =         suggestions.resources.results.products;
    if (productSuggestions.length > 0) {
      console.log(productSuggestions);
    }
  });
}
```

如我们所见，根据搜索查询检索预测搜索结果相对简单，因为我们现在唯一需要做的是将所需的值传递给我们的函数：

```php
const searchSelector = document.querySelectorAll     (".search-bar__input");
if (searchSelector.length) {
  for (let i = 0; i < searchSelector.length; i++) {
    searchSelector[i].addEventListener('input',       function(e){
      e.preventDefault();
      predictiveSearch(this.value, 4,           "product,page,article,collection");
    });
  }
}
```

在搜索框内输入搜索查询后，我们会注意到我们已经在控制台日志中成功检索到了最多四个产品、页面、文章或 JSON 对象的集合，这些集合部分或完全符合我们的搜索查询。现在我们唯一要做的就是使用响应值在 DOM 内生成结果。

使用 `/products/{product-handle}.js` 端点，我们有一个参数允许我们检索一个 JSON HTML 字符串，以便轻松将其输出到 DOM 中。然而，对于 `/search/suggest.json` 端点来说并非如此；为了渲染这些结果，我们需要使用 JavaScript 来创建所需的布局和功能。为了保持简洁并直击要点，我们不会在本书中涵盖这一点。然而，我们建议完成这个项目，因为它将是一个极好的实践，将帮助你巩固迄今为止所学的一切。

如需有关预测搜索参数及其一般要求和限制的更多信息，请参阅[`shopify.dev/api/ajax/reference/predictive-search`](https://shopify.dev/api/ajax/reference/predictive-search)。

# 摘要

在本章的最后，我们熟悉了 Shopify Ajax API，并了解了不同类型的用例。首先，我们学习了如何使用 `/cart/add.js` 端点升级当前的购买流程，通过这个端点，我们可以将任何数量的产品、数量和行项目定制直接添加到当前的购物车会话中。

通过学习如何处理 `/cart/change.js` 端点，我们获得了创建包含特定产品和数量的功能所需的知识，例如自动赠品或升级销售功能。结合使用 `/cart/update.js` 和 `/cart.js` 端点，我们学会了如何动态更新购物车的内容并检索它。然后我们可以利用这一点来创建购物车抽屉功能。

此外，我们还学会了如何使用 `/products/{product-handle}.js` 端点检索推荐产品的自动列表，并将它们的内容渲染到我们选择的区域。

最后，我们了解了 `/search/suggest.json` 端点，它允许我们创建预测搜索功能，这是店主们最常请求的功能之一。

从这本书的最初开始，我们就一起努力拓展我们的知识边界，并建立了一个坚实的理解流程，这将帮助我们成为 Shopify 专家的道路上。虽然我们没有覆盖每一行 Liquid 代码，但我们参与了一些令人兴奋的项目，从中我们学到了很多更有益的知识。我们的目标不仅仅是创建一个列表，列出所有不同的方法和属性，这些我们总是可以通过查阅 Shopify 文档找到，而是要学习 Shopify 和 Liquid 如何工作。

虽然可以说，通过在这里获得的知识，我们应该准备好独立开始 Shopify 主题的开发工作，但请注意，我们的冒险之旅并未结束——它才刚刚开始。

Shopify 是一个不断发展的平台，我们需要跟上所有最新的公告和策略。幸运的是，Shopify 提供了各种社区，以进一步丰富我们的知识或从其他 Shopify 专家那里获得各种主题的帮助。最后但同样重要的是，我们有一个可用的 Discord 频道，我们可以与其他开发者交谈，在我们需要时获得帮助，或者与开发者分享我们的知识：[`discord.gg/shopifydevs`](https://discord.gg/shopifydevs).

# 进一步阅读

+   Shopify 官方文档：[`shopify.dev/`](https://shopify.dev/)

+   Shopify 速查表：[`cheat.markdunkley.com/`](http://cheat.markdunkley.com/)

+   开发者变更日志：[`shopify.dev/changelog`](https://shopify.dev/changelog%20)

+   官方社区：[`community.shopify.com/`](https://community.shopify.com/%20)

+   Twitter 公告：[`twitter.com/shopifydevs`](https://twitter.com/shopifydevs%20)

+   Shopify 开发者 YouTube 频道：[`www.youtube.com/channel/UCcYsEEKJtpxoO9T-keJZrEw`](https://www.youtube.com/channel/UCcYsEEKJtpxoO9T-keJZrEw%20)

+   Shopify 官方博客，提供关于 Shopify 世界的最新信息：[`www.shopify.com/partners/blog`](https://www.shopify.com/partners/blog)
