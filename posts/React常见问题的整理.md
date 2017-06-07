> React 起源于 Facebook 的内部项目，用来架设 Instagram 的网站。React 的设计可以简单理解为 **组件的组合模式和数据的单向流动**。以下是 React 常见问题的整理

#### setState之后发生了什么？

调用 setState() 函数之后，React 会将传入的参数对象与组件当前的状态进行合并。在合并过程中，React 会以相对高效的方式根据新的状态构建 React 元素树并重新渲染整个UI界面。在 React 得到元素树之后，React 会自动计算出新的树与老树的节点差异，然后根据差异对界面进行最小化重渲染。在差异计算 Diff 算法中，React 能够相对精确地知道哪些位置发生了改变以及应该如何改变，这就保证了按需更新，而不是全部重新渲染。

#### React 中 Element 和 Component 的区别是？

简单而言，React Element 是描述屏幕上所见内容的数据结构，是对于 UI 的对象表述。典型的 React Element 就是利用 JSX 构建的声明式代码片然后被转化为 createElement 的调用组合。而 React Component 则是可以接收参数输入并且返回某个 React Element 的函数或者类。

#### React 中 ref 的作用是什么？

ref 是 React 提供给我们安全访问 DOM 元素或者某个组件实例的句柄。我们可以为元素添加 ref 属性然后在回调函数中接受该元素在 DOM 树中的句柄，该值会作为回调函数的第一个参数返回：

```javascript
class CustomForm extends Component {
  handleSubmit = () => {
    console.log("Input Value: ", this.input.value)
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input type='text' ref={(input) => this.input = input} />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}
```

上述代码中的 input 域包含了一个 ref 属性，该属性声明的回调函数会接收 input 对应的 DOM 元素，我们将其绑定到 this 指针以便在组件内的其他函数中使用。另外值得一提的是，refs 并不是类组件的专属，函数式组件同样能够利用闭包暂存其值：

```javascript
function CustomForm ({handleSubmit}) {
  let inputElement;
  return (
    <form onSubmit={() => handleSubmit(inputElement.value)}>
      <input type='text' ref={(input) => inputElement = input} />
      <button type='submit'>Submit</button>
    </form>
  )
}
```

#### React 中 keys 的作用是什么？

Keys 是 React 用于追踪哪些列表中元素被修改、被添加或者被移除的辅助标识。

```javascript
render () {
  const {
    todoItems
  } = this.state;
  return (
    <ul>
      {todoItems.map(item => {
        return <li key={item.id}>{item}</li>
      })}
    </ul>
  )
}
```

在开发过程中，我们需要保证某个元素的 key 在其同级元素中具有唯一性。在 React Diff 算法中 React 会借助元素的 Key 值来判断该元素是新近创建的还是被移动而来的元素，从而减少不必要的元素重渲染。此外，React 还需要借助 Key 值来判断元素与本地状态的关联关系，因此我们绝不可忽视转换函数中 Key 的重要性。

key 必须是字符串类型，它的取值可以用数据对象的某个唯一属性，或是对数据进行hash来生成key。但是强烈不推荐用数组 index 来作为 key。如果数据更新仅仅是数组重新排序或在其中间位置插入新元素，那么视图元素都将重新渲染。来看下例子：

```javascript
// this.state.list = ['a','b','c']
<ul>
  {list.map((val, index) => {
   	return <li key={index}>{val}</li>
  })}
</ul>

<ul>
    <li key="0">a</li>
    <li key="1">b</li>
    <li key="2">c</li>
</ul>

// 数组重排 -> ['c','a','b']
<ul>
    <li key="0">c</li>
    <li key="1">a</li>
    <li key="2">b</li>
</ul>
```

React 发现 key 为 0,1,2 的元素的 text 都变了，将会修改三者的 DOM，而不是移动它们。

#### Controlled Component 与 Uncontrolled Component 之间的区别是什么？

React 的核心组成之一就是能够维持内部状态的自治组件，不过当我们引入原生的HTML表单元素时，如：input , select , textarea 等，我们是否应该将所有的数据托管到 React 组件中还是将其仍然保留在 DOM 元素中呢？这个问题的答案就是受控组件与非受控组件的定义分割。

受控组件（Controlled Component）代指那些交由 React 控制并且所有的表单数据统一存放的组件。譬如下面这段代码中 username 变量值并没有存放到 DOM 元素中，而是存放在组件状态数据中。任何时候我们需要改变 username 变量值时，我们应当利用 onChange 事件去调用 setState() 函数进行修改。

```javascript
class ControlledForm extends Component {
  state = {
    username: ''
  }
  updateUsername = (e) => {
    this.setState({
      username: e.target.value,
    })
  }
  handleSubmit = () => {
    console.log("Input Value: ", this.state.username);
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
     	  type='text'
          value={this.state.username}
          onChange={this.updateUsername} 
        />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}
```

而非受控组件（Uncontrolled Component）则是由 DOM 存放表单数据，并非存放在 React 组件中。我们可以使用 refs 来操控DOM元素，或者通过 id 来获取表单值

