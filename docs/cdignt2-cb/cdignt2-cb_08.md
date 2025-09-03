# 第八章. 日历、正确的时间和地点

在本章中，我们将涵盖：

+   使用数据库结果构建 CodeIgniter 日历助手

+   使用日历库构建约会管理器

+   创建一个用于处理个人出生日期的助手

+   在 CodeIgniter 中处理模糊日期

# 简介

CodeIgniter 附带了许多函数和助手，以帮助支持您在处理时间、日期、日历等时的应用程序。在本章中，我们将使用其中的一些，但我们还将创建一些自己的助手，这些助手在日常任务中非常有用，例如计算一个人的年龄（对于年龄验证脚本很有用）和处理模糊日期（即，编写日期或时间的描述而不是只写出准确的日期）。

# 使用数据库结果构建 CodeIgniter 日历助手

CodeIgniter 附带了一个非常有用的日历助手，允许您以网格形式显示月份。您可以开发从数据库（例如存储日记约会的表）中提取事件的功能，并告知用户在特定一天是否有约会。

## 准备工作

由于我们在数据库中存储约会，我们需要一个数据库表。将以下代码复制到您的数据库中：

```php
CREATE TABLE `appointments` (
  `app_id` int(11) NOT NULL AUTO_INCREMENT,
  `app_date` varchar(11) NOT NULL,
  `app_url` varchar(255) NOT NULL,
  `app_name` varchar(255) NOT NULL,
  PRIMARY KEY (`app_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=5 ;

INSERT INTO `appointments` (`app_id`, `app_date`, `app_url`, `app_name`) VALUES
(1, '1375465528', 'http://localhost/1', 'My Appointment'),
(2, '1375638327', 'http://localhost/2', 'My Second Appointment'),
(3, '1375897527', 'http://localhost/3', 'My Third Appointment'),
(4, '1381167927', 'http://localhost/4', 'My Forth Name');
```

## 如何做到这一点…

我们将创建两个文件：

+   `/path/to/codeigniter/application/controllers/app_cal.php`

+   `/path/to/codeigniter/application/models/app_cal_model.php`

+   `/path/to/codeigniter/application/views/app_cal/view.php`

为了创建这两个文件，我们执行以下步骤：

1.  创建文件 `/path/to/codeigniter/application/controllers/app_cal.php` 并添加以下代码：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class App_cal extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->helper('url');
            $this->load->helper('date');
        }

      public function index() {
            redirect('app_cal/show');
        }

        public function show() {
            $prefs = array (
               'start_day'    => 'monday',
               'month_type'   => 'long',
               'day_type'     => 'short',
               'show_next_prev'  => TRUE,
               'next_prev_url'   => 'http://www.your_domain.com/app_cal/show/'
             );

            $this->load->library('calendar', $prefs);

            if ($this->uri->segment(4)) {
                $year= $this->uri->segment(3);
                $month = $this->uri->segment(4);
            } else {
                $year = date("Y", time());
                $month = date("m", time());
            }

            $this->load->model('App_cal_model');
            $appointments = $this->App_cal_model->get_appointments($year, $month);
            $data = array();

            foreach ($appointments->result() as $row) {
                $data[(int)date("d",$row->app_date)] = $row->app_url;
            }

            $data['cal_data'] = $this->calendar->generate($year, $month, $data);

            $this->load->view('app_cal/view', $data);
        }        
    }
    ```

1.  创建文件 `/path/to/codeigniter/application/models/app_cal_model.php` 并添加以下代码：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class App_cal_model extends CI_Model {
        function __construct() {
            parent::__construct();
        }

        function get_appointments($year, $month) {
            $month_as_written = array(
                '01' => 'January',
                '02' => 'February',
                '03' => 'March',
                '04' => 'April',
                '05' => 'May',
                '06' => 'June',
                '07' => 'July',
                '08' => 'August',
                '09' => 'September',
                '10' => 'October',
                '11' => 'November',
                '12' => 'December'
            );

            $start_date = '01' . ' ' . $month_as_written[$month] . ' ' . $year;
            $start_of_month = strtotime($start_date);

            $end_date = days_in_month($month, $year) . ' ' . $month_as_written[$month] . ' ' . $year;
            $end_of_month = strtotime($end_date);

            $this->db->where('app_date > ', $start_of_month);
            $this->db->where('app_date < ', $end_of_month);
            $query = $this->db->get('appointments');

            return $query;
        }
    }
    ```

1.  创建文件 `/path/to/codeigniter/application/views/app_cal/view.php` 并添加以下代码：

    ```php
    <?php echo $cal_data ; ?>
    ```

## 它是如何工作的…

我们首先在我们的网页浏览器中加载控制器 `app_cal`（如果你想知道，它代表约会日历），在构造函数中加载助手 `'url'` 和 `'date'`：

```php
        $this->load->helper('url');
        $this->load->helper('date');
```

加载的第一个函数是 `index()`，它将我们重定向到 `show()` 函数，在那里我们立即开始定义日历功能的一些偏好：

```php
        $prefs = array (
           'start_day'    => 'monday',
           'month_type'   => 'long',
           'day_type'     => 'short',
           'show_next_prev'  => TRUE,
           'next_prev_url'   => 'http://www.your_domain.com/app_cal/show/'
         );
