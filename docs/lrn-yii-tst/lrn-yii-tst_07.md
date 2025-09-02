# 第七章。享受浏览器测试的乐趣

我们终于到达了测试的最后阶段：**验收测试**。这是使用 Codeception 在 Yii 中测试应用程序的最高方式。

正如我们在初始章节中看到的，功能测试和验收测试在形式和实现上相当相似，所以你在这个章节中不会看到任何特别新的内容。

掌握我们将要创建的测试的本质非常重要。我们可以从第六章，*测试 API – PHPBrowser 来拯救*中回忆起，功能测试用于确保我们从更高的角度对所构建的内容进行技术正确性验证，这比单元测试更为重要。而验收测试是确保在所有内容实施并整合后，最初定义的验收标准仍然有效度的最佳方式。

在本章中，我们将回顾现有的测试，安装和配置 **Selenium**，然后实现一个小功能，将所有已经完成的工作结合起来。

在本章中，我们将讨论两个主要主题：

+   介绍 Selenium WebDriver

+   创建验收测试

# 介绍 Selenium WebDriver

Yii 随带了一些验收测试。它们涵盖了我们已经看到的相同元素。这两个测试之间的唯一区别是技术性的，通过查看配置，你可以看到它们已经被配置为使用 PHPBrowser 运行。这种设置可能对你来说已经足够好，或者甚至更好，因为 PHPBrowser 的运行速度比可用的验收测试套件要快。

除了我们在第六章，*测试 API – PHPBrowser 来拯救*中已经介绍过的 PHPBrowser 之外，Codeception 可以与其他测试套件一起使用，这些套件可以执行更真实的端到端测试，包括 JavaScript 交互。

你可以选择的两个选项是 Selenium WebDriver 和 PhantomJS。我们不会涉及 PhantomJS，但你应该知道它是一个无头浏览器测试套件，它使用 WebDriver 接口定义。

Selenium WebDriver 也被称为 Selenium 2.0 + WebDriver。与 Cucumber 一起，它可能是最知名的端到端测试工具。它们的使用已经被一些大公司，如 Google，所改进。它们稳定且功能丰富。

这是对 Selenium 1.0 的某种自然演变，Selenium 1.0 存在一些限制，例如使用 JavaScript 与网页交互。因此，它运行在 JavaScript 沙盒中。这意味着，为了绕过同源策略，它必须与 Selenium RC 服务器一起运行，这导致了一些与浏览器设置相关的问题。

现在 Selenium 服务器已经取代了 RC，同时保持向后兼容并原生支持 WebDriver。

WebDriver 使用浏览器的本地实现与之交互。这意味着它可能并不总是适用于特定的语言/设备组合。然而，它提供了在不进行模拟的情况下控制页面的最佳灵活性。

