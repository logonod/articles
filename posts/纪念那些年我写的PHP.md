> 还是熟悉的口号：PHP 是世界上最好的语言

#### 为什么会接触PHP
接触PHP完全是因为 Web 开发。由 PHP 天然与 HTML、JavaScript 有亲近感，也就是原生味儿十足。即使是PHP与HTML代码混杂，从某种角度而言，也能帮助初学者直观地了解 Web 技术机理。

从单页 PHP 页面到 MVC ，再到 改造原有 MVC 架构 到服务于前端 React 的 API 设计。那段在学校养闲食堂三楼创业园区的时光中，我写过 PHP 代码甚至比前端代码还要多。。。

#### 基于MVC框架的改造之路

获取接触过 PHP  MVC 开发的同学都了解过 Codeignitor 。与大部分的 MVC 模式一样，`model` 负责获取数据库元数据，`controller` 负责业务逻辑处理，`view` 负责前端页面的渲染，具体流程如下：

![](http://img.ptcms.csdn.net/article/201510/21/56275703bfc42_middle.jpg)

但随着 AJAX 技术的流行，MVC 模式也进行了一丝丝改进：

![](http://img.ptcms.csdn.net/article/201510/21/562758bfa5c45_middle.jpg)

但在实际项目中，由于我们前端使用了 React ，相比较于传统的前端，我们已经将原来 `view` 层所需要处理的逻辑/用户操作完全交给了前端：

![](http://img.ptcms.csdn.net/article/201510/21/5627594400199_middle.jpg)

这样一来 MVC 我们只用到了 M 和 C，所以随着项目的不断迭代，`controller` 会变得越来越不堪重负。

#### 面向视图  => 面向API

基本开发流程为：

1. `model` 层与数据库交互，获取元数据。
2. `controller` 层接受前端传递的参数，进行业务处理，并返回前端想要的 `JSON` 格式。  

当业务逻辑变得复杂，需求频繁更改的时候容易出现以下弊端：

- `JSON` 格式不规范。
- `model` 层和 `controller` 层的业务紊乱。
- 新API需求提出之后可能会影响老API的正常使用，主要原因有：1、`model` 函数被更改；2、API名冲突；3、API 参数名更改等等。
- 前后端沟通成本太大。
- 代码复用性低。

#### 修改CI的结构并制定开发规范

1. 首先在 `model` 层之上添加了一层，这里我们称之为 `logic` 层，由于之前业务逻辑游离在 `model` 和 `controller` 之间，没有明确的一个规范，随着需求的变更和开发的迭代，两层处理的结构已经不能满足开发的需求，我们需要一层专门处理逻辑，将业务从 `model` 和 `controller` 中剥离出来。
2. 在开发过程中我们发现，`model` 层对数据库的操作(这里说的操作是除去业务的)，都有共性，即都是通用的增删查改。我们把这些数据操作称为：`CRUD` ，它包括：Creact 、Read 、Update 、Delete。 以 User_model 为例：
  ```php
  class User_model extends CI_Model {
    
    private $table = 'user';
    
    public function __construct() {
    	$this->load->database();
    }
    
    add_user($data = array()) {}
    
    get_user($u_id, $column = '*') {}
    
    get_user_page($where = array(), $column = '*', $offset = 0, $limit = 100) {}
    
    update_user($u_id, $data = array()) {}
    
    delete_user($u_id) {}
    
    is_user($where = array()) {}
  }
  ```
3. 在 `logic` 层中我们编写业务代码，如：把从 `model` 中获取来的元数据进行处理，组装成前端想要的数据格式。
4. 在 `controller` 中我们首先对所有的请求进行日志记录，身份校验，参数校验之后直接把过滤后的数据丢给 `logic` 。这样子一来 `controller` 只是充当了 路由 + 过滤器 的作用，如果之后修改 API ，前端的请求地址不需要修改，只要在 `logic` 中修改或者重新一个方法就行了。
5. 为了减少前后端沟通的成本，我们可以在 `controller` 中再做点文章，比如说前端在测试 API 的时候传过来一个事先约定好的 `key`  然后 `controller` 直接 `return` 一个构造好的数据格式，方便前端进行参数的构造。