### 后台API架构  
##### 说在开头  
`Codeignitor` 是一个基于 `MVC` 的 `PHP` 框架。与大部分的 `MVC` 模式一样，`model` 负责获取数据库元数据，`controller` 负责业务逻辑处理，`view` 负责前端页面 `HTML` 的渲染。
但在实际的使用中，由于我们前端使用了 `React` ，相比较于传统的前端，我们已经将原来 `view` 层所需要处理的前端逻辑/用户操作完全交给了 `React`, 这样子我们只使用了 `MV` 两层。

##### 原先架构 MV => API  
原先采用的基本开发流程为：
1. `model` 层与数据库交互，获取元数据。
2. `controller` 层接受 `POST` 参数，进行业务处理，并返回前端想要的 `JSON` 格式。  

当业务逻辑变得复杂，需求频繁更改的时候容易出现以下弊端：
* `JSON` 格式不规范。
* `model` 层和 `controller` 层的业务紊乱。
* 新API需求提出之后可能会影响老API的正常使用，主要原因有：1、`model` 函数被更改；2、API名冲突；3、`POST` 参数名更改等等。
* 前后端沟通成本太大。
* 代码复用性低。

##### 于是提出了新一轮API的编写的规范  
1. 首先在 `model` 层之上添加了一层，这里我们称之为 `logic` 层，由于之前业务逻辑游离在 `model` 和 `controller` 之间，没有明确的一个规范，随着需求的变更和开发的迭代，两层处理的结构已经不能满足开发的需求，我们需要一层专门处理逻辑，将业务从 `model` 和 `controller` 中剥离出来。
2. 在开发过程中我们发现，`model` 层对数据库的操作(这里说的操作是除去业务的)，都有共性，即都是通用的增删查改。我们把这些数据操作称为：`CRUD` ，它包括：`Creact` 、`Read` 、`Update` 、`Delete`。 所以我们可以大致把我们的 `model` 函数概括为以下几个，以 `User` 表为例：
   * `add_usr($data = array())`
   * `get_user($id, $column = '*')` 
   * `get_users($where = array(), $column = '*', $offset = 0, $limit = 100)`
   * `update_user($id, $data = array())`
   * `delete_user($id)` 
   * `is_user($where = array())` 
3. 在 `logic` 层中我们编写业务代码，如把从 `model` 中获取来的元数据进行处理，组装成前端想要的数据格式；或者是对前端传来的参数进行校验。
4. 在 `controller` 中我们首先对所有的请求进行日志记录，`session` 处理（之后将会移除token），直接把 `post` 的数据丢给 `logic` ，由 `logic` 进行校验。这样子一来 `controller` 只是后台暴露给前端的一个地址，如果之后修改API，前端的请求地址不需要修改，只要在 `logic` 中修改或者重新一个方法就行了。
5. 在我们内部，我们可以利用 `logic` 这层进行单元测试，因为 `controller ` 只是起到传参数的作用，所以单元测试中我们mock的数据是和真实的 `post` 调用是一样的，只是缺少了日志的记录，这恰好是我们测试人员不想被记录的。 
6. 为了减少前后端沟通的成本，我们可以在 `controller` 中再做点文章，比如说是前端在测试API的时候先 `post` 过来一个 `key`  然后 `controller` 直接 `return` 一个构造好的数据格式。