```javascript
class UnControlledForm extends Component {
  handleSubmit = () => {
    console.log("Input Value: ", this.input.value)
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type='text'
          ref={(input) => this.input = input} 
        />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}
```

#### 在生命周期中的哪一步你应该发起 AJAX 请求？

我们应当将AJAX 请求放到 componentDidMount 函数中执行，主要原因有下：

- React 下一代调和算法 Fiber 会通过开始或停止渲染的方式优化应用性能，其会影响到 componentWillMount 的触发次数。对于 componentWillMount 这个生命周期函数的调用次数会变得不确定，React 可能会多次频繁调用 componentWillMount。如果我们将 AJAX 请求放到 componentWillMount 函数中，那么显而易见其会被触发多次，自然也就不是好的选择。
- 如果我们将 AJAX 请求放置在生命周期的其他函数中，我们并不能保证请求仅在组件挂载完毕后才会要求响应。如果我们的数据请求在组件挂载之前就完成，并且调用了 setState() 函数将数据添加到组件状态中，对于未挂载的组件则会报错。而在 componentDidMount 函数中进行 AJAX 请求则能有效避免这个问题。


#### shouldComponentUpdate 的作用是啥以及为何它这么重要？

组件在决定重新渲染之前会调用 shouldComponentUpdate，该函数将是否重新渲染的权限交给了开发者，默认直接返回 `true`，表示默认直接出发dom更新。我们应该根据组件的应用场景设置函数的合理返回值能够帮我们避免不必要的更新。值得注意的是，React 会非常频繁的调用该函数，所以如果你打算自己实现该函数的逻辑，请尽可能保证性能。

#### 传入 setState 函数的第二个参数的作用是什么？

该函数会在 setState 函数调用完成并且组件开始重渲染的时候被调用，我们可以用该函数来监听渲染是否完成：

```javascript
this.setState(
  { username: 'yanm1ng' },
  () => console.log('setState has finished and the component has re-rendered.')
)
```

一个小栗子：

```javascript
componentDidMount(){
    this.setState({val: this.state.val + 1}, ()=>{
      console.log("In callback " + this.state.val);
    });

    console.log("Direct call " + this.state.val);   

    setTimeout(()=>{
      console.log("begin of setTimeout" + this.state.val);

       this.setState({val: this.state.val + 1}, ()=>{
          console.log("setTimeout setState callback " + this.state.val);
       });

      setTimeout(()=>{
        console.log("setTimeout of settimeout " + this.state.val);
      }, 0);

      console.log("end of setTimeout " + this.state.val);
    }, 0);
}
```

结果：

```bash
> Direct call 0
> In callback 1
> begin of setTimeout 1
> setTimeout setState callback 2
> end of setTimeout 2
> setTimeout of settimeout 2
```

#### React 绑定 this 的三种方式

React 可以使用 React.createClass 、ES6 classes 、纯函数3种方式构建组件。使用 React.createClass 会自动绑定每个方法的 this 到当前组件，但使用 ES6 classes 或纯函数时，就要靠手动绑定 this 了。

##### bind

`Function.prototype.bind(thisArg [, arg1 [, arg2, …]])` 是 ES5 新增的函数扩展方法， bind() 返回一个新的函数对象，该函数的 this 被绑定到 thisArg 上，并向事件处理器中传入参数。

```javascript
import React, { Component } from 'react'

class Test extends React.Component {
    constructor (props) {
        super(props)
        this.state = {
          message: 'Hello!'
        }
    }
    handleClick (name, e) {
        console.log(this.state.message + name)
    }
    render () {
        return (
            <div>
                <button onClick={this.handleClick.bind(this, 'yanm1ng')}>Say Hello</button>
            </div>
        )
    }
}
```

##### 构造函数

在构造函数  constructor 内绑定 this，好处是仅需要绑定一次，避免每次渲染时都要重新绑定，函数在别处复用时也无需再次绑定。

```javascript
import React, { Component } from 'react'

class Test extends React.Component {
    constructor (props) {
        super(props)
        this.state = {
          message: 'Hello!'
        }
        this.handleClick = this.handleClick.bind(this)
    }
    handleClick (e) {
        console.log(this.state.message)
    }
    render () {
        return (
            <div>
                <button onClick={this.handleClick}>Say Hello</button>
            </div>
        )
    }
}
```

##### 箭头函数

箭头函数则会捕获其所在上下文的 this 值，作为自己的 this 值，使用箭头函数就不用担心函数内的 this 不是指向组件内部了。可以按下面这种方式使用箭头函数：

```javascript
class Test extends React.Component {
    constructor (props) {
        super(props)
        this.state = {
          message: 'Hello!'
        }
    }
    handleClick (e) {
        console.log(this.state.message)
    }
    render () {
        return (
            <div>
                <button onClick={() => this.handleClick()}>Say Hello</button>
            </div>
        )
    }
}
```

#### React 组件生命周期

![](https://segmentfault.com/img/remote/1460000008325026?w=1674&h=1258/view)


