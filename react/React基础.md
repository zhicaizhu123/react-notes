### state和props
state是组件对内的接口，组件内部状态的变化通过state来反映；props是组件对外的接口，组件通过props接收外部传入的数据（包括方法）；props是只读的，不能再组件内部修改props；state是可变的，组件状态的变化通过修改state来实现。

### 有状态组件和无状态组件
如果一个组件的内部状态是不变的，当然就用不到state，这样的组件成为无状态组件；一个组件的内部状态会发生变化，就需要使用state来保存变化，这样的组件称为有状态组件。      
定义无状态组件除了使用ES6 Class方式外，还可以使用函数定义；一个函数组件接收props左外参数，返回代表这个组件UI的React元素结构。
``` js
function Welcome (props) {
    return <h1>Hello, {props.name}</h1>
}
```
**在使用无状态组件时，应该尽量将其定义成函数组件。** React应用组件设计的一般思路：通过定义少数的有状态组件管理整个应用的状态变化，并且将状态通过props传递给其余的无状态组件，由无状态组件页面绝大部分UI的渲染工作；有状态组件主要关注处理状态变化的业务逻辑，无状态组件主要关注组件UI的渲染。

### 属性校验和默认属性
React提供了PropTypes对象，用于校验组件属性的类型。PropTypes包含组件属性所有可能的类型，通过定义一个对象（对象的key是组件的属性名，value是对应属性的类型）实现组件属性类型。
``` js
import { Component } from 'react'
import PropTypes from 'prop-types'

class PostItem extends Component {
    // ...
}

PostItem.propTypes = {
    post: PropTypes.object,
    onVote: PropTypes.func
}
```
对于PropTypes.object和PropTypes.array检验属性类型是，如果想知道对象的结构或者数组元素的类型，可以使用PropTypes.shape或PropTypes.arrayOf
``` js
style: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number;
})

sequence: PropTypes.arrayOf(PropTypes.number)
```
React还提供了为组件属性设置默认值的特性，可以通过defaultProps实现。
``` js
function Welcome (props) {
    return <h1>hello, {props.name}</h1>
}

Welcome.defaultProps = {
    name: 'Peter'
}
``` 

### 组件和元素
React元素是普通的一个Javascript对象，这个对象通过DOM节点或React组件描述界面是什么样子的；React组件是一个class或者function，它接收一些属性作为输入，返回一个React元素。
``` js
// Button是一个React组件
class Button extends React.Component {
    render () {
        return <button>确定</button>
    }
}

// button是一个代表组件Button的React元素
const button = <Button />
```

### 组件的生命周期
##### 挂载阶段
> 依次调用constructor，componentWillMount，render，componentDidMount

- constructor   
接收一个props参数，props是从父组件中传入的属性对象，必须再这个方法首先调用super(props)才能确保props被传入组件中。   
constructor通常用于初始化组件的state以及绑定事件处理方法的工作
- componentWillMount    
这个方法在组件被挂载到DOM前调用，且只会被调用一次。 在这个方法中调用this.setState不会引起组件的重新渲染。
- render    
这是定义组件时唯一必要的方法，render并不负责组件的实际渲染工作，他只是返回一个UI的描述，正在渲染页面DOM的工作由React自身负责。render是一个纯函数，在这个方法里面不能执行任何有副作用的操作，不能再render中调用this.setState，这会改变组件的状态。
- componentDidMount     
在组件被挂载到DOM后调用，且只会被调用一次，这个时候已经获取到DOM的结果，可以在这个方法里面对DOM节点进行操作。通常在这个方法里面向服务器端请求数据。在这个方法中调用setState会引起组件的重新渲染。

##### 更新阶段
> 依次调用componentWillReceiveProps，shouldComponentUpdate，componentWillUpdate，render，componentDidUpdate

- componentWillReceiveProps(nextProps)     
这个方法只有在props 引起组件更新过程中，才会被调用，state引起的组件更新不会触发这个方法的执行。方法的参数nextProps是父组件传递给当前组件的新的props，父组件的render调用并不能保证传递给子组件的props发生变化，所以**需要比较nextProps和this.props来决定是否执行props发生变化后的逻辑**。
- shouldComponentUpdate(nextProps， nextState)    
**这个方法决定组件是否继续执行更行过程**，返回true会继续更新过程，返回false，则更新过程停制，后续的componentWillUpdate，render，componentDidUpdate都不会被动用。一般通过比较nextProps，nextState和组件当前的props，state决定这个方法的返回结果。这个方法可以用来减少组件的不必要渲染，从而优化组件的性能。
- componentWillUpdate(nextProps, nextState)   
这个方法在组件render调用之前执行，可以作为组件更新发生前执行某些工作的地方，一般很少用到。
- componentDidUpdate(prevProps, prevState)      
组件更新后被调用，可以在这个方法操作DOM，可以接收两个参数更新前的props和state

##### 卸载阶段
- componentWillUnmount      
在组件卸载前被调用，可以在这里执行一些清理工作，以避免引起内存泄漏

### 列表和Keys
``` js
class PostList extends Component {
    ...
    render () {
        return (
            <ul>
                {
                    this.state.list.map(item => (
                        <li key={item.id}>{item.name}</li>
                    ))
                }
            </ul>
        )
    }
}
```
React使用key属性来标识列表中的每个元素，当列表数据变化时，React可以通过keys知道哪些元素发生了变化，从而只重新渲染发生变化的元素，提高渲染效率。  
但并不推荐使用索引作为key，因为一旦列表的数据发生重拍，数据的索引就会发生变化，不利于React的渲染优化。

