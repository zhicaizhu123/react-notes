### 安装react-redux
Redux和React并没有直接的关系，Redux可以和很多库一起使用，为了方便在React中使用Redux，需要使用react-redux这个库
``` sh
npm i -S react-redux
```
### 展示组件和容器组件（根据意图区分）
- 展示组件：负责应用的UI展示，也就是组件如何渲染，具有很强的内聚性。展示组件不关心渲染时使用的数据时如何获取的，它只要知道有了这些数据后，组件应该如何渲染就足够了。
- 容器组件：负责应用逻辑的处理，如发送网络请求，处理返回数据，将处理的数据传递给展示组件使用等，容器组件还提供修改源数据的方法，通过展示组件的props传递给展示组件，当展示组件的状态变更引起源数据变化时，展示组件通过调用容器组件的提供的方法同步这些变化。一个容器组件可以包含多个展示组件和其他容器组件，一个展示组件也可以包含容器组件和其他展示组件。**容器组件负责业务逻辑处理，展示组件只关注UI的渲染逻辑。**

### connect
> react-redux提供一个connect组件，用于把React组件和Redux的store连接起来，生成一个容器组件，负责数据管理和业务管理。

假如有connect()(TodoList)生成VisibleTodoList组件，从而让TodoList与Redux连接起来。根据Redux的数据流过程，VisibleTodoList需要承担两个工作：
1. 从Redux的store中获取展示组件所需的应用状态
2. 把展示组件的状态改变同步到Redux的store中     

通过connect传递连个参数可以让TodoList具备这两个功能
``` js
import { connect } from 'react-redux'
import TodoList from './TodoList'

const VisibleTodoList = connect(
    mapStateToProps,
    mapDispatchToProps
)(TodoList)
```
mapStateToProps和mapDispatchToProps都是函数，**mapStateToProps负责从全局的应用状态state中提取出所需的数据，映射到展示组件的props，mapDispatchToProps负责把需要的action映射到展示组件的props上。**
- mapStateToProps
``` js
// ownProps是组件的props对象
function mapStateToProps (state, ownProps) {
    ...
}
```
- mapDispatchToProps
``` js
// ownProps是组件的props对象
function mapDispatchToProps (dispatch, ownProps) {
    ...
}
```
示例
``` js
// toggleTodo(id)返回一个action
function toggleTodo(id) {
    return { type: 'TOGGLE_TODO', id }
}

function mapDispatchToProps (dispatch) {
    return {
        onTodoClick: function (id) {
            dispatch(toggleTodo(id))
        }
    }
}
```
### Provider组件
通过connect函数创建出容器组件，但是这个容器组件时如何获取到Redux的store？react-redux提供了一个Provider组件。Provider需要接收一个store属性，然后把这个属性保存到context，Provider是通过context把store传递给子组件的，所以使用Provider组件时，一般把它作为跟组件，这样内层的任意组件都可以从context中获取到store对象
``` js
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import todoReducers from './reducers'
import App from './App'

const store = createStore(todoReducers)

render(
    <Provider store={store}>
        <App />
    <Provider>,
    document.querySelector('#root')
)
```