```

每个项目相当直观，但我仍然会详细介绍它们：

| Preference | 描述 |
| --- | --- |
| start_day | 指定日历网格中最左侧是星期几，所以如果你输入 'sunday'，日历网格中的日行（描述日子的行）将从星期日开始。如果你出于某种奇怪的原因想让你的日历从星期三开始，你将输入 'wednesday'，日历周将从星期三开始。但请真的不要这样做；这看起来会很奇怪！ |
| month_type | 指定月份的书写方式。'long' 是完整的月份名称，例如 August，而 'short' 是缩写版本，例如 Aug。 |
| day_type | 指定日历网格中日期行的星期几是如何书写的。'long' 是 Monday、Tuesday、Wednesday 等等，而 'short' 是 Mon、Tue、Wed 等等。 |
| show_next_prev | 可以是 TRUE 或 FALSE。这会让 CodeIgniter 知道它是否应该显示下一个和上一个的箭头（ << 和 >>）；点击这些箭头将使日历向前或向后移动一个月。如果设置为 TRUE（在这个菜谱中就是这样），你需要指定 next_prev_url。 |
| next_prev_url | 指定 CodeIgniter 应该使用的 << 或 >> 链接的 URL 代码。 |

接下来，我们加载日历库，并将 `$prefs` 数组作为第二个参数传递给它。然后，我们检查是否存在第四个 `uri` 段：

```php
if ($this->uri->segment(4)) {
$year= $this->uri->segment(3);
$month = $this->uri->segment(4);
} else {
$year = date("Y", time());
$month = date("m", time());
}
```

第三个和第四个 `uri` 段分别是年份（YYYY）和月份（MM），如果它们不存在，可能是因为第一次加载日历（或者日历不是通过 `'next_prev_url'` 访问的）。无论如何，因为我们没有第三个或第四个 `uri` 段传递给我们的模型，我们不得不自己创建它们。但我们应该使用什么？比如当前月份和当前年份（参见前面的高亮代码）？

现在，我们加载模型 `App_model` 并将我们的 `$year` 和 `$month` 变量传递给它：

```php
$this->load->model('App_cal_model');
$appointments = $this->App_cal_model->get_appointments($year, $month);
```

现在我们来看看模型，看看发生了什么。我们使用时间戳在数据库中存储我们的预约，因为年份和月份被作为字符串 'YYYY' 和 'MM' 传递给 `app_cal` 控制器，所以我们需要将 'YYYY'、'MM' 字符串转换为时间戳，以便我们可以查询数据库并确定在所选月份的特定一天是否有预约。这意味着我们需要使用 PHP 函数 `strtotime`。

对于熟悉该函数（或者甚至只是阅读了函数名称）的人来说，会明白 `strtotime` 将写成的英文字符串转换为 Unix 时间戳，例如，写入 "last Wednesday"，`strtotime` 将返回上一次周三的时间戳。这是一个获取时间戳的好方法，但它确实意味着你需要为要计算的日期生成某种类型的字符串描述。

我们想要获取特定月份的所有预约，这意味着生成一个带有 WHERE 子句的数据库查询，查找 "预约大于代表一个月第一天的日期时间戳，并且小于代表一个月最后一天的日期时间戳"。因此，为了准备这个，让我们看看代码中的以下 `$month_as_written` 数组：

```php
$month_as_written = array(
'01' => 'January',
'02' => 'February',
'03' => 'March',
'04' => 'April',
'05' => 'May',
'06' => 'June',
'07' => 'July',
'08' => 'August',
'09' => 'September',
'10' => 'October',
'11' => 'November',
'12' => 'December'
);
```

你会看到数组中每个项目的键都与 `$month (MM)` 的格式相匹配。这很重要，因为我们将使用 `$month` 中的值来用英文写出所需的月份名称。

我们将用 `'01 '` 预先添加，以表示月初，然后用 `$year` 后缀，如下所示：

```php
$start_date = '01' . ' ' . $month_as_written[$month] . ' ' . $year;
$start_of_month = strtotime($start_date);
```

写入的字符串存储在变量 `$start_date` 中，然后传递给 `strtotime()`，它反过来返回月份开始的 Unix 时间戳。接下来，我们计算结束日期：

```php
$end_date = days_in_month($month, $year) . ' ' . $month_as_written[$month] . ' ' . $year;
$end_of_month = strtotime($end_date);
```

接下来，我们使用 CodeIgniter 函数 `days_in_month()`；传入 `$month` 和 `$year` 参数，它将返回该月的天数作为一个整数。然后我们将这个值与一个空格 `' '` 和 `$month_as_written` 数组中的月份名称连接起来，最后以 `$year` 结尾。然后将这个字符串传递给 `strtotime($end_date)`，它给出了月份结束的 Unix 时间戳；这个值存储在变量 `$end_of_month` 中。

我们将在数据库查询中使用两个变量 `$start_of_month` 和 `$end_of_month`，要求它返回计算月份开始之后的预约，但不超过月份结束：

```php
$this->db->where('app_date > ', $start_of_month);
$this->db->where('app_date < ', $end_of_month);
$query = $this->db->get('appointments');

return $query;
```

接下来，我们需要构建一个数组来存储预约和 URL。首先，让我们声明这个数组：

```php
$data = array();
```

现在，我们将遍历 `App_model` 结果（包含在变量 `$appointments` 中），在遍历过程中构建数组：

```php
foreach ($appointments->result() as $row) {
$data[(int)date("d",$row->app_date)] = $row->app_url;
}
```

完成后，数组应该具有以下结构：

```php
array(3) {
  [2]=>
  string(18) "http://localhost/1"
  [4]=>
  string(18) "http://localhost/2"
  [7]=>
  string(18) "http://localhost/3"
}
```

`$data` 数组与 `$year` 和 `$month` 一起传递给日历库函数 `generate()`：

```php
$data['cal_data'] = $this->calendar->generate($year, $month, $data);

