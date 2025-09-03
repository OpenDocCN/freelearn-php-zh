# 第三章. 处理和装饰表单

在本章中，我们将涵盖：

+   创建表单

+   使用表单视图辅助工具

+   创建自定义表单元素和表单视图辅助工具

# 简介

在本章中，我们将讨论表单，特别是它们的生成和处理。表单在与用户通信中起着非常重要的作用，因为它是从用户那里接收信息的一种方式。同时，结合 JavaScript 和 PHP，使用表单对元素进行大量验证也是一个很好的方法。如果我们还能让它看起来很棒，我们为什么不这样做呢？

# 创建表单

这个配方涉及创建表单的不同方法，然后我们将讨论如何将元素添加到表单中。在这个配方的最后部分，我们将讨论如何验证表单，以及完成此任务的最佳方式。

## 准备工作...

创建和输出表单所需的至少有一个模块的基本 ZF2 骨架应用程序是必要的。

如果我们想使用表单注解，我们也需要在骨架中初始化 `Doctrine\Common`，因为它有解析注解的解析引擎。如果我们使用 composer（它包含在 Zend Framework 2 骨架中），我们可以简单地更新我们的 `composer.json` 文件，在所需部分添加以下行：

```php
"doctrine/common": ">=2.1",

```

### 小贴士

确保行尾的逗号只有在还有行在其下方时才存在。如果除了一个闭合括号之外没有其他行跟在此行之后，请勿添加逗号，因为它将导致过程失败。

接下来是运行 composer update 命令以确保其被安装，可以使用以下类似命令：

```php
php composer.phar update

```

