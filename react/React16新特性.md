### render新的返回类型
React16之前，render只支持返回单个元素，现在render支持三个返回类型：**单个React元素、数组（有React元素组成）和字符串**

### 错误处理
React 16之前，组件在运行期间如果执行出错，就会阻塞整个应用的渲染，这时候只能刷新页面才能回复应用。React 16引入了新的错误处理机制，默认情况下，当组件抛出错误时，这个组件就会从组件树种卸载，从而避免整个应用的崩溃。React 16还提供了一种更加友好的错误处理方式——**错误边界**。错误边界时错误捕获子组件的错误并对其做优雅处理的组件。  
定义了`componentDidCatch(error, info)`这个方法的组件将成为一个错误边界
``` js
class ErrorBoundary extends Component {
    constructor (props) {
        super(props)
        this.state = {
            hasError: false
        }
    }
    
    componentDidCatch (error, info) {
        this.setState({
            hasError: true
        })
        console.log(error, info)
    }
    
    render () {
        if (this.state.hasError) {
            return <h1>Oops, has Comoponent is Error</h1>
        }
        return this.props.children
    }
}
```
让后在App种使用ErrorBoundary
``` js
class App extends Component {
    contructor (props) {
        super(props)
        this.state = { 
            user: {
                name: 'React'
            }
        }
    }
    
    onClick = () => {
        this.setState({user: null})
    }
    
    render () {
        return (
            <div>
                <ErrorBoundary>
                    <Profile user={this.state.user} />
                </ErrorBoundary>
                <button onClick={this.onClick}>更新</button>
            </div>
        )
    }
}
```

### Portals
React 16的Protals特性可以把组件
渲染到当前组件树意外的DOM节点上，这个特性典型的应用场景就是渲染应用的全局弹窗，使用Protals后，任意组件都可以将弹窗组件渲染到根节点上，以方便弹窗的显示。
``` js
ReactDOM.createProtal(child, container)
```
child为可以被渲染的React节点，container是一个DOM元素，child将挂载到这个DOM节点。
``` js
class Modal extends Component {
    contructor (props) {
        super(props)
        this.container = document.createElement('div')
        document.body.appendChild(this.container)
    }
    
    componentWillUnmont () {
        document.body.removeChild(this.container)
    }
    
    render () {
        return ReactDOM.createProtal(
            <div className="modal">
                {this.props.children}
            </div>,
            this.container
        )
    }
}
```
在App中使用Modal
``` js
class App extends Component {
    contructor (props) {
        super(props)
        this.state = { showModal: false } 
    }
    
    onClose = () => {
        this.setState({
            showModal: false
        })
    }
    
    render () {
        return (
            <div>
                {
                    this.state.showModal && (
                        <Modal onClose={onClose}>I am a Modal</Modal>
                    )
                }
            </div>
        )
    }
}

```

### 自定义DOM属性
React 16之前会忽略不识别的HTML和SVG属性，现在React 16会把不识别的属性传递给DOM元素。