$this->load->view('app_cal/view', $data);
```

这个结果存储在 `$data['cal_data']` 中，然后传递给视图 `app_cal/view`，从那里渲染到屏幕上。

# 使用日历库构建预约管理器

之前的食谱使用了 CodeIgniter 日历库来帮助构建交互式日历。然而，您只能查看数据库中已经存在的日历项目。下一步的逻辑是构建一个小型应用程序，允许您通过表单创建日历项目；一个简单的预约管理器就可以做到这一点。我们基于之前的食谱；然而，您不需要回到并完成那个食谱。您需要的所有内容都包含在这个食谱中。

## 准备工作

我们需要创建一个数据库表来存储我们的预约。如果您已经使用了之前的食谱，您应该已经有了数据库表；如果是这样，请在您的数据库中运行以下代码：

```php
ALTER TABLE  `appointments` ADD  `app_description` VARCHAR( 255 ) NOT NULL AFTER  `app_name`
```

或者，如果您还没有创建表格，请在您的数据库中运行以下代码：

```php
CREATE TABLE `appointments` (
  `app_id` int(11) NOT NULL AUTO_INCREMENT,
  `app_date` varchar(11) NOT NULL,
  `app_url` varchar(255) NOT NULL,
  `app_name` varchar(255) NOT NULL,
  `app_description` varchar(255) NOT NULL,
  PRIMARY KEY (`app_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;
```

现在数据库已经排序，让我们看看代码：

## 如何做到这一点…

我们将要创建以下六个文件：

+   `/path/to/codeigniter/application/controllers/app_cal.php`：这个文件包含运行显示所需的所有代码，包括日历的 HTML 6 模板

+   `/path/to/codeigniter/application/models/app_cal_model.php`：这个文件包含与数据库交互所需的所有代码

+   `/path/to/codeigniter/application/views/app_cal/view.php`：这个页面将显示日历

+   `/path/to/codeigniter/application/views/app_cal/appointment.php`：这个页面将显示一个表单，您可以在其中添加预约

+   `/path/to/codeigniter/application/views/app_cal/new.php`：这个页面显示一个表单，允许用户创建新的预约

+   `/path/to/codeigniter/application/views/app_cal/delete.php`：此文件显示删除确认消息

我们需要执行以下步骤来创建这些文件：

1.  创建文件 `/path/to/codeigniter/application/controllers/app_cal.php` 并向其中添加以下代码（这是一个相当大的控制器，所以我将分解较大的函数来解释它们的功能）：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class App_cal extends CI_Controller {
        function __construct() {
            parent::__construct();
            $this->load->helper('url');
            $this->load->helper('date');
            $this->load->helper('form');
            $this->load->model('App_cal_model');
        }

      public function index() {
            redirect('app_cal/show');
        }
    ```

    公共函数 `show()` 将在屏幕上显示日历；它负责决定显示哪个月份和哪一年份。

    ```php
        public function show() {
            if ($this->uri->segment(4)) {
                $year= $this->uri->segment(3);
                $month = $this->uri->segment(4);
            } else {
                $year = date("Y", time());
                $month = date("m", time());
            }

            $tpl = '
               {table_open}<table border="1" cellpadding="15" cellspacing="1">{/table_open}

               {heading_row_start}<tr>{/heading_row_start}

               {heading_previous_cell}<th><a href="{previous_url}">&lt;&lt;</a></th>{/heading_previous_cell}
               {heading_title_cell}<th colspan="{colspan}">{heading}</th>{/heading_title_cell}
               {heading_next_cell}<th><a href="{next_url}">&gt;&gt;</a></th>{/heading_next_cell}

               {heading_row_end}</tr>{/heading_row_end}

               {week_row_start}<tr>{/week_row_start}
               {week_day_cell}<td>{week_day}</td>{/week_day_cell}
               {week_row_end}</tr>{/week_row_end}

               {cal_row_start}<tr>{/cal_row_start}
               {cal_cell_start}<td>{/cal_cell_start}

               {cal_cell_content}'.anchor('app_cal/create/'.$year.'/'.$month.'/{day}', '+').' <a href="{content}">{day}</a>{/cal_cell_content}
               {cal_cell_content_today}<div class="highlight">'.anchor('app_cal/create/'.$year.'/'.$month.'/{day}', '+').'<a href="{content}">{day}</a></div>{/cal_cell_content_today}

               {cal_cell_no_content}'.anchor('app_cal/create/'.$year.'/'.$month.'/{day}', '+').' {day}{/cal_cell_no_content}
               {cal_cell_no_content_today}<div class="highlight">'.anchor('app_cal/create/'.$year.'/'.$month.'/{day}', '+').'{day}</div>{/cal_cell_no_content_today}

               {cal_cell_blank}&nbsp;{/cal_cell_blank}

               {cal_cell_end}</td>{/cal_cell_end}
               {cal_row_end}</tr>{/cal_row_end}

               {table_close}</table>{/table_close}' ;

            $prefs = array (
                'start_day'         => 'monday',
                'month_type'        => 'long',
                'day_type'          => 'short',
                'show_next_prev'    => TRUE,
                'next_prev_url'     => 'http://www.your_domain.com/app_cal/show/',
                'template'          => $tpl         
             );

            $this->load->library('calendar', $prefs);

            $appointments = $this->App_cal_model->get_appointments($year, $month);
            $data = array();

            foreach ($appointments->result() as $row) {
                $data[(int)date("d",$row->app_date)] = $row->app_url;
            }

            $data['cal_data'] = $this->calendar->generate($year, $month, $data);

            $this->load->view('app_cal/view', $data);
        }        
    ```

    公共函数 `create()` 将处理预约的创建，因此它将显示预约表单，验证输入，并将数据发送到模型以将其插入数据库。

    ```php
        public function create() {
            $this->load->library('form_validation');
            $this->form_validation->set_error_delimiters('', '<br />');

            $this->form_validation->set_rules('app_name',  'Appointment Name', 'required|min_length[1]|max_length[255]|trim');
            $this->form_validation->set_rules('app_description',  'Appointment Description', 'min_length[1]|max_length[255]|trim');
            $this->form_validation->set_rules('day',  'Appointment Start Day', 'required|min_length[1]|max_length[11]|trim');
            $this->form_validation->set_rules('month',  'Appointment Start Month', 'required|min_length[1]|max_length[11]|trim');
            $this->form_validation->set_rules('year',  'Appointment Start Year', 'required|min_length[1]|max_length[11]|trim');

            if ($this->uri->segment(3)) {
                $year   = $this->uri->segment(3);
                $month  = $this->uri->segment(4);
                $day    = $this->uri->segment(5);
            } elseif ($this->input->post()) {
                $year   = $this->input->post('year');
                $month  = $this->input->post('month');
                $day    = $this->input->post('day');
            } else {
                $year   = date("Y", time());
                $month  = date("m", time());
                $day    = date("j", time());
            }

            if ($this->form_validation->run() == FALSE) { // First load, or problem with form
                $data['app_name']           = array('name' => 'app_name', 'id' => 'app_name', 'value' => set_value('app_name', ''), 'maxlength'   => '100', 'size' => '35');
                $data['app_description']    = array('name' => 'app_description', 'id' => 'app_description', 'value' => set_value('app_description', ''), 'maxlength' => '100', 'size' => '35');

                $days_in_this_month = days_in_month($month,$year);

                $days_i = array();
                for ($i=1;$i<=$days_in_this_month;$i++) {
                    ($i<10 ? $days_i['0'.$i] = '0'.$i : $days_i[$i] = $i) ;
                }

                $data['days']   = $days_i;
                $data['months'] = array('01' => 'January','02' => 'February','03' => 'March','04' => 'April','05' => 'May','06' => 'June','07' => 'July','08' => 'August','09' => 'September','10' => 'October','11' => 'November','12' => 'December');
                $data['years']  = array('2013' => '2013');
                $data['day']    = $day;
                $data['month']  = $month;
                $data['year']   = $year;
                $this->load->view('app_cal/new', $data);
            } else {
                $app_date = mktime(0,0,0,$month,$day,$year);

                $data = array(
                    'app_name'          => $this->input->post('app_name'),
                    'app_description'   => $this->input->post('app_description'),
                    'app_date'          => $app_date,
                    'app_url'           => base_url('index.php/app_cal/appointment/'.$year.'/'.$month.'/'.$day)
                    );

                if ($this->App_cal_model->create($data)) {
                    redirect('app_cal/show/'.$year.'/'.$month);
                } else {
                    redirect('app_cal/index');
                }  
            }      
        }
    ```

    公共函数 `delete()` 负责删除（删除）一个预约。它将加载删除确认表单，验证输入，并将数据传递到模型以从数据库中删除。

    ```php
            public function delete() {
            $this->load->library('form_validation');
            $this->form_validation->set_error_delimiters('', '<br />');

            if ($this->input->post('app_id')) {
                $id = $this->input->post('app_id');
            } else {
                $id = $this->uri->segment(3);
            }        

            $this->form_validation->set_rules('app_id',  'Appointment ID', 'min_length[1]|max_length[11]|is_natural|trim');

            if ($this->form_validation->run() == FALSE) { // First load, or problem with form
                $appointment = $this->App_cal_model->get_single($id);
                $data['id'] = $id;

                foreach ($appointment->result() as $row) {
                    $data['app_name'] = $row->app_name;
                    $data['app_date'] = $row->app_date;
                }

                $this->load->view('app_cal/delete', $data);
            } else {
                if ($this->App_cal_model->delete($id)) {
                    redirect('app_cal/index');
                } else {
                    redirect('app_cal/index');
                }
            }
        }    

        public function appointment() {
            if ($this->uri->segment(3)) {
                $year   = $this->uri->segment(3);
                $month  = $this->uri->segment(4);
                $day    = $this->uri->segment(5);

                $data['appointments'] = $this->App_cal_model->get_appointment($year, $month, $day);
                $this->load->view('app_cal/appointment', $data);
            } else {

            }
        }
    }
    ```

1.  创建文件 `/path/to/codeigniter/application/models/app_cal_model.php` 并向其中添加以下代码：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class App_cal_model extends CI_Model {

        function __construct() {
            parent::__construct();
        }

        function get_appointments($year, $month) {
            $month_as_written = array(
                '01' => 'January',
                '02' => 'February',
                '03' => 'March',
                '04' => 'April',
                '05' => 'May',
                '06' => 'June',
                '07' => 'July',
                '08' => 'August',
                '09' => 'September',
                '10' => 'October',
                '11' => 'November',
                '12' => 'December'
            );

            $sd = '01' . ' ' . $month_as_written[$month] . ' ' . $year;
            $start_of_month = strtotime($sd);

            $ed = days_in_month($month, $year) . ' ' . $month_as_written[$month] . ' ' . $year;
            $end_of_month = strtotime($ed);

            $this->db->where('app_date > ', $start_of_month);
            $this->db->where('app_date < ', $end_of_month);
            $query = $this->db->get('appointments');
            $this->db->last_query();

            return $query;
        }

        function get_appointment($year, $month, $day) {
            $start_of_day = mktime(0,0,0,$month,$day,$year);
            $end_of_day = $start_of_day + 86400;
            $this->db->where('app_date >= ', $start_of_day);
            $this->db->where('app_date <= ', $end_of_day);
            $query = $this->db->get('appointments');

            return $query;
        }

        function create($data) {
            if ($this->db->insert('appointments', $data)) {
                return true;
            } else {
                return false;
            }
        }

        function delete($id) {
            $this->db->where('app_id', $id);
            if ($this->db->delete('appointments')) {
                return true;
            } else {
                return false;
            }
        }

        function get_single($id) {
            $this->db->where('app_id', $id);
            $query = $this->db->get('appointments');
            return $query;
        }
    }
    ```

1.  创建文件 `/path/to/codeigniter/application/views/app_cal/view.php` 并向其中添加以下代码：

    ```php
    <?php echo anchor ('app_cal/create', 'New Appointment') ; ?>
    <?php echo $cal_data ; ?>
    ```

1.  创建文件 `/path/to/codeigniter/application/views/app_cal/appointment.php` 并向其中添加以下代码：

    ```php
    <?php echo anchor ('app_cal/index', 'View Calendar') ; ?>
    <a href=""></a><h2>Appointments</h2>

    <?php foreach ($appointments->result() as $row) : ?>
    	<?php echo anchor('app_cal/delete/'.$row->app_id, 'Delete') ; ?><br />
    	<?php echo date("j-m-Y",$row->app_date) ; ?><br />
    	<?php echo $row->app_name ; ?><br />
    	<?php echo $row->app_description ; ?>
    	<hr>
    <?php endforeach ; ?>
    ```

1.  创建文件 `/path/to/codeigniter/application/views/app_cal/delete.php` 并向其中添加以下代码：

    ```php
    <h2>Delete Appointment</h2>

    <?php if (validation_errors()) : ?>
        <p><?php echo validation_errors() ;?></p>
    <?php endif ; ?>

    <?php echo form_open('app_cal/delete') ; ?>
    <h4>Are you sure you want to delete the following appointment?</h4>

    <?php echo $app_name . ' on ' . date("d-m-Y h:i:s", $app_date); ?>

    <?php echo form_hidden('app_id', $id) ; ?>

    <br /><br />

    <input type="submit" value="Delete" />
    or <?php echo anchor ('app_cal', 'Cancel') ; ?>
    <?php echo form_close() ; ?>
    ```

1.  创建文件 `/path/to/codeigniter/application/views/app_cal/new.php` 并向其中添加以下代码：

    ```php
    <h2>New Appointment</h2>
    <h4>Appointment Name</h4>

    <?php if (validation_errors()) : ?>
        <p><?php echo validation_errors() ;?></p>
    <?php endif ; ?>

    <?php echo form_open('app_cal/create') ; ?>
    <?php echo form_input($app_name); ?>
    <h4>Appointment Description</h4>
    <?php echo form_input($app_description); ?>
    <h4>Appointment Date</h4>
    <?php echo form_dropdown('day', $days, $day); ?>
    <?php echo form_dropdown('month', $months, $month); ?>
    <?php echo form_dropdown('year', $years, $year); ?>

    <br /><br />

    <input type="submit" value="Save" />
    or <?php echo anchor ('app_cal', 'Cancel') ; ?>
    <?php echo form_close() ; ?>
    ```

## 它是如何工作的...

大部分功能与之前的菜谱类似，但有一些差异；我们添加了管理预约的支持。因此，让我们首先看看 `public function view()`。你会注意到我们已经移动了一些代码；从 uri 中获取日期的代码或即时生成日期的代码现在在 `$prefs` 数组之前——这是由于 `$tpl` 变量的原因。那么 `$tpl` 是什么呢？

`$tpl` 变量字符串的内容，更具体地说，它是日历库使用的日历模板；模板支持用于天数的标签——`{day}`——但不支持月份或年份。这意味着我们必须手动将这些值插入模板中。但为了做到这一点，我们需要事先知道年份和月份的值；这就是为什么计算月份和日期的代码现在被移动到 `$prefs` 数组之前。我使用的模板代码是 CodeIgniter 网站用户指南中可用的修改版本：[`ellislab.com/codeigniter/user-guide/libraries/calendar.html`](http://ellislab.com/codeigniter/user-guide/libraries/calendar.html)。

从这里开始，其功能与之前菜谱中的 `view()` 函数相同：我们加载日历库，获取当前月份的所有预约，并将其传递给 `app_cal/view.php` 视图文件。让我们更详细地了解一下一些新函数：

### 公共函数 `create()`

当用户输入新预约的详细信息并提交创建表单时，公共函数`create()`首先声明新预约的验证规则。然后我们需要从`post`或`get`数组中获取特定预约的年、月和日。公共`create()`函数会检查这一点并将日期值存储在变量中：

```php
if ($this->uri->segment(3)) {
   $year   = $this->uri->segment(3);
   $month  = $this->uri->segment(4);
   $day    = $this->uri->segment(5);
} elseif ($this->input->post()) {
   $year   = $this->input->post('year');
   $month  = $this->input->post('month');
   $day    = $this->input->post('day');
} else {
   $year   = date("Y", time());
   $month  = date("m", time());
   $day    = date("j", time());
}
```

这里有三个测试，因为公共函数`create()`可以通过不同的方式访问。第一个测试是查看是否有人通过点击日历网格中的添加预约链接**+**来访问页面，第二个测试是查看页面是否已提交，第三个（else）是当点击**新预约**链接时。

接下来，我们检查页面验证是否通过；`FALSE`可能意味着验证失败，或者`create()`是第一次被访问。

我们为 CodeIgniter 设置一些表单值以在视图中渲染，并开始构建日期下拉菜单所需的变量：

```php
$days_in_this_month = days_in_month($month,$year);
$days_i = array();
for ($i=1;$i<=$days_in_this_month;$i++) {
   ($i<10 ? $days_i['0'.$i] = '0'.$i : $days_i[$i] = $i) ;
}
```

我们随后展示`view`文件并等待用户提交。在提交成功（即，当表单通过验证时），我们计算预约日期变量（日、月和年）的 Unix 时间戳，并将所有内容打包到`$data`数组中，以便插入数据库。成功插入后，将用户重定向到他们的预约所在的月份和年份。

### 公共函数`delete()`

这相当简单；我们检查`get`或`post`数组中是否存在预约 ID：

```php
if ($this->input->post('app_id')) {
$id = $this->input->post('app_id');
} else {
$id = $this->uri->segment(3);
}
```

我们在`get`和`post`中都查找，因为函数可以由第一次点击 URL 的人访问，也可以由第二次点击（当他们点击视图中的**确认删除**按钮时）的人提交。

如果表单提交时出现错误，或者第一次运行，将从数据库中获取预约详情（前一段代码已传递`$id`）。

```php
$appointment = $this->App_cal_model->get_single($id);

foreach ($appointment->result() as $row) {
$data['app_name'] = $row->app_name;
$data['app_date'] = $row->app_date;
}

$this->load->view('app_cal/delete', $data);
```

我们随后从数据库结果中获取`app_name`和`app_date`并将它们作为项目存储在`$data`数组中，以便传递给`app_cal/delete.php`视图文件。在成功提交（如果没有验证失败）后，调用模型函数`delete()`，如果发生了删除，用户将被重定向到他们之前删除的预约所在的相同月份和年份。

# 创建一个助手函数来处理一个人的出生日期

有时你需要一个年龄验证脚本，一个确定用户是否达到一定年龄的方法。根据他们的年龄，他们可能被允许或禁止查看内容，例如，推广成人产品的网站，如酒精或烟草，或者推广特定年龄评级游戏的网站。这个菜谱中的代码帮助你确定用户的年龄，将其与最低年龄要求进行比较，并相应地显示 HTML 文件。

## 如何做到这一点...

我们将创建五个文件：

+   `/path/to/codeigniter/application/controllers/register.php`：这是我们的菜谱控制器

+   `/path/to/codeigniter/application/helpers/dob_val_helper.php`：此文件计算用户的年龄，将其与所需年龄进行比较，并根据结果返回 `TRUE` 或 `FALSE`

+   `/path/to/codeigniter/application/views/register/signup.php`：此文件显示年龄验证表单

+   `/path/to/codeigniter/application/views/register/enter.php`：如果用户可以进入，则显示此文件

+   `/path/to/codeigniter/application/views/register/noenter.php`：如果用户不能进入，则显示此文件

1.  创建控制器文件 `register.php`，并将以下代码添加到其中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Register extends CI_Controller {
      function __construct() {
        parent::__construct();
        $this->load->helper('form');
        $this->load->helper('dob_val');
      }
      public function index() {	
        $this->load->library('form_validation');
        $this->form_validation->set_error_delimiters('', '<br />');
        $this->form_validation->set_rules('year', 'Year', 'required|min_length[4]|max_length[4]|trim');
        $this->form_validation->set_rules('month', 'Month', 'required|min_length[2]|max_length[2]|trim');
        $this->form_validation->set_rules('day', 'Day', 'required|min_length[2]|max_length[2]|trim');

        if ($this->form_validation->run() == FALSE) {
          $this->load->view('register/signup');
        } else {
          $dob = array(
            'year' => $this->input->post('year'),
            'month' => $this->input->post('month'),
            'day' => $this->input->post('day')
          );
          $at_least = 18;
          if (are_they_old_enough($dob, $at_least = 18)) {
            $this->load->view('register/enter');
          } else {
            $this->load->view('register/noenter');
          }
        }
      }
    }
    ```

1.  创建辅助程序文件 `dob_val_helper.php`，并将以下代码添加到其中：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');
    function are_they_old_enough($dob, $at_least = 18) {
      $birthday = strtotime($dob['year'].'-'.$dob['month'].'-'.$dob['day']);
      $diff = floor((time() - $birthday) / (60 * 60 * 24 * 365));

      if ($diff >= $at_least) {
        return true;
      } else {
        return false;
      }
    }
    ```

1.  创建视图文件 `signup.php`，并将以下代码添加到其中：

    ```php
    <?php echo form_open() ; ?>

      <?php echo validation_errors() ; ?>

      Day <input type="text" name="day" size="5" value="<?php echo set_value('day') ; ?>"/>

      Month <input type="text" name="month" size="5" value="<?php echo set_value('month') ; ?>"/>

      Year <input type="text" name="year" size="5" value="<?php echo set_value('year') ; ?>"/>

      <input type="submit" value="go" />

    <?php echo form_close() ; ?>
    ```

1.  创建视图文件 `enter.php`，并将以下代码添加到其中：

    ```php
    <p>You are old enough to view page</p>
    ```

1.  创建视图文件 `noenter.php`，并将以下代码添加到其中：

    ```php
    <p>You are NOT old enough to view page</p>
    ```

## 它是如何工作的...

首先，我们来到控制器；控制器加载 URL 辅助程序（因为我们使用 `redirect()` 函数以及我们将创建的名为 `dob_val` 的辅助程序）：

```php
function __construct() {
  parent::__construct();
  $this->load->helper('form');
  $this->load->helper('dob_val');
}
```

然后，我们设置表单验证并从 HTML 中设置我们的日、月和年字段的规则。`register/signup.php` 视图文件被加载，准备用户在三个表单字段中输入他们的出生日期。

用户将点击提交，如果提交通过验证，则三个表单值将被放入 `$dob` 数组中：

```php
$dob = array(
	'year' => $this->input->post('year'),
	'month' => $this->input->post('month'),
	'day' => $this->input->post('day')
);
```

我们设置最小年龄（用户必须达到的年龄才能查看受年龄限制的内容）如下：

```php
$at_least = 18;
```

然后，我们将 `$dob` 数组以及 `$at_least` 变量传递给 `dob_val` 辅助程序：

```php
if (are_they_old_enough($dob, $at_least = 18)) {
  $this->load->view('register/enter');
} else {
  $this->load->view('register/noenter');
}
```

辅助程序会计算用户是否超过或低于 `$at_least` 年龄，如果超过则返回 `TRUE`，如果没有则返回 `FALSE`。如果用户符合条件，他们会看到 `register/enter` 视图文件，如果不满足条件，则会看到 `register/noenter` 视图文件。

# 在 CodeIgniter 中处理模糊日期

什么是模糊日期？模糊日期是一种更熟悉和通用的方式来描述数据或时间，而不是精确的时间；它以一种对读者来说更熟悉的方式描述事件。例如，与其说一封电子邮件在 17:41 发送，不如说它是在“不到一分钟前”发送的（假设你在最后一分钟内发送了它）或者甚至是“几分钟前”。某事发生的精确时间被认为是不重要的一—或者至少是不必要的——信息，取而代之的是对日期和时间的更一般、非正式和会话式的描述。

## 如何实现...

我们将要创建两个文件：

+   `/path/to/codeigniter/application/controllers/fuzzy_date.php`：此控制器将调用 `fuzzy_date_helper.php` 文件，并将一些日期（作为 Unix 时间戳）传递给它以供辅助程序转换。

+   `/path/to/codeigniter/application/helpers/fuzzy_date_helper.php`：这个辅助程序将由控制器调用，并将转换传递给它的日期，每次都返回一个文字描述。

1.  创建控制器文件，`fuzzy_date.php`，并向其中添加以下代码：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    class Fuzzy_date extends CI_Controller {
    	function __construct() {
    		parent::__construct();
    		$this->load->helper('url');
    		$this->load->helper('fuzzy_date_helper');););
    	}

    	public function index() {
    		echo describe_the_time(time() + 30);
    	}
    }
    ?>
    ```

1.  创建辅助程序文件，`fuzzy_date_helper.php`，并向其中添加以下代码：

    ```php
    <?php if ( ! defined('BASEPATH')) exit('No direct script access allowed');

    function describe_the_time($time_in) {
      define('SECOND', 1);
      define('MINUTE', 60 * SECOND);
      define('HOUR', 60 * MINUTE);
      define('DAY', 24 * HOUR);
      define('MONTH', 30 * DAY);
      define('YEAR', 12 * MONTH);

      $past_descriptions = array(
        1 => 'about a minute ago',
        2 => 'a few minutes ago',
        3 => 'within the last hour',
        4 => 'earlier today',
        5 => 'yesterday',
        6 => 'earlier this week',
        7 => 'earlier this month',
        8 => 'last month',
        9 => 'earlier this year',
        10 => 'last year',
        11 => 'a long time ago',
        12 => 'I don\'t know that time'
        );

      $future_descriptions = array(
        1 => 'a minute from now',
        2 => 'in the next few minutes',
        3 => 'in the next hour',
        4 => 'later today',
        5 => 'tomorrow',
        6 => 'later this week',
        7 => 'later this month',
        8 => 'next month',
        9 => 'later this year',
        10 => 'next year',
        11 => 'a long way off',
        12 => 'I don\'t know that time'
        );

      $now = time();

      if ($time_in < $now) {
        if ($time_in > $now - MINUTE) { // About a minute ago
          return $past_descriptions[1];
        } elseif ( ($time_in >= $now - (MINUTE * 5) ) && ($time_in <= $now ) ) { // A few minutes ago
          return $past_descriptions[2];
        } elseif ( ($time_in >= $now - (MINUTE * 60)) && ($time_in <= $now ) ) { // Within the last hour
          return $past_descriptions[3];
        } elseif ( ($time_in >= $now - (HOUR * 24)) && ($time_in <= $now - (MINUTE * 60) ) ) { // Earlier today
          return $past_descriptions[4];
        } elseif ( ($time_in >= $now - (HOUR * 48)) && ($time_in <= $now - (HOUR * 24) ) ) { // Yesterday
          return $past_descriptions[5];
        } elseif ( ($time_in >= $now - (DAY * 7)) && ($time_in <= $now - (HOUR * 48) ) ) { // Earlier this week
          return $past_descriptions[6];
        } elseif ( ($time_in >= $now - (DAY * 31)) && ($time_in <= $now - (DAY * 7) ) ) { // Earlier this month
          return $past_descriptions[7];
        } elseif ( ($time_in >= $now - (DAY * 62)) && ($time_in <= $now - (DAY * 31) ) ) { // Last Month
          return $past_descriptions[8];
        } elseif ( ($time_in >= $now - (MONTH * 12)) && ($time_in <= $now - (MONTH * 31) ) ) { // Earlier this year
          return $past_descriptions[9];
        } elseif ( ($time_in >= $now - (MONTH * 24)) && ($time_in <= $now - (MONTH * 12) ) ) { // Last year
          return $past_descriptions[10];
        } elseif ( ($time_in >= $now - (MONTH * 24) && ($time_in <= $now - (MONTH * 12) ) ) ){  // Last year
          return $past_descriptions[11];
        } else {
          return $past_descriptions[12];
        }
      } else {
        if ($time_in < $now + MINUTE) { // A minute from now
          return $future_descriptions[1];
        } elseif ( ($time_in <= $now + (MINUTE * 5) ) && ($time_in >= $now ) ) { // In the next few minutes
          return $future_descriptions[2];
        } elseif ( ($time_in <= $now + (MINUTE * 59)) && ($time_in >= $now ) ) { // In the next hour
          return $future_descriptions[3];
        } elseif ( ($time_in <= $now + (HOUR * 24)) && ($time_in >= $now + (MINUTE * 59) ) ) { // Later today
          return $future_descriptions[4];
        } elseif ( ($time_in <= $now + (HOUR * 48)) && ($time_in >= $now + (HOUR * 24) ) ) { // Yesterday
          return $future_descriptions[5];
        } elseif ( ($time_in <= $now + (DAY * 7)) && ($time_in >= $now + (HOUR * 48) ) ) { // Earlier this week
          return $future_descriptions[6];
        } elseif ( ($time_in <= $now + (DAY * 31)) && ($time_in >= $now + (DAY * 7) ) ) { // Earlier this month
          return $future_descriptions[7];
        } elseif ( ($time_in <= $now + (DAY * 62)) && ($time_in >= $now + (DAY * 31) ) ) { // Last Month
          return $future_descriptions[8];
        } elseif ( ($time_in <= $now + (MONTH * 12)) && ($time_in >= $now + (MONTH * 31) ) ) { // Earlier this year
        return $future_descriptions[9];
        } elseif ( ($time_in <= $now + (MONTH * 24)) && ($time_in >= $now + (MONTH * 12) ) ) { // Last year
          return $future_descriptions[10];
        } elseif ( ($time_in <= $now + (MONTH * 24) ) ) { // Last year
          return $future_descriptions[11];
        } else {
          return $future_descriptions[12];
        }
      }
    }

    ?>
    ```

## 它是如何工作的……

让我们看看`fuzzy_date`控制器，控制器在构造函数中加载，它反过来加载我们的`fuzzy_date_helper`，这是将 Unix 时间戳转换为描述性文本的辅助程序：

```php
  function __construct() {
    parent::__construct();
    $this->load->helper('url');
    $this->load->helper('fuzzydate_helper');
  }
```

然后，我们加载公共函数`index()`，它调用`fuzzydate_helper`，并传递给它一个时间戳（目前，传入的输入设置为`time() + 30`）。

为什么是`time() + 30`？好吧，`time()`是`php`函数，它返回“现在”的 Unix 时间戳（对你来说，“现在”是什么时候），而`+ 30`是将 30 秒加到`time()`返回的当前时间戳值上，意味着“现在加 30 秒”（或“未来 30 秒”）。我将其设置为初始演示，但稍后我将描述如何修改它。

在控制器中，我们将“现在加 30 秒”传递给辅助程序：

```php
echo describe_the_time(time() + 30);
```

进入函数的传入函数（即我们在控制器中定义的）在辅助程序中本地声明为`$time_in`：

```php
function describe_the_time($time_in) {
```

辅助程序获取`$time_in`变量，查看其值，并计算出时间戳值是否大于或小于辅助程序中定义的`$now`（即$now = time()）：

```php
if ($time_in < $now) {
... // $time_in is in the past
} else {
... // $time_in is in the future
}
```

由于“现在加 30 秒”大于现在（精确地说，多了 30 秒），辅助程序进入`if`结构的`else`部分，然后开始一系列比较，试图找到一个地方，其中在`$time_in`中定义的值可以匹配。由于“现在加 30 秒”小于“现在加 60 秒”（实际上少了 30 秒），第一个`if`语句被应用，辅助程序返回`$future_descriptions`数组中的第一个元素：

```php
if ($time_in < $now + MINUTE) { // A minute from now
  return $future_descriptions[1];
}
```

那将是现在一分钟之后吗？

当然，您可以在控制器中修改传递给辅助程序的时间戳；您可以将其设置为`time() + 250`（这将现在是 250 秒）。现在，加上 250 秒大于一分钟（60 秒），但小于五分钟（300 秒），这将导致第二个`if`语句应用并返回文本“在接下来的几分钟内”。

我们还可以传递一个比“现在”更低的时间戳值：`time() – 4000`（即“现在减去 4000 秒”）。传递一个较低的时间戳将导致辅助程序的行为改变。如何？别忘了最初的`if`语句？

```php
if ($time_in < $now) {
... 
} else {
...
}
```

这次，我们不会进入它的 else 部分；相反，初始的`if`部分将被触发，辅助程序将开始处理过去的时间。因此，通过将辅助程序输入设置为`time() - 4000`（现在是 4000 秒之前，超过一分钟，超过五分钟，超过一小时但不到一天），辅助程序将返回文本“今天早些时候”。

当然，你可以添加更多更多详细的比较和描述；实际上，我鼓励你这样做——尽情发挥吧！

在实际情况中，你会在控制器中修改辅助函数的调用，移除`time() +`或`- number_of_seconds`的输入，只保留你想要描述的事物的时戳——无论是博客文章的创建日期、文件上传日期、预约日期……无论什么……我相信你明白了我的意思。