### 事件处理
- 事件的命名采用驼峰命名方式，如onclick要写成onClick
- 处理事件的响应函数要以对象的形式赋值给事件属性
``` js
<button onClick={onSubmit}>确认</button>
```
- React的事件是合成事件，并不是原生的DOM事件，在React事件中，如果要阻止事件默认行为，需要显示的调用事件对象的preventDefault方法。
- React事件处理函数的写法
    - 使用箭头函数
    ``` js
    ...
    onSubmit (event) {
        ...
    }
    
    render() {
        return (
            <button onClick={(event) => { this.onSubmit(event) }>确认</button>
        ) 
    }
    ...
    ```
    直接在render方法中为元素定义事件处理函数，最大的问题是，每次render调用时，但会重新创建一个新的处理函数，带来额外的性能开销，组件所处层级越低，开销越大
    - 使用组件方法
    ``` js
    ...
    constructor (props) {
        super(props)
        this.onSubmit = this.onSubmit.bind(this)
    }
    
    onSubmit(event) {
        ...
    }
    
    render () {
        return (
            <button onClick={this.onSubmit}>确认</button>
        )
    }
    ...
    ```
    这种方式不需要每次render执行都重新创建一个回调函数，但是需要在构造函数中为事件处理函数绑定this
    ``` js
    ...
    constructor (props) {
        super(props)
    }
    
    onSubmit(msg, event) {
        ...
    }
    
    render () {
        return (
            <button onClick={this.onSubmit.bind(this, 'I Hello world')}>确认</button>
        )
    }
    ...
    ```
    上述方式可以为事件处理函数传递额外的参数，但是也会在render调用时重新创建一次事件处理函数。
    - 属性初始化语法（ES7的）
    ``` js
    onSubmit: (event) => {
        ...
    }
    
    render () {
        return (
            <button onClick={this.onSubmit}>确认</button>
        )
    }
    ```

### 表单
##### 受控组件
> 如果一个表单元素的值是有React来管理，那么它就是一个受控组件。React组件渲染时，并在用户和表单元素发生交互时控制表单元素的行为，保证组件的state成为界面上所有元素状态的唯一来源。

类型：
- 文本框    
文本框包含类型为input和textarea，它们受控的主要原理是，通过表单元素的value属性设置表单元素的值，通过表单元素的onChange事件监听值的变化，并将改变同步到React组件的state中。
``` js
···
constructor (props) {
    this.state = {
        name: '',
        description: ''
    }
}

onChange: (event) => {
    const { name, value } = event.target
    this.setState({
        [name]: value
    })
}

render () {
    return (
        <form>
            <input name="name" value={this.state.name} onChange={this.onChange} />
            <textarea name="description" value={this.state.description} onChange={this.onChange}></textarea>
        </form>
    )
}
···
```
- 列表
``` js
...
constructor (props) {
    super(props)
    this.state = {
        value: '',
        options: [
            {
                label: 'React',
                value: 1,
            },
            {
                label: 'Vue',
                value: 2,
            },
            {
                label: 'Angular',
                value: 3,
            }
        ]
    }
}

onChange: (event) => {
    const { value } = event.target
    this.setState({
        value
    })
}

render () {
    return (
        <form>
            <select value={this.state.value} onChange={this.onChange}>
                {
                    this.state.options.map(item => {
                        <option value={item.value}>{item.label}</option>
                    })
                }
            </select>
        </form>
    )
}
...
```
- 复选框和单选框
复选框的类型为checkbox的input元素，单选框是类型为radio的input，让门的受控方式不同与类型为text的input元素，复选框和单选框的值是不变的，需要改变的是它们的checked状态，所以React控制的属性不再是value属性，而是checked属性。
``` js
constructor (props) {
    super(props)
    this.state = {
        checked: '',
        radio: ''
    }
}

onChange: (event) => {
    const { name, value } = event.target
    this.setState({
        [name]: value
    })
}

render () {
    return (
        <form>
            <input value="checkbox" name="checkbox" type="checkbox" checked={this.state.checkbox} onChange={this.onChange} />
            <input value="radio" name="radio" type="radio" checked={this.state.radio} onChange={this.onChange} />
        </form>
    )
}
...
```

##### 非受控组件
> 使用受控组件虽然保证了表单元素的状态由React统一管理，但是需要为每个表单元素定义onChange事件的处理函数，然后把表单状态的更改同步到React组件的state，这一过程比较繁琐，由一种可以替代的解决方案就是使用非受控组件。

**非受控组件指表单元素的状态依然由表单元素自己管理，而不是交给React组件管理**。使用非受控组件需要有一种方式可以获取到表单元素的值，React提供了一个特殊的属性ref，用来引用React组件或DOM元素的示例，所以可以通过为表单元素定义ref属性获取元素的值。
``` js
...
constructor (props) {
    super(props)
}

onSubmit: (event) => {
    console.log(this.input.value)
}

render () {
    return (
        <form onSubmit={this.onSubmit}>
            <input type="text" ref={(input) => this.input = input} />
        </form>
    )
}
...
```
**ref的值是一个函数，这个函数可以接收当前元素作为参数**     
在使用非控组件时，可以使用`defaultValue`属性指定默认值。
``` js
render () {
    return (
        <form onSubmit={this.onSubmit}>
            <input defaultValue="搜索关键词" type="text" ref={(input) => this.input = input} />
        </form>
    )
}
```