Codeception 使用由 Facebook 开发的 PHP 实现，称为 php-webdriver；其源代码和问题跟踪器可以在 [`github.com/facebook/php-webdriver`](https://github.com/facebook/php-webdriver) 找到。

在其最简单实现和配置中，Selenium 服务器仅作为特定机器上的服务监听调用，并在请求执行测试时启动浏览器。

因此，作为第一步，我们需要安装 Selenium 服务器，运行它，在 Codeception 中配置它，调整现有的测试以便它们能够与之协同工作，然后向其中添加新的测试。

## 安装和运行 Selenium 服务器

从 1.7 版本开始，Codeception 包含了现成的 php-webdriver 库。

如文档所述，这些文档可以从 Codeception ([`codeception.com/11-20-2013/webdriver-tests-with-codeception.html`](http://codeception.com/11-20-2013/webdriver-tests-with-codeception.html)) 或 Selenium 的官方网站 ([`docs.seleniumhq.org/docs/03_webdriver.jsp`](http://docs.seleniumhq.org/docs/03_webdriver.jsp)) 获取，您需要下载服务器二进制文件，然后在您打算运行浏览器的机器上运行它。

访问 [`www.seleniumhq.org/download/`](http://www.seleniumhq.org/download/) 并下载软件的最新版本。在我的情况下，它将是 `selenium-server-standalone-2.44.0.jar`。

你保存它的位置并不重要，因为一旦它启动，它的服务器将监听任何网络接口：

```php
$ java -jar selenium-server-standalone-2.44.0.jar
22:03:31.892 INFO - Launching a standalone server
22:03:31.980 INFO - Java: Oracle Corporation 24.65-b04
22:03:31.980 INFO - OS: Linux 3.17.1 amd64
22:03:32.002 INFO - v2.44.0, with Core v2.44.0\. Built from revision 76d78cf
...

```

### 配置 Yii 以与 Selenium 协同工作

为了让 Codeception 自动获取并使用 WebDriver，我们需要调整我们的验收套件配置：

```php
# tests/codeception/acceptance.suite.yml
class_name: AcceptanceTester
modules:
    enabled:
        - WebDriver
    config:
        WebDriver:
            url: 'http://basic-dev.yii2.sandbox'
            browser: firefox
            host: 192.168.56.1
            restart: true
            window_size: 1024x768
```

这是一个简单的过程。我们需要用 WebDriver 替换 PHPBrowser 模块，并对其进行配置。

+   `url`（必需）：这是用于连接到您的应用程序以执行测试的主机名。

+   `browser`（必需）：这将指定您想要使用的浏览器。还有一些其他驱动程序适用于手机（Android 和 iOS），更多关于这些的信息可以从在线 Selenium 文档中获得，该文档可在 [`docs.seleniumhq.org/docs/03_webdriver.jsp#selenium-webdriver-s-drivers`](http://docs.seleniumhq.org/docs/03_webdriver.jsp#selenium-webdriver-s-drivers) 找到。

+   `host`：此键指定将运行 Selenium 服务器的机器。默认情况下，它将连接到您的本地主机。例如，我正在使用 VirtualBox 主机机的 IP 地址。您也可以指定端口号，默认情况下，它将使用 `4444`。

+   `restart`：这告诉 WebDriver 在执行测试时重置会话。如果你不希望状态从一个测试传递到另一个测试，这特别有用。例如，当你需要（重新）设置 cookies 来测试自动登录功能时，可以使用此功能。

+   `window_size`：这仅指定窗口的大小。

还有其他一些选项，其中一些在测试多个浏览器时非常有用。特别是，你可以设置 Selenium 2.0 的期望能力，例如能够传递浏览器的特定配置文件（在执行回归测试时非常有用）等等。有关 WebDriver 模块的信息，尽管不是我想看到的那么多，可以在 Codeception 文档页面上找到，网址为 [`codeception.com/docs/modules/WebDriver`](http://codeception.com/docs/modules/WebDriver)。

## 实现由 WebDriver 引导的测试

在我们开始实现将挂钩到 API 的接口之前，该 API 我们在 第六章 中实现，*测试 API – PHPBrowser 来拯救*，查看现有的验收测试并查看是否有任何新内容需要我们考虑将会非常有用。

你将找到四个测试：`HomeCept`、`AboutCept`、`LoginCept` 和 `ContactCept`。

如前所述，语法并不罕见，我们可以看到对底层结构的了解程度比我们已涵盖的功能测试要有限。

我们需要再次强调的重要事情是，我们的 `AcceptanceTester` 可以在页面上执行的所有操作，例如 `click()`、`fillField()`，以及它可以执行的所有断言，例如 `see()`、`seeLink()` 等，都接受所谓的定位器作为其实际参数之一。

定位参数可以是字符串或数组。

当作为字符串或所谓的**模糊定位器**（在 Codeception 术语中称为）传递时，它将通过正式走过一系列步骤来猜测你正在寻找的内容。如果你编写 `click('foo')`，那么它将执行以下操作：

1.  它试图找到具有 ID `#foo` 的元素。

1.  它试图找到类 `.foo`。

1.  然后，它将其解释为 XPath 表达式。

1.  最后，它将抛出 `ElementNotFound` 异常。

当使用数组表示法或**严格定位器**时，你可以更加具体。

+   `['id' => 'foo']` 匹配 `<div id="foo">`

+   `['name' => 'foo']` 匹配 `<div name="foo">`

+   `['css' => 'input[type=input][value=foo]']` 匹配 `<input type="input" value="foo">`

+   `['xpath' => "//input[@type='submit'][contains(@value, 'foo')]"]` 匹配 `<input type="submit" value="foobar">`

+   `['link' => 'Click here']` 匹配 `<a href="google.com">Click here</a>`

+   `['class' => 'foo']` 匹配 `<div class="foo">`

以下示例取自 Codeception 文档，可以在[`codeception.com/docs/modules/WebDriver`](http://codeception.com/docs/modules/WebDriver)找到。

这清楚地说明了如何与网页进行交互。现在，其他重要的部分可以在现有的测试中找到，例如`LoginCept`和`ContactCept`。在这里，在断言验证错误的存在之前，我们有以下条件语句：

```php
if (method_exists($I, 'wait')) {
    $I->wait(3); // only for selenium
}
```

Selenium 引入了两种等待类型：隐式等待和显式等待。这会导致从服务器获取信息，然后对这些信息进行解释和渲染。

隐式等待可以在`acceptance.suite.yml`文件中进行配置，并且它会静默地告诉 Selenium 在它寻找的元素不可立即获得时，每隔*X*秒进行轮询。默认情况下，没有设置隐式等待。

显式等待与前面的代码片段类似。执行简单的`$I->wait(X)`会触发`sleep()`，并允许浏览器执行所需的操作。例如，它可以帮助浏览器完成动画或完成从服务器端获取和操作数据的操作。

您还有其他等待某种东西的方法，其中一些方法可能更加积极主动，例如`waitForElement()`、`waitForElementChange()`、`waitForElementVisible()`和`waitForElementNotVisible()`。所有这些方法都接受一个定位器，使用上述格式，以及以秒为单位的超时作为参数。我们将在稍后看到如何使用这些方法。

WebDriver Codeception 模块还提供了其他方法，您可以在调试测试的同时使用这些方法，以防事情没有按照您的预期进行。

现在，让我们尝试运行可用的测试并查看它们通过：

```php
$ ../vendor/bin/codecept run acceptance
Codeception PHP Testing Framework v2.0.9
Powered by PHPUnit 4.6-dev by Sebastian Bergmann and contributors.

Acceptance Tests (5) --------------------------------------------------------------
Trying to ensure that about works (AboutCept) Ok
Trying to ensure that contact works (ContactCept) Ok
Trying to ensure that home page works (HomeCept) Ok
Trying to ensure that login works (LoginCept) Ok
--------------------------------------------------------------

Time: 38.54 seconds, Memory: 13.00Mb

OK (4 tests, 23 assertions)

```

### 注意

根据您将用于运行这些测试的机器，它们的速度将会有所变化，尽管它永远不会像执行单元测试那样快，这主要是因为整个浏览器堆栈必须启动才能运行这些测试。

在执行测试的过程中，您将简要看到浏览器打开和关闭几次。一切应该在最后看起来都很好，如果不好，那么请查看`tests/codeception/_output/`文件夹。在这里，您将找到在失败时捕获的页面标记和截图。这种调试行为在使用 PHPBrowser 进行功能测试时也存在。

# 创建验收测试

现在我们已经看到了使用 Selenium WebDriver 的验收测试的结构，我们可以开始整合上一章完成的工作，并开始添加我们想要的测试。

对于这类测试，通常标记的定义留给实现布局的人，您需要定义您界面的功能，实现测试，然后实现标记，并在需要时添加 JavaScript 功能。在完成这些操作后，您将添加 DOM 交互的具体细节。

了解有多少开发者将前端功能定义留到最后一刻，"测试先行"将迫使您改变工作方式，尽可能详细地预测前方的情况，并立即发现设计的任何关键方面。

我们将尝试实现一些足够简单的东西，让您可以从头开始，然后您可以稍后对其进行改进或扩展。我们知道我们使用的 HTTP Basic Auth 不允许有状态的登录。因此，我们将在 JavaScript 中保持某种类型的会话对象来模拟它。如何实现这一点可以从我们为用户 API 编写的测试中得知。这是最佳的实际文档。

因此，我们的场景可以这样描述：

```php
I WANT TO TEST MODAL LOGIN

I am on homepage
I see link 'Login'
I don't see link 'Logout' # i.e. I'm not logged
I am going to 'test the login with empty credentials'
.I click 'Login'
I wait for modal to be visible
I submit form with empty credentials
I see 'Error'

I am going to 'test the login with wrong credentials'
I submit form with wrong credentials
I see 'Error'

I am going to 'test the login with valid credentials'
I submit form with valid credentials
I don't see the modal anymore
I see link 'Logout'
I don't see link 'Login'

I am going to 'test logout'
I click 'Logout'
I don't see link 'Logout'
I see link 'Login'
```

上述语法是明确从我们的测试生成的文本版本中提取的。

## 实现模态窗口

通过 Bootstrap，这是一个与基本应用程序捆绑的默认框架，实现模态窗口的工作将变得简单。

模态窗口由几乎预先确定的标记和额外的 JS 组成，它提供了交互部分，从而将其连接到界面的其余部分。我希望其实现的简单性能让您专注于其背后的目标。

以下代码是故意从 Bootstrap 文档中提取的，该文档可以在[`getbootstrap.com/javascript/#modals`](http://getbootstrap.com/javascript/#modals)找到。由于模态窗口可以从网站的任何部分打开而不必转到登录页面，因此我们必须将其添加到整体布局模板中：

```php
    <!-- views/layouts/main.ph -->
    </footer>

    <div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal"><span aria-hidden="true">&times;</span><span class="sr-only">Close</span></button>
                    <h4 class="modal-title">Login</h4>
                </div>
                <div class="modal-body">
                    <?= $this->render('/site/login-modal.php', ['model' => Yii::$app->controller->loginForm]); ?>
                </div>
            </div>
        </div>
    </div>

<?php $this->endBody() ?>
<!-- ... -->
```

如您所见，我们已经将模态形式移动到了一个独立的模板中，该模板将作为变量接收表单的模型，保持一切自我包含和组织。请注意模型从哪里获取其值。我们将在实现控制器时讨论这个问题。

`login-modal.php` 模板只是原始 `login.php` 模板的复制品，可以在同一目录下找到，标题中没有 `H1`，也没有“记住我”复选框。我们只需要添加一个占位符来显示来自 API 的错误。这样做是为了通知和调试。

```php
<!-- views/site/login-modal.php
<!-- ... -->
<div class="alert alert-danger alert-dismissible fade in">
    <button type="button" class="close" data-dismiss="alert"><span aria-hidden="true">×</span><span class="sr-only">Close</span></button>
    <p class="error"></p>
</div>
<!-- ... -->
```

我们可以将这个片段放在复制的标记的第一段之后。

## 使服务器端工作

正如我们之前所说的，我们希望模态窗口在所有地方都可用。我们将通过在 `SiteController` 中保存一个公开可访问的属性来实现这一点，这样我们就可以从视图中检索它。记住，如果你是从 Yii 1 过来的，那么现在视图是独立的对象，而不是控制器的一部分。

让我们使用 `init()` 方法来这样做：

```php
// controllers/SiteController.php

public $loginForm = null;

public function init()
{
    $this->loginForm = new LoginForm();
}
```

一旦完成，我们可以无错误地加载我们的页面。

在下一步中，我们将添加与 JavaScript 的交互。

## 添加 JavaScript 交互

在本节中，我们将介绍一些内容。我们将讨论与模态的基本功能交互，与表单的交互，然后学习如何通过边缘情况和错误场景关闭一切。

与模态的交互将通过重用已经存在的位于菜单右上角的登录按钮来实现。我们将禁用它，但请记住，如果出现问题，它将提供回退兼容性。

模态窗口的基本打开和关闭是开箱即用的。我们只会在需要时触发它，例如在认证成功时。

让我们添加 JS 模块的第一个基本框架：

```php
// web/js/main.js

var YII = YII || {};
```

对于我们应用的这部分，我们需要模块模式来创建一个自包含的应用。

```php
YII.main = (function ($) {
    'use strict';
```

让我们先缓存所有我们在旅途中将要需要的 jQuery 元素：

```php
    var $navbar = $('.navbar-nav.nav'),
        $modal = $('#myModal'),
        $modalBody = $modal.find('.modal-body'),
        $modalAlertBox = $modal.find('.alert').hide(),
        $modalError = $modalAlertBox.find('.error'),
        $CTALogin = $navbar.find('.login'),
```

一旦我们登录，我们将交换链接以一个“假”注销按钮：

```php
        $CTALogout = $('<li class="logout"><a href="#">Logout</a></li>'),
```

我们需要一些数据字段来存储我们的登录信息并创建某种形式的会话：

```php
        authorization = null,
        username = null,
        userID = null;
```

现在是脚本的主要部分，我们将初始化我们的点击和提交动作的事件监听器：

```php
    /**
     * initialise all the events in the page.
     */
    (function init() {
        $navbar.append($CTALogout);
        $CTALogout.hide();
```

让我们先添加并隐藏我们的注销按钮；我们只会在登录成功时显示它，并定义它应该有的点击动作：

```php
        $navbar.on('click', '.logout a', function (e) {
            e.preventDefault();

            // unset the user info
            authorization = null;
            username = null;
            userID = null;

            // restore the login CTA
            $CTALogout.hide();
            $CTALogin.show();
        });
```

我们需要禁用登录按钮的点击事件。否则，我们将被带到登录页面，而不是打开模态窗口：

```php
        $navbar.on('click', '.login a', function (e) {
            e.preventDefault();
        });
```

模态触发事件是通过修改登录按钮的标记自动完成的。因此，导航到 `views/layouts/main.php` 并按照以下方式调整：

```php
'label' => 'Login',
'url' => ['/site/login'],
'options' => [
    'data-toggle'=>'modal',
    'data-target'=> '#myModal',
    'class' => 'login'
]
```

接下来，我们将处理表单提交：

```php
        $modalBody.on('submit', '#login-form', function (e) {
            e.preventDefault();
```

在禁用默认提交事件后，我们需要捕获用户名和密码，并将其保存以供将来使用：

```php
            username = this['loginform-username'].value;
            // we don't care to store the password... sorta
            authorization = btoa(username + ':' + this['loginform-password'].value);
```

`authorization` 变量将保存我们准备发送的授权头：

```php
            $.ajax(
                {
                    method: 'GET',
                    url: '/v1/users/search/' + username,
                    dataType: 'json',
                    async: false,
                    beforeSend: authorize,
                    complete: function (xhr, status) {

                        if (status === 'success') {
                            // save the user ID
                            userID = xhr.responseJSON.id;
                            // set the logout button
                            $CTALogin.hide();
                            $CTALogout.show();
                            // clear the status errors
                            $modalError.html('');
                            $modalAlertBox.hide();
                            // close the modal window
                            $modal.modal('hide');
                        }
                        else {
                            // display the error
                            $modalError.html('<strong>Error</strong>: ' + xhr.statusText);
                            $modalAlertBox.show();
                        }
                    }
                }
            );
        });
    })();
})(jQuery);
```

这段代码足够简单。在成功的情况下，我们保存用户 ID 以供后续调用，隐藏登录按钮并显示注销按钮，清除错误信息并隐藏模态窗口。否则，我们只显示错误信息。

`beforesend` 选项将由 `authorize` 函数初始化，该函数定义如下：

```php
    /**
     * modifies the XHR object to include the authorization headers.
     *
     * @param {jqXHR} xhr the jQuery XHR object, is automatically passed at call time
     */
    function authorize(xhr) {
        xhr.setRequestHeader('Authorization', 'Basic ' + authorization);
    }
```

在这样做之后，我们不再需要其他任何与页面交互的东西。所以，让我们把所有东西都放在一起。

## 将一切整合起来

到目前为止，我们只需将我们的 JS 添加到页面中，然后完成我们的测试。为了将我们的文件添加到页面中，我们需要了解资产和资产包是什么。

### 处理 Yii 2 资产包

Yii 2 根本改变了处理资产的方式。它引入了资产包的概念。

资产包是脚本和样式的集合，与过去相比，它们具有更高的可配置性。

这个基本应用已经有了基本的实现。所以，让我们导航到 `/assets/AppAsset.php` 并看看内容是如何组织的：

```php
<?php // assets/AppAsset.php

namespace app\assets;

use yii\web\AssetBundle;

class AppAsset extends AssetBundle
{
    public $basePath = '@webroot';
    public $baseUrl = '@web';
    public $css = [
        'css/site.css',
    ];
    public $js = [];
    public $depends = [
        'yii\web\YiiAsset',
        'yii\bootstrap\BootstrapAsset',
    ];
}
```

`AppAsset` 从 `yii\web\AssetBundle` 类扩展而来，它仅仅定义了一系列公共属性。

前两个属性，`$basePath` 和 `$baseUrl`，是最重要的。`$basePath` 定义了资产在公开可访问位置的位置，而 `$baseUrl` 定义了它们的资源如何链接到网页，即它们的 URL。

通过使用这两个属性，这个资产定义了所谓的“已发布资产”。换句话说，它定义了一个资产包，这些资产在公开可访问的位置可用。

您可以有“外部资产”，这些资产由外部位置的资源组成，以及“源资产”，这些资产不包含来自公开位置的资源。这些资产仅定义了一个 `$sourcePath` 属性，Yii 将它们复制到公开可访问的资产文件夹中，并相应地命名。

源资产通常由库和小部件提供，因此我们在这里不会涉及它们。建议将已发布的资产放入 `web/` 文件夹中以将资产合并到页面或页面上。

在前面的示例中，您看到我们定义了资产依赖项，在我们的情况下，这是通过 jQuery 和 Bootstrap 来实现的。这正是我们为什么使用它们来开发主要的 JavaScript 模块。

最后，我们需要了解如何使用资产包来处理我们的标记。这可以通过查看模板视图的顶部来实现。为此，导航到 `/views/layouts/main.php`。在这里，我们可以看到这两行：

```php
// views/layouts/main.php
use app\assets\AppAsset;
AppAsset::register($this);
```

请记住，虽然不特别建议，但将任何资产与特定布局关联的旧方法尚未被移除。这和 Yii 1 中的工作方式相同，即通过使用 `registerCssFile()` 和 `registerJsFile()`。

资产还有许多其他选项，例如压缩和编译 SASS 和 LESS 文件，使用 Bower 或 NPM 资产等。请访问文档页面，目前它处于良好的状态，并且相当全面，链接为 [`www.yiiframework.com/doc-2.0/guide-structure-assets.html`](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html)。

在我们的工作中，我们需要稍微调整由添加 JS 文件提供的资产包，并调整它将要添加到页面上的位置，否则在页面解析之前运行它时可能会遇到一些问题。考虑以下代码片段：

```php
    public $js = [
        'js/main.js',
    ];
    public $jsOptions = [
        'position' => \yii\web\View::POS_END
    ];
```

一旦你将前面的行添加到资产包中，你需要回到模态中包含的表单模板。实际上，这会引发一些问题，因为它需要将一些脚本注入页面以使客户端验证工作。这是一个主要问题；大多数时候，你将不得不覆盖`ActiveForms`的工作方式，所以你应该学习如何做到这一点。

```php
// views/site/login-modal.php

    <?php $form = ActiveForm::begin([
        'id' => 'login-form',
        'options' => ['class' => 'form-horizontal'],
        'enableClientScript' => false,
        'enableClientValidation' => false,
        'fieldConfig' => [
            'template' => "{label}\n<div class=\"col-lg-4\">{input}</div>\n<div class=\"col-lg-5\">{error}</div>",
            'labelOptions' => ['class' => 'col-lg-offset-1 col-lg-2 control-label'],
        ],
    ]); ?>
```

这里显示的两个选项将禁用客户端验证和任何额外的脚本功能。仅禁用其中一个选项不起作用。

我们现在可以加载页面，并且控制台不会显示任何错误消息。

## 完成测试

到目前为止，我们只需将我们的场景转换为实时代码。

让我们以创建单元测试和功能测试相同的方式开始创建测试：

```php
$ ../vendor/bin/codecept generate:cept acceptance LoginModalCept
Test was created in LoginModalCept.php

```

导航到文件，让我们首先断言初始语句：我们所在的位置，并确保我们没有登录：

```php
<?php
// tests/codeception/acceptance/LoginModalCept.php
$I = new AcceptanceTester($scenario);
$I->wantTo('test modal login');

$I->amOnPage(Yii::$app->homeUrl);
$I->seeLink('Login');
$I->dontSeeLink('Logout');
```

虽然这看起来像是一种确定用户是否登录的简单方法，但它达到了目的。如果我们发现需要更复杂的方法，我们总是可以在以后添加：

```php
$I->amGoingTo('test the login with empty credentials');
$I->click('Login');
$I->waitForElementVisible('.modal');
$I->fillField('#loginform-username', '');
$I->fillField('#loginform-password', '');
$I->click('#login-form button');
$I->see('Error', '.alert .error');
```

这个测试最重要的部分是使用显式等待，`waitForElementVisible()`。它做了它所说的：等待直到具有类.modal 的 DOM 元素渲染并可见。

最后的断言并没有检查任何特定的错误。所以你可以在这里添加任何级别的定制，因为我已经尽量保持尽可能通用。

下一个测试也是同样的情况：

```php
$I->amGoingTo('test the login with wrong credentials');
$I->fillField('#loginform-username', 'admin');
$I->fillField('#loginform-password', 'wrong password');
$I->click('#login-form button');
$I->see('Error', '.alert .error');
```

测试的这个有趣部分出现在我们尝试使用有效凭据访问时。实际上，正如我们在之前创建的脚本中所看到的，模态窗口将被关闭，登录按钮将被注销链接替换：

```php
$I->amGoingTo('test the login with valid credentials');
$I->fillField('#loginform-username', 'admin');
$I->fillField('#loginform-password', 'admin');
$I->click('#login-form button');
$I->wait(3);
$I->dontSeeElement('.modal');
$I->seeLink('Logout');
$I->dontSeeLink('Login');
```

为了做到这一点，我们需要为 AJAX 调用完成添加另一个显式等待，然后窗口将消失。使用`waitForElementNotVisible()`可能不起作用，因为它涉及动画。它还取决于你正在测试的系统的响应性，因为它可能不会按预期工作，有时会失败。所以，`wait()`似乎是解决问题的最简单方法。考虑以下代码片段：

```php
$I->amGoingTo('test logout');
$I->click('Logout');
$I->dontSeeLink('Logout');
$I->seeLink('Login');
```

最后一个测试不需要太多关注，你应该能够理解它而不会遇到任何问题。

现在我们已经编写了测试，让我们运行它们：

```php
$ ../vendor/bin/codecept run acceptance
Codeception PHP Testing Framework v2.0.9
Powered by PHPUnit 4.6-dev by Sebastian Bergmann and contributors.

Acceptance Tests (6) ---------------------------------------------
Trying to ensure that about works (AboutCept) Ok
Trying to ensure that contact works (ContactCept) Ok
Trying to ensure that home page works (HomeCept) Ok
Trying to ensure that login works (LoginCept) Ok
Trying to test modal login (LoginModalCept) Ok
------------------------------------------------------------------

Time: 47.33 seconds, Memory: 13.00Mb

OK (5 tests, 32 assertions)

```

## 测试多个浏览器

从版本 1.7 开始，Codeception 还提供了一种测试多个浏览器的方法。这并不特别困难，并且可以确保实现跨浏览器兼容性。

这通常是通过在 `acceptance.suite.yml` 文件中配置环境来完成的，在文件底部添加类似以下内容：

```php
env:
    chrome39:
        modules:
            config:
                WebDriver:
                    browser: chrome
    firefox34:
        # nothing changed
    ie10:
        modules:
            config:
                WebDriver:
                    host: 192.168.56.102
                    browser: internetexplorer
```

`env` 变量下的每个键都代表你想要在测试上运行的特定浏览器，这是通过覆盖我们已定义的默认配置来实现的。

### 小贴士

在 `env` 中，你可以覆盖 YAML 配置文件中指定的任何其他键。

你可以拥有几台机器，每台机器上运行不同版本的浏览器，Selenium 服务器在这些机器上监听，这样你还可以在决定为最近引入的新功能使用哪些 polyfills 时进行回溯兼容性测试，同时也取决于你的浏览器支持图表。

为了触发各种环境，只需在 `run` 命令后附加 `--env <环境>` 参数：

```php
$ ../vendor/bin/codecept run acceptance --env chrome39 --env firefox34 --env ie10

```

Internet Explorer 需要在主机机器上安装自己的驱动程序，并执行一些额外的步骤来正确设置，这可以在 Selenium 文档中找到，文档地址为 [`code.google.com/p/selenium/wiki/InternetExplorerDriver`](https://code.google.com/p/selenium/wiki/InternetExplorerDriver)。

## 理解 Selenium 的限制

到现在为止，你可能已经看到了 Selenium 的强大之处。通过使用浏览器原生功能，你最终可以以编程方式与网站交互。这将节省人类在执行重复性任务上花费的大量时间。重复性只有在人类手中才会成为问题，所以这实际上是一件好事。

不幸的是，Selenium 不能做任何事情，如果你已经开始研究它的全面使用和潜力，那么你可能已经注意到它的一些使用限制。

显然，任何类型的“像素完美”测试几乎都不可能使用 Selenium 来重现，尽管可以创建一些针对设计的测试，特别是针对响应式设计的测试。其他框架，如 Galen，覆盖了这一功能（[`galenframework.com/`](http://galenframework.com/))。

需要花一些时间讨论悬停效果，因为它们可能相当难以实现，你可能需要使用 `moveMouseOver()` 方法来触发它。

# 摘要

我们在本章中已经涵盖了测试的最后一个方面。我们回顾了提供的测试。我们还理解了任何额外的语法，配置了 Selenium，运行了第一批测试，然后继续将之前开发的 API 实现并绑定到具有模态登录功能的界面中。

在下一章中，我们将学习大量的日志以及我们的测试如何生成信息以更好地理解测试。我们还将看看我们是否遗漏了什么。