如果我们不使用 composer，我们最好查看 Doctrine 项目网站 ([`www.doctrine-project.org/projects/common.html`](http://www.doctrine-project.org/projects/common.html)) 以获取有关如何安装此项目的更多信息。

## 如何做到这一点...

我们首先将讨论创建表单和元素，然后我们将讨论添加过滤器和验证。

### 创建基本表单

表单始终需要是以下之一：

+   从 `Zend\Form` 类扩展的类

+   使用 `Zend\Form\Annotation` 定义方法的类

### 定义一个从 Zend\Form 扩展的表单

我们将从定义第一个方法开始，通过从 `Zend\Form` 类扩展它。如果我们是 Zend Framework 2 (ZF2) 的新手，这可能是开始的最简单方法。

基本思想是我们的表单类应该从 `Zend\Form` 类扩展，并且至少有一个 `__construct` 方法来定义我们的元素。

让我们看看 `/module/Application/src/Application/Form/NormalForm.php` 文件中的以下示例：

```php
<?php

// We define our namespace here
namespace Application\Form; 

// We need to use this to create an extend 
use Zend\Form\Form; 

// Starting class definition, extending from Zend\Form
class NormalForm extends Form 
{
  // Define our constructor that sets up our elements 
  public function __construct($name = null) 
  {
    // Create the form with the following name/id
    parent::__construct($name);
  }
}
```

如果我们现在去我们的控制器，比如 `Application` 模块的 `IndexController`，我们可以通过在文件 `/module/Application/src/Application/Controller/IndexController.php` 中执行以下操作将表单输出到视图：

```php
<?php

// Namespace of the controller
namespace Application\Controller;

// Use the following classes at a minimum
use Zend\Mvc\Controller\AbstractActionController;
use Application\Form\NormalForm;
use Zend\View\Model\ViewModel;

// Begin our class definition
class IndexController extends AbstractActionController
{
  // Set up our indexAction,  in which we want to 
  // display our form.
  public function indexAction()
  {
    // Initialize our form
    $form = new NormalForm();

    // Return the view model to the user, with the 
    // attached form
    return new ViewModel(array(
        'form' =>  $form
    ));
  }
}
```

如果我们现在查看我们的视图脚本，我们可以看到我们有一个可用的变量。现在，我们将通过以下示例将表单实际输出到屏幕（文件`/module/Application/view/application/index/index.phtml`）：

```php
<?php
  // Output the opening FORM tag: <form>
  echo $this->form()->openTag($this->form); 

  // Output the formatted elements of the form
  echo $this->formCollection($this->form);

  // Output the closing FROM tag </form>
  echo $this->form()->closeTag();
```

这个代码示例的输出将类似于以下内容：

```php
<form action="" method="POST" name="normalform" id="normalform"></form>
```

这告诉我们实例化进行得很顺利，并且它是完全功能的。正如我们也可以看到，我们定义的名称（`"normalform"`）正在作为表单的`name`和`id`返回。

### 定义一个使用 Zend\Form\Annotation 的表单

让我们看看一个空表单（`/module/Application/src/Application/Form/AnnotationForm.php`）在注解表单中的样子：

```php
<?php

// We first define our namespace as usual
namespace Application\Form;

// We need to use this otherwise it will not parse the 
// elements correctly.
use Zend\Form\Annotation;

/**
 * We want to name this form annotationform, which is 
 * why we use the tag below, defining the name. 
 *
 * @Annotation\Name("annotationform")
 * 
 * A hydrator makes sure our framework can 'read' the 
 * properties in our object, in this case we tell our 
 * annotation engine that we have an object that needs 
 * its properties read. There is probably a more 
 * technical, accurate way of explaining it, but let's 
 * just keep it to this for now. 
 *
* @Annotation\Hydrator(
 *     "Zend\Stdlib\Hydrator\ObjectProperty
 * ")
*/
class AnnotationForm
{
  /**
   * If we want to exclude properties in our form just 
   * use the Exclude annotation.
   * 
   * @Annotation\Exclude()
   */
  public $id;
}
```

如果我们现在想开始将表单输出给用户，我们可以用类似正常表单的方式来做（幸运的是）。为此，我们首先需要做的是实际上再次将表单分配给视图（`/module/Application/src/Application/Controller/IndexController.php`），这是与正常表单创建略有不同的一点。

```php
<?php

// Namespace of the controller
namespace Application\Controller;

// Use the following classes at a minimum
use Zend\Mvc\Controller\AbstractActionController;
use Application\Form\AnnotationForm;
use Zend\Form\Annotation\AnnotationBuilder;
use Zend\View\Model\ViewModel;

// Begin our class definition
class IndexController extends AbstractActionController
{

  // Set up our indexAction,  in which we want to 
  // display our form.
  public function indexAction()
  {
    // Set up the output model
    $viewModel = new ViewModel;

    // Instantiate the AnnotationBuilder which will 
    // create the actual form object
    $builder = new AnnotationBuilder();

    // Instantiate our annotated form
    $annotationForm = new AnnotationForm();

    // Now let the annotation builder create the form 
    // from scratch
    $form = $builder->createForm($annotationForm);

    // Set our form to be the form variable in the view
    $viewModel->setVariable('form', $form);

    // Return the view model to the user
    return $viewModel;
  }
}
```

如果我们现在想将表单输出到我们的视图（文件`/module/Application/view/application/index/index.phtml`），我们可以简单地像处理其他表单一样做：

```php
<?php
  // Output the opening FORM tag: <form>
  echo $this->form()->openTag($this->form); 

  // Output the formatted elements of the form
  echo $this->formCollection($this->form);

  // Output the closing FROM tag </form>
  echo $this->form()->closeTag();
```

这个示例的 HTML 输出将产生以下内容：

```php
<form action="" method="POST" name="annotationform" id="annotationform"></form>
```

### 向 Zend\Form 扩展表单添加元素

在这种表单中创建元素相当简单，让我们通过一个简短的例子来看看它是什么样子（文件`/module/Application/src/Form/NormalForm.php`）：

```php
// Adding a simple input text field
public function __construct($name = null) 
{
  // Create the form with the following name/id
  parent::__construct($name); 

  $this->add(array( 	
    // Specifying the name of the field
    'name' => 'name', 

    // The type of field we want to show
    'type' => 'Zend\Form\Element\Text', 

    // Any extra attributes we can give the element
    'attributes' => array( 
      // If there is no text we will display the 
      // placeholder
      'placeholder' => 'Your name here...', 

      // Tell the validator if the element is required 
      // or not
      'required' => 'required', 
    ), 

    // Any extra options we can define
    'options' => array(  
      // What is the label we want to give this element
      'label' => 'What is your name?', 
    ), 
  )); 
}
```

### 向注解表单添加元素

让我们以一个注解元素创建的例子为例：

```php
class AnnotationForm
{
 /**
   * Add two filters to this element.
   *
   * @Annotation\Filter({"name": "StringTrim"})
   * @Annotation\Filter({"name": "StripTags"})

   * Add a validator to make sure the string length 
   * isn't going to be longer than 50, but also not 
   * smaller than 5.
   *
   * @Annotation\Validator({
   *    "name": "StringLength", 
   *    "options":{
   *        "min": 5, 
   *        "max": 50, 
   *        "encoding": "UTF-8"
   * }})
   *

   * Set this element to be required.
   * 
   * @Annotation\Required(true)

   * Set the attributes for the element
   *
   * @Annotation\Attributes({
   *     "type": "text", 
   *     "placeholder": "Your name here...", 
   * })

   * Set the options of this element.
   *
   * @Annotation\Options({
   *    "label": "What is your name?"
   * })
   */
  public $name;
```

### 验证表单输入

拥有表单最重要的一个方面是使用我们应用程序中的数据，因为如果不是为了使用这些数据，我们最初为什么要创建表单呢？

让我们去创建一个简单的模型（`/module/Application/src/Application/Model/SampleModel.php`），稍后我们可以用它作为例子，但这个模型对这道菜谱来说完全没有其他用途。

```php
<?php

namespace Application\Model;

class SampleModel
{
  public function doStuff($array) { 
    return true; 
  }
}
```

如我们所见，这个模型根本什么都没做，但我们稍后还需要它。

我们现在已经创建了自己的表单扩展，所以现在是时候创建我们的`InputFilter`类了，这个类将过滤和验证我们将要放入表单中的值，稍后通过`setInputFilter`将其附加到表单上（我们将编辑文件`/module/Application/src/Application/Form/NormalFormValidator.php`）：

```php
<?php

// Of course our namespace first
namespace Application\Form; 

// As this will be an input filter, we need the 
// following imports to make it work
use Zend\InputFilter\Factory as InputFilterFactory; 
use Zend\InputFilter\InputFilter; 
use Zend\InputFilter\InputFilterAwareInterface; 
use Zend\InputFilter\InputFilterInterface; 

// Create our class, which should be implementing the 
// InputFilterAwareInterface if we want to attach it to 
// the form later on
class NormalFormValidator implements 
InputFilterAwareInterface
{ 
  // This is the input filter that we will create
  protected $inputFilter; 

  // This method is required by the implementation, but 
  // we will just throw an exception instead of setting 
  //the input filter as we don't want anyone to override 
  // us
  public function setInputFilter(InputFilterInterface $inputFilter) 
  { 
    // We want to make sure that we cannot set an input 
    // filter, as we already do that ourselves
    throw new \Exception("Cannot set input filter."); 
  } 
```

我们现在已经开始创建我们的输入过滤器类，并且已经创建了`InputFilterAwareInterface`的两个必需方法之一。现在，让我们继续到实现第二个方法，并构建实际的过滤器：

```php
// This is the second method that is required by the 
// interface
public function getInputFilter() 
{ 
  // If our input filter doesn't exist yet, create one
  if ($this->inputFilter === null) {
    // Create the input filter which we will put in our 
    // property later
    $inputFilter = new InputFilter(); 

    // Also instantiate our factory so we can get more 
    // filters at ease
    $factory = new InputFilterFactory(); 

    // Let's add a filter for our name Element in our 
    // form
    $inputFilter->add($factory->createInput(array(
      // This is the element is applies to
      'name' => 'name', 

      // We want no one to skip this field, we need it
      'required' => true, 

      // Now we are defining the filters, which make 
      // sure that no malicious or invalid characters 
      // are supplied
      'filters' => array( 
        // Make sure no tags are in our value, which 
        // could make our system vulnerable for hacks
        array('name' => 'StripTags'), 

        // We want to make sure our string doesn't 
        // have any leading or trailing spaced	
        array('name' => 'StringTrim'), 
      ), 

      // Validators make the form generate errors when 
      // the data is invalid, filters only filter	
      'validators' => array( 
        array ( 
          // We want to add a validator that checks the 
          // length of the string received
          'name' => 'StringLength', 
          'options' => array( 
            // Check if the string is in UTF-8 encoding  
            // and between the 5 and 50 characters long
            'encoding' => 'UTF-8', 
            'min' => '5', 
            'max' => 50', 
          ), 
        ), 
      ), 
    )));
```

我们刚刚添加了一个简单的验证器，确保字符串的长度不小于 5 个字符，也不大于 50 个字符，当然，在我们的案例中，我们还想使用`UTF-8`字符，但显然，如果我们需要的话，我们可以取消选择这个选项或更改字符集。

现在，我们将添加一个简单的密码字段验证器和过滤器，但下一个验证器将检查`repeat_password`字段是否与密码字段值相同。我个人非常喜欢这个验证器，因为它简单而且足够强大，可以减少一些手动劳动。

```php
    // We are doing the same trick again for the 
    // password, so we can just skip over this, as this 
    // was just necessary for the one after this one.
    $inputFilter->add($factory->createInput(array(
      'name' => 'password', 
      'filters' => array( 
        array('name' => 'StripTags'), 
        array('name' => 'StringTrim'), 
      ), 
      'validators' => array( 
        array ( 
          'name' => 'StringLength', 
          'options' => array( 
            'encoding' => 'UTF-8', 
            'min' => '5', 
          ), 
        ), 
      ), 
    )));

    // And here is the great piece of validation we 
    // wanted to show off. This validator checks if the 
    // value of the given element is identical to 
    // another fields value. This way we don't have to 
    // manually check if the password is the same as the 
    // repeat password field.
    $inputFilter->add($factory->createInput(array(
      'name' => 'password_verify', 
      'filters' => array( 
        // The usual filters, as we almost always want 
        // to be sure it contains no tags or 
        //trailing/leading spaces
        array('name' => 'StripTags'), 
        array('name' => 'StringTrim'), 
      ), 
      'validators' => array( 
        array( 
          'name' => 'identical', 
          'options' => array( 
            'token' => 'password', 
          ), 
        ), 
      ), 
    )));
```

在那个巧妙的验证器之后，我们现在将添加一个简单的电子邮件验证器，它也将有一个非空验证器，用于检查字段是否为空。我们将使用以下代码进行电子邮件验证：

```php
// Email validator works perfectly, especially if we 
// don't want to trust any client side validation 
// (which we shouldn't)
$inputFilter->add($factory->createInput(array(
  'name' => 'email', 
  'filters' => array( 
     array('name' => 'StripTags'), 
     array('name' => 'StringTrim'), 
  ), 
  'validators' => array( 
    array ( 
       'name' => 'StringLength', 
       'options' => array( 
       'encoding' => 'UTF-8', 
       'min' => '5', 
       'max' => '250', 
        ), 
      ), 
      array( 
        // Don't you hate it when you get email 
        // addresses that are not valid? Well, no 
        // more as we can simply validate on that 
        // as well.
        'name' => 'EmailAddress', 
        'options' => array( 
        'messages' => array( 
            // We can even leave a neat little error 
            // message to display
            'emailAddressInvalidFormat' => 'Your email seems to be invalid', 
          ) 
        ), 
      ), 
      array( 
        // This validator makes sure the email 
        // address is not left empty. And although we 
        // can simply say this field is required, 
        // this will give us the opportunity to leave 
        // a nice error message that is relevant to 
        // the user as well
        'name' => 'NotEmpty', 
        'options' => array( 
        'messages' => array( 
            // This message is displayed when the 
            // field is empty, instead of a 'field 
            // required' message as we didn't make 
            // the field required
            'isEmpty' => 'I am sorry, your email is required', 
          ) 
        ), 
      ), 
    ), 
  )));
```

即使是日期验证也不是问题，我们甚至可以做得更好，只允许我们选择日期范围，这在某些情况下（例如 18+网站）是非常有用的。

```php
      $inputFilter->add($factory->createInput(array(
        'name' => 'birthdate', 
        'required' => true, 
        'filters' => array( 
          array('name' => 'StripTags'), 
          array('name' => 'StringTrim'), 
        ), 
        'validators' => array( 
          array(
            'name' => 'Between',
            'options' => array(
              // We can define the ranges of dates 
              // here, min and max are both optional, 
              // as long as one of them at least exists
              'min' => '1900-01-01', 
              'max' => '2013-01-01', 
            ),
          ),
        ), 
      )));

      // Set the property
      $this->inputFilter = $inputFilter;
    }

    // End of our method, just return our created input 
    // filter now
    return $this->inputFilter;
  } 
}
```

让我们立即开始，看看一个简单的例子，它使用我们的`normalform`，就像之前一样（`/module/Application/src/Application/Controller/IndexController.php`）：

```php
<?php

// Define the namespace of our controller
namespace Application\Controller;

// We need to use the following classes
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;
use Application\Form\NormalForm;
use Application\Form\NormalFormValidator;
use Application\Model\SampleModel;

// Set up our class definition
class IndexController extends AbstractActionController
{
  // We want to parse/display our form on the index
  public function indexAction()
  {
    // Initialize our form
    $form = new NormalForm(); 

    // Set our request in a local variable for easier
    // access
    $request = $this->getRequest();

    if ($request->isPost() === true) {
      // Create a new form validator
      $formValidator = new NormalFormValidator();

      // Set the input filter of the form to the form
      // validator
      $form->setInputFilter(
          $formValidator->getInputFilter()
      );

      // Set the data from the post to the form
      $form->setData($request->getPost());

      // Check with the form validator if the form is 
      // valid or not
      if ($form->isValid() === true) { 
        // Do some Model stuff, like saving, this is 
        // just an empty model we created to show what 
        // probably would happen after a validation 
        // success.
        $user = new SampleModel();

        // Get *only* the filtered data from the form
        $user->doStuff($form->getData());

        // Done with this, unset it
        unset($user);
      }
    }

    // Return the view model to the user
    return newViewModel(array(
        'form' => $form
    ));
  }
}
```

## 它是如何工作的…

让我们了解我们是如何实现我们所实现的。

### 设置基本表单

之前的第一例，创建一个从`Zend\Form`扩展的表单类，这是设置表单的最基本要求。正如我们所看到的，这个表单目前没有任何元素或属性设置，它唯一定义的是表单对象的 DOM 元素的`name/id`。在那之后我们所做的是首先初始化表单，然后将`ViewModel`分配给它，因为这将是要输出到屏幕的视图。

在例子中我们所做的唯一一件事是首先输出`<form>`标签——包括它的所有属性，如`method`、`action`等。第二件事是我们确实输出了表单中的所有元素（在这个例子中是没有的），最后我们输出表单结束标签`</form>`，这现在结束了表单声明。

如果我们打开浏览器查看我们的代码，我们将看到与之前没有太大的不同，可能是一个空页面。然而，当我们打开该页面的源代码（在 Firefox 中，这是在页面上右键单击并点击**查看页面源代码**）时，我们看到我们实际上已经正确地实例化了表单，在 HTML 中。

我们的基本表单实例化现在已经完成，如果我们想要一个更高级但同时也更吸引人的定义表单的方式，我们应该继续阅读下一部分。

### 设置带注释的表单

定义带注释的表单与普通表单略有不同，主要区别在于带注释的表单只是一个具有属性的类，它没有从任何其他类扩展，而另一种方法要求我们扩展`Zend\Form`类。在先前的例子中，我们首先使用注释方法创建了一个非常简单且为空的表单。我们还可以看到，我们需要一个 Hydrator 来让注释引擎理解我们在说什么，但我们不需要扩展类，所以我们可以在那里自由地做我们想做的事情。

我们唯一需要小心的是，我们表单中需要的每个元素，都应该将属性访问设置为 public，否则从技术上讲，注解引擎无法识别它。我们不需要为属性创建 getter/setter（除非我们想为自己使用它），因为注解引擎直接使用公共属性。

在控制器中使用表单与正常表单略有不同，因为如果我们只是实例化类并使用它作为表单，最终会出错。类需要首先通过`AnnotationBuilder`来实际构建表单。这就是为什么我们需要执行`createForm()`，然后输出一个表单。

这将不会输出任何可见的内容，但如果我们查看页面源代码（在 Firefox 中，这是通过右键单击页面然后点击**查看页面源代码**来实现的），我们会看到我们有一个新的表单开启标签`<form>`和一个表单结束标签`</form>`。在这些标签之间，你可以看到我们的表单，名为`annotationform`，现在被设置为表单的`name`和`id`。

一些开发者认为这种定义表单的方式有点过度，因为最终可能看起来我们没有增加很多可用性，这在所有公平的讨论中确实有点道理。这完全取决于具体情况，某种方法是否比其他方法更好，但公平地说，这是一种相当流畅的定义表单的方式！

### 向表单添加元素

如果我们以与之前相同的方式设置了表单，那么我们有两种定义元素到表单的方法。第一种将是定义表单的正常方法，它是对`Zend\Form\Form`的扩展，就像在*如何做...*部分中的表单示例一样，以及它的注解表单，就像在*如何做...*部分中的`AnnotationForm`第二个示例。

第一个示例假设我们在扩展自`Zend\Form\Form`的表单中定义了`__construct()`。它所做的就是调用`Zend\Form\Form`的`add()`方法，我们向该方法提供一个方法数组（是的，你同样可以在配置文件中创建整个表单！）。

添加一个元素就像那样简单。显然，还有更多的元素可供选择，它们都有自己的选项和属性，但我们不会深入讨论所有这些，因为讨论起来会非常冗长。

向注解表单添加元素既简单又复杂。说它简单是因为在最基本的概念中，它只需要你向类中添加一个属性，这已经足够简单了。但如果你想做得更多，添加验证或过滤器，你需要在属性上方添加注解注释。

正如我们在前面的例子中所看到的，通过注解定义元素的方式并不特别困难，只是我们需要知道使用哪个`@Annotation`。当设置属性/选项或有时其他注解时，我们会看到两个花括号`{}`，这代表 JavaScript 中的一个对象，并用于 JSON。

显然，这并不困难，但需要我们稍微改变一下思维方式。

### 表单、过滤和验证

一个从`Zend\Form\Form`扩展的普通表单通过查看表单的`$this->elements`来创建元素，其中所有表单元素都将被存储。一旦触发表单渲染器，所有这些元素都将装饰成真实的 HTML 标签。在注解表单中，将类转换为 HTML 的过程需要额外一步，简单来说就是将注解类转换成一个类似于`Zend\Form\Form`扩展类的框架。这样我们就可以像使用真实表单对象一样使用从注解类构建的表单。

当我们提交表单时（你不必一定指定一个 POST，因为它默认已经是`POST`了），我们让表单检查值是否正确，更重要的是，我们想要确保我们得到的是我们预期的值。

不仅从安全角度验证表单很重要，从过滤角度也很重要。如果我们对我们的元素应用多个过滤器（例如，字符串修剪和去除标签），我们希望它们都为我们准备好了，而不是在之后再次使用这些过滤器。显然，更大的问题是保护我们的应用程序免受恶意用户的侵害，并验证用户的输入。

正如我们在最后一个前面的代码示例中所看到的，我们首先创建表单，然后检查用户是否尝试提交表单。如果是这样，我们将设置我们为该表单创建的特定表单验证器。然后，我们将请求数据（这是用户填写我们的表单的内容）分配给表单。在将数据分配给表单后，我们调用`isValid()`来查看数据是否有效。如果是，我们使用`getData()`将过滤后的数据分配给我们的示例模型以保存它。

最后，我们将表单再次分配给视图，这样我们就可以显示在验证过程中发生的任何验证错误。很简单！

## 还有更多...

我们也可以仅通过配置来定义一个表单形式，这被称为通过工厂创建表单形式，我们鼓励您看看它是如何工作的，因为这同样是一种创建表单的极好方式。

为了增加表单的安全性，人们可能会考虑向我们的表单中添加一个 `Zend\Form\Element\Csrf` 元素，该元素检查表单的来源以确保没有执行跨站请求伪造（CSRF）。这是一个添加到表单中的唯一密钥，用于验证过程。我们甚至会进一步说，建议创建一个已经添加了 CSRF 元素的基础表单，这样我们就不必担心是否忘记了，只要我们扩展自基础表单即可。

# 使用表单视图助手

与 Zend Framework 1 的装饰器（在表单的创建和渲染中是一个关键）不同，我们现在知道在 Zend Framework 2 中，使用不同的视图助手和渲染器来渲染表单会更好。

## 如何做到这一点...

视图助手对于开发者来说是非常重要的工具，在这里我们将讨论如何在我们的代码中使用它们。

### Form

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// Just open and close the form tag
echo $this->form()->openTag();
echo $this->form()->closeTag();

// Use a form to pull the attributes from
echo $this->form()->openTag($formObject);

/** Do stuff in between **/

// Close the tag again with no form object attached
echo $this->form->closeTag();
```

这将渲染出以下内容：

```php
<form></form>
```

### FormButton

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// First we create a simple button (this is better done 
// inside a form/controller or model of course)
$buttonElement = new \Zend\Form\Element\Button(
  // This is the name of the button
  'somebutton'
);

// Render the button immediately through the button 
// element
echo $this->formButton($buttonElement);

// Render the button in 3 steps:
// Step 1, the opening tag: Can be called without a 
// parameter, and array of attributes or an instance of
// Zend\Form\Element
echo $this->formButton()->openTag($buttonElement);

// Step 2, the inner HTML: Output our custom inner HTML 
// here, like the label of the button
echo '<span>Life is short, click now!</span>';

// Step 3, the closing tag: Close the tag again.
echo $this->formButton()->closeTag();
```

如果我们现在查看渲染输出，它应该看起来像以下这样：

```php
<button name="somebutton"><span>Life is short, click now!</span></button>
```

### FormCaptcha

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php
$captchaElement = new \Zend\Form\Element\Captcha(array(
  // What is the name of the element
  'name' => 'captcha',
  // Now add some captcha specific configuration
  'captcha' => array(
    // The class is necessary for the factory to know 
    // what kind of captcha we want. The options are 
    // Dumb, Figlet, Image and the famous ReCaptcha
    'class' => 'Dumb',
  )
));

// That's all folks, the $captchaElement needs to be of 
// the instance Zend\Captcha\AdapterInterface to make it 
// work
echo $this->formCaptcha($captchaElement);
```

### FormCheckbox

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php
// Create a simple checkbox with the name someCheckbox
$checkboxElement = new \Zend\Form\Element\Checkbox('someCheckbox');

// The $checkboxElement needs to be of the instance 
// Zend\Form\Element\Checkbox to make it work
echo $this->formCheckbox($checkboxElement);
```

渲染输出可能如下所示：

```php
<input type="checkbox" name="someCheckbox" />
```

### FormCollection

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

$object = new \Zend\Form\Element\Collection(
  // The name of the collection
  'someCollection', 

  // Some additional options
  array(
    // The label we want to display
    'label' => 'collectionSample',

    // Should the collection create a template of our 
    // template element so that we easily duplicate it
    'should_create_template' => true,

    // Are we allowed to add new elements
    'allow_add' => true,

    // And how many elements do we want to render
    'count' => 2,

    // Define the target element to render
    'target_element' =>array(
      'type' => 'Zend\Form\Element\Text'
    ),
));

// The $object can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formCollection($object);
```

这将渲染出如下极其模糊的输出：

```php
<fieldset><legend>collectionSample</legend><span data-template="&lt;input type=&quot;text&quot; name=&quot;__index__&quot; value=&quot;&quot;&gt;"></span>
```

这对于集合了解它需要做什么已经足够了，就像在这个例子中，它持有我们的 `input` 字段的模板。

### FormColor

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// We want a simple text field for our color
$color = new \Zend\Form\Element\Color('someColor');

// The $color can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formColor($color);
```

### FormDate, FormDateTime, 和 FormDateTimeLocal

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php
// Create a date element
$date = new \Zend\Form\Element\Date('someDateElement');

// The $date can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formDate($date);
echo $this->formDateTime($date);
echo $this->formDateTimeLocal($date);
```

### FormEmail

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php
// Add a simple text field
$element = new \Zend\Form\Element\Text('someElement');

// The $email can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formEmail($email);
```

### FormFile

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// The $file can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formFile($file);
```

### FormHidden

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// The $hidden can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formHidden($hidden);
```

### FormImage

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// The $image can be of any class that implements the 
// Zend\Form\ElementInterface
$image->setAttrib('src', '/our/image.jpg');

echo $this->formImage($image);
```

### FormInput

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// The $input can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formInput($input);
```

### FormLabel

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php
// Create a simple text input
$element = new \Zend\Form\Element\Text('someElement');

// 1\. This will declare the label immediately. The
// $element can be of any class that implements
// the Zend\Form\ElementInterface

echo $this->formLabel($element);

// 2\. Or we can declare the formLabel like this
echo $this->formLabel()->openTag(array(
    'for' => 'someElement',
));

// We are putting some html in between the 
// <label></label> tags
echo "Some output in between!";
// Close the tag again
echo $this->formLabel()->closeTag();

// 3\. Or as a last method, there is still some other way 
// to define the element. This will prepend 
// $someOtherElement with our $element's label. Instead 
// of prepend we can also use append.
echo $this->formLabel(
    $element, 
    $someOtherElement, 
    'prepend'
);
```

### FormElementErrors

我们对名为 `example-viewscript.phtml` 的视图脚本进行了以下修改：

```php
<?php

// Create a simple text box
$element = new \Zend\Form\Element\Text('someInput');

// 1\. Just display the element errors, with the optional 
// attributes added as the second parameter.
// The $element can be of any class that implements the 
// Zend\Form\ElementInterface
echo $this->formElementErrors($element, array(
    'class' => 'element-error',
    'id' => 'error_three'
));

// 2\. Custom formatted validation error messages.
echo $this->formElementErrors()
          ->setMessageOpenFormat('<a href="/help-me">')
          ->setMessageSeparatorString(
                   '</a><a href="/help-me">'
         )->setMessageCloseString('</a>')
          ->render($element);
```

## 它是如何工作的...

表单元素视图助手是一种渲染表单元素的好方法。在 Zend Framework 的上一个版本中，这是通过表单装饰器完成的，这些装饰器与 ZF2 中的视图助手不同，因为它们是在表单到达视图脚本之前使用的。现在的工作方式是，当表单到达视图脚本时，它仍然处于其原始状态，这意味着我们可以完全操纵表单以符合我们的布局。这创建了一个更动态的输出，我们可以为每个视图脚本定义布局（这在 ZF1 中非常难以实现）。

由于表单元素视图助手负责在视图脚本中渲染元素，它们也可以更贴近开发者的需求。总的来说，这是一种创建外观和功能都出色的表单的绝佳方法。

可以使用各种视图助手和/或渲染器来创建完美的布局。有许多标准视图助手可以用来标记你的表单。

### Form

这个助手渲染你的 `<form />` 标签，如果需要，可以从我们的 `Zend\Form` 对象中提取一些属性作为属性使用。

表单助手（通过解析表单）支持的属性有 `accept-charset`、`action`、`autocomplete`、`enctype`、`method`、`name`、`novalidate` 和 `target`。

### FormButton

我们可以使用这个助手来渲染 `<button />` 标签，显然它可以根据我们的需求以不同的方式工作。它可以通过 `Zend\Form\Element` 来渲染按钮，或者以三步法来完成，其中我们可以在中间添加自己的内容。

`FormButton` 助手（通过解析 `Element`）支持的属性有 `name`、`autofocus`、`disabled`、`form`、`formaction`、`formenctype`、`formmethod`、`formnovalidate`、`formtarget`、`type` 和 `value`。

### FormCaptcha

`Captcha` 用于防止用户在不验证其为人类的情况下提交表单。偶尔，我们会收到被大量垃圾邮件填满的表单。这就是为什么我们现在有了这个小工具，它可以生成一个小图像，这是一个自动化的图灵测试，用来确定我们是否是人类。

这个助手只能通过 `Zend\Element\Captcha` 对象来渲染，所以在这方面没有太多可以进一步解释的。

### FormCheckbox

默认情况下，这个助手将渲染两个元素：

+   类型为 `checkbox` 的 `<input />` 元素

+   一个类型为 `hidden` 的 `<input />` 元素，其值为复选框状态

它创建隐藏输入，因为如果复选框未被选中，则不会提交，所以我们可以想象当元素不存在时表单验证的后果。这就是为什么总是在复选框元素之前渲染一个隐藏字段，以确保至少有某些内容被提交。

此外，复选框元素还有一些其他酷炫的选项，例如使用隐藏字段。对于那些有复选框经验的开发者来说，他们可以松一口气，因为未选中的复选框永远不会在表单中由浏览器提交。

因此，在复选框元素之前放置了一个隐藏字段，其名称与复选框元素相同，但填充了未选中的值。这意味着每当复选框未选中时，它将发送隐藏字段的值，否则复选框的选中值将覆盖它。

### FormCollection

这个辅助器在例如我们想要在一个实例中渲染一个完整的表单时使用。如果我们使用`Zend\Form`对象作为此辅助器的参数，我们将得到一个完全渲染的 HTML 表单返回。如果我们使用`Zend\Form\Element\Collection`，另一方面，我们将得到一个完全渲染的 HTML 集合返回，如果需要，还包括模板。

### FormColor

这是一个 HTML5 元素，它是一个具有类型 color 的`<input />`元素。它创建一个用户可以选择颜色的输入表单，或者当在非 HTML5 兼容的浏览器中使用时，它将简单地显示一个输入字段。

### FormDate、FormDateTime 和 FormDateTimeLocal

另一个输出具有类型`date`的`<input />`元素的 HTML5 元素是`FormDate`。在 HTML5 兼容的浏览器中，它通常会输出一个日历下拉菜单，用户可以选择他们喜欢的日期；在非兼容的浏览器中，它再次只显示一个文本输入字段。

### FormEmail

这个 HTML5 字段是一个很好的字段，它随 HTML5 兼容的浏览器一起提供，具有巧妙的验证功能，可以检查输入的值是否为实际的电子邮件地址。最好不要过于依赖它，我们仍然需要自行验证值，以防用户没有使用 HTML5 兼容的浏览器。

可以在`FormEmail`上设置的属性有`name`、`autocomplete`、`autofocus`、`disabled`、`form`、`list`、`maxlength`、`multiple`、`pattern`、`placeholder`、`readonly`、`required`、`size`、`type`和`value`。

### FormFile

`FormFile`辅助器对于显示具有类型 file 的`<input />`非常有帮助。它不仅显示了输入元素，还可以为任何我们想要监控的上传进度准备元素。像许多其他元素辅助器一样，此辅助器也支持属性：`name`、`accept`、`autofocus`、`disabled`、`form`、`multiple`、`required`、`type`和`value`。

### FormHidden

隐藏的`<input />`字段便于在不要求用户输入的情况下将信息发布到应用程序。这个辅助器没有什么特别之处，但它支持`name`、`disabled`、`form`、`type`和`value`属性。

### FormImage

`FormImage` `<input />`标签主要用于在表单中替代**提交**按钮。使用简单，只需`src`属性（图像的位置）。它还支持`name`、`alt`（推荐）、`autofocus`、`disabled`、`form`、`formaction`、`formenctype`、`formmethod`、`formnovalidate`、`formtarget`、`height`、`type`和`width`属性。

### FormInput

`FormInput` 是一个简单的 `<input />` 元素，通过自然选择类型为我们渲染元素。并不一定推荐使用这个，因为它相当通用，并且会有其缺陷（例如，当它不是一个必需的 `input` 标签时）。

### FormLabel

如果我们想显示一个 `<label />`，那么使用这个助手是完美的，因为我们可以声明标签的位置（`FormLabel::APPEND` 或 `FormLabel::PREPEND`），我们还可以添加标签的内容。它只支持 `for` 和 `form` 作为属性。

### FormElementErrors

此助手用于显示表单验证错误。默认情况下，这将在表单元素下方显示，但使用此助手我们可以更自定义地显示此错误。

# 创建自定义表单元素和表单视图助手

随着我们继续在 Zend Framework 2 中开发，并且我们的应用程序不断增长，就越有必要停止复制粘贴，只是用简单地输出我们想要的类的复制部分来替换所有这些重复的部分。在 ZF2 中，这可以通过视图助手轻松完成。

## 如何做到这一点...

在这个菜谱中，我们将创建我们自己的表单元素，以及相应的视图助手来显示它。

### 创建新元素

我们所需要做的只是设置元素的类型，然后就这样了。我们对 `/module/Application/src/Application/Form/Element/Video.php` 文件进行了以下修改，让我们看看代码应该是什么样子：

```php
<?php

// Set our namespace just right
namespace Application\Form\Element;

// We need to extend from the base element
use Zend\Form\Element;

// Set the class name, and make sure we extend from the 
// base element
class Video extends Element
{
  // The type of the element is video, 'nuff said.
  protected $attributes = array(
      'type' => 'video',
  );
}
```

如我们所见，这是一项相当容易的工作，我们现在已经成功创建了一个新元素，可以在 ZF2 中使用。

#### 创建新的视图助手

视图助手将创建我们刚刚声明的 HTML 元素，让我们看看视图助手在 `/module/Application/src/Application/Form/View/Helper/FormVideo.php` 文件中应该是什么样子：

```php
<?php

namespace Application\Form\View\Helper;

use Zend\Form\View\Helper\AbstractHelper;
use Zend\Form\ElementInterface;
use Zend\Form\Exception;

class FormVideo extends AbstractHelper
{
  /**
   * Attributes valid for the video tag
   *
   * @var array
   */
  protected $validTagAttributes = array(
    'autoplay' => true,
    'controls' => true,
    'height' => true,
    'loop' => true,
    'muted' => true,
    'poster' => true,
    'preload' => true,
    'src' => true,
    'width' => true,
  );
```

首先，我们添加了此元素可以拥有的属性，这是为了确保我们不会声明不存在的属性（尽管在大多数情况下这不会造成太大的问题）。

```php
  /**
   * Invoke helper as functor
   *
   * Proxies to {@link render()}.
   *
   * @param ElementInterface|null $element
   * @return string|FormInput
   */
  public function __invoke(ElementInterface $element = null)
  {
    if (!$element) {
      return $this;
    }

    return $this->render($element);
  }
```

创建了前面的 `__invoke` 方法，这样我们就不必在我们想要调用视图助手之前初始化类。这样我们就可以通过使用 `formVideo()` 来在视图脚本中使用它，而不是首先实例化一个新的 `FormVideo()`。

```php
  /**
   * Creates the <source> element for use in the <video>
   * element.
   * 
   * @param array|string $src	Can either be an 
   *                           array of strings, or a 
   *                           string alone.
   * @return string
   */
  protected function createSourcesString($src) 
  {
    $retval = '';

    if (is_array($src) === true) {
      foreach ($src as $tmpSrc) {
        $retval .= $this->createSourcesString($tmpSrc);
      }
    } else {
     $retval = sprintf(
       '<source src="img/%s">',
       $src
     );
    }

    return $retval;
  }
```

`createSourcesString` 方法获取包含所有视频 URL 的字符串或数组。如前所述，这可以是字符串或数组，在后一种情况下，它将遍历数组并输出带有源标签的字符串。

```php
  /**
   * Render a form <video /> element from the provided 
   * $element
   *
   * @param ElementInterface $element
   * @throws Exception\DomainException
   * @return string
   */
  public function render(ElementInterface $element)
  {
    // Get the src attribute of the element
    $src = $element->getAttribute('src');

    // Check if the src is null or empty, in that case 
    // throw an error as we can 't play a video without 
    // a video link!
    if ($src === null || $src === '') {
      throw new Exception\DomainException(sprintf(
        '%s requires that the element has an assigned'.   
        'src; none discovered',
        __METHOD__
      ));
    }

    // Get the attributes from the element
    $attributes = $element->getAttributes();

    // Unset the src as we don't need it right here as 
    // we render it separately
    unset($attributes['src']);

    // Return our rendered object
    return sprintf(
        '<video %s>%s</video>',
        $this->createAttributesString($attributes),
        $this->createSourcesString($src)
    );
  }
}
```

### 将视图助手添加到配置中

现在我们需要将视图助手添加到模块配置中，以确保视图助手可以在视图脚本中找到。我们可以简单地通过在我们的 `/module/Application/Module.php` 中添加另一个方法来实现，如下面的代码所示：

```php
class Module 
{
  public function getViewHelperConfig()   
  {
    return array(
        'invokables' => array(
        // Add our extra view helper to render our video
        'formVideo' => 'Application\Form\View\Helper\FormVideo',
      )
    );
  }
}
```

我们没有把整个类放进去，因为这会对这个例子来说太多无用的信息。然而，想法是，我们可以简单地把这个方法放在我们的 `Module.php` 中，以确保我们的视图助手会被定位。

### 显示新的元素

我们对 `/module/Application/view/application/index/video.phtml` 文件做了以下修改：

```php
<?php
use Application\Form\Element\Video;

// Declare a new video element
$video = new Video();

// Set the attribute src for this element
$video->setAttribute('src', array(
// These are some public video urls from 
// w3schools.com
  'http://www.w3schools.com/html/mov_bbb.mp4',
  'http://www.w3schools.com/html/mov_bbb.ogg',
  ));

// We also want to begin auto playing once loaded
$video->setAttribute('autoplay', true);

// Output the formatted element
echo $this->formVideo($video);
```

现在我们已经创建了一个新的表单元素和一个新的表单视图助手！

## 它是如何工作的...

### 创建元素

首先，我们需要在 ZF2 中创建新的元素，然后再使用它。这可以通过扩展 `Zend\Form\Element` 的基本元素来轻松完成。

接下来是视图助手，因为我们想确保我们的元素也能正确地渲染给用户。由于我们的元素不是任何现有类型（否则这将是一个非常无聊的食谱），我们需要确保我们为自己创建一个视图助手。

我们代码的最后一部分是创建实际的渲染方法，正如其名称所表明的，它渲染实际的 HTML 对象。

在我们的案例中，我们希望在 `src` 未定义时触发一个异常，因为没有它，这个 HTML 元素将非常无用。现在，我们已经设置好了一切，我们可以使用这个元素在表单中，或者在其自身的视图脚本中单独使用。在上一个例子中，我们只是在视图脚本中声明了表单元素来展示它如何工作；然而，在视图脚本中使用逻辑并不是一个建议的做法，因为我们希望保持视图尽可能干净，并且只输出与之相关的代码。任何与 HTML 或用户输出相关的内容都应该放在控制器或模型中。

### 我们做了什么

我们所做的是创建了一个新的表单元素，它原本应该是一个 `<video />` 标签，一个全新的 HTML5 元素。这个视频标签可以拥有几个属性，其中之一就是 `src`。在这个例子中，`src` 属性告诉视频元素我们可以在哪里找到我们想要播放的视频。

创建我们自己的视图助手的一个好理由是，如果我们有一段 HTML 代码在我们的应用程序中反复出现（比如工具提示或帮助文本），并且只需要复制粘贴并更改一些属性就可以使用。为了节省我们的时间和空间（从代码和可读性的角度来看），我们会将其转换成一个简单的视图助手类，它复制了确切的对象，我们可以通过添加选项来转换这个对象。

最后，我们在视图脚本中简单地使用 `formVideo` 视图助手来实际渲染对象，这样就可以减轻我们的负担，因为它渲染了一段易于复制的代码。
