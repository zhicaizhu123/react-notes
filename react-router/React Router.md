### React Router安装
React Router有三个库：`react-router`、`react-router-dom`、`react-router-native`，其中`react-router-dom`、`react-router-native`都依赖于`react-router`，所以安装`react-router-dom`、`react-router-native`是会同时安装`react-router`
``` sh
npm i -S react-router-dom
```

### 路由器（Router）
- Router Router通过Router和Route两个组件完成路由功能。Router可以理解为路由器,一个应用只需要一个Router示例，所有的路由配置组件Route都定义为Router的子组件。
- 在Web项目中，一般会使用对Router进行包装的BrowserRouter或HashRouter两个组件。
    - BrowserRouter： 使用HTML5的history API实现应用的UI和URL的同步，一般还需要服务器进行配置，让服务器能正确地处理所有可能的URL
    - HashRouter：使用URL的hash事项应用的UI和URL的同步
- Router只能有唯一的一个子元素
``` js
ReactDOM.render((
    <BrowserRouter>
        <App />
    </BrowserRouter>
), document.querySelector('#root'))
```


### 路由配置（Route）
- Route是React Router中用于配置路由信息的组件，也是React Router使用频繁最高的组件，每当有一个组件选哟根据URL决定是否渲染时，就需要创建一个Route
    - path： 用来描述这个Route匹配的URL的pathname，例如`<Route path="/index" />`会匹配一个pathname已`/index`开始的URL（如：http://www.my.com/index）
    - match：当URL和Route匹配时，Route会创建一个match对象作为props中的一个属性传递给被渲染的组件，match有四个属性
        - params：Route的path属性可以包含参数，例如`<Route path="/detail/:id" />`，URL为 http://www.my.com/detail/2020 ，则params则为`params={id: 2020}`
        - isExact：是一个boolean，当URL完全匹配时，返回true，否则返回false
        - path：Route的path属性，构建嵌套路由的时候会使用到
        - url：URL的匹配部分
    - Route渲染组件的方式
        - compponent：值是一个组件，当URL和Route匹配时，component属性定义的组件就会被渲染
        ``` js
        <Route path="/index" component={Index} />
        ```
        - render：值是一个函数，该函数返回一个React元素，这种方式便于为待渲染的组件传递额外的属性
        ``` js
        <Route path="/index" render={
            (props) => (
                <Index {...props} data={extraProps} />
            )
        } />
        ```
        - children：值是一个函数，该函数返回一个React元素，与上面两种方式不一样的是，无论是否匹配成功，children返回的组件都会被渲染，但是当匹配步成功，match属性值为null
        ``` js
        <Route path="/index" children={
            (props) => (
                <div className={props.match ? 'active' : ''}>
                    <Index {...props} />
                </div>
            )
        } />
        ```
    - Switch和exact     
    当URL和多个Route匹配时，这个Route都会执行渲染
        - Switch: 如果只想第一个匹配的Rote渲染，那么可以将这些Route包到一个Switch组件中
        - exact：如果想让URL和Route完全匹配时，Route才渲染，那么可以使用Route的属性
        - Switch和exact常常联合使用，用于应用首页的导航
        ``` js
        <Router>
            <Switch>
                <Route exact path="/" component={Index} />
                <Route path="/list" component={List} />
                <Route path="/:user" component={User} />
            </Switch>
        </Router>
        ```
        如果不使用Switch，当URL的pathname 为 `/list`时，会匹配`<Route path="/list" component={List} />`和`<Route path="/:user" component={User} />`，使用后只会渲染第一个匹配`<Route path="/list" component={List} />`
    - 嵌套路由：指的是在Route渲染的组件内部定义写的Route
    ``` js
    <Route path="/articles" render={
            (props) => (
                <div>
                    <Route path={`${props.match.url}/:id`} component={articleDetail} />
                    <Route exact path={props.match.url} component={articleList} />
                </div>
                <Index {...props} data={extraProps} />
            )
        } />
    ```
### 链接（Link）
Link 是React Router提供的链接组件，一个Link组件定义了当点击该Link时，页面该如何跳转路由
``` js
const Navigation = () => (
    <header>
        <nav>
            <ul>
                <li>
                    <Link to="/">主页</Link>
                </li>
                <li>
                    <Link to="/mine">个人中心</Link>
                </li>
            </ul>
        </nav>
    </header>
)
```
其中to属性可以时String或Obejct类型，当to为Object类型时，可以包含pathname、search、hash、state四个属性。
``` js 
<Link to={{
  pathname: '/user',
  search: '?uid=2020',
  hash: '#top',
  state: { fromIndex: true }
}}>
    个人中心
</Link>
```
除了Link，还可以时使用history对象手动实现导航，history常用的方法时`push(path, [state])`和`repalce(path, [state])`，push会向浏览器历史记录新增一条记录，repalce会用新的记录替换当前记录
``` js
history.push('/user')

history.replace('/user', { fromIndex: true })
```

### 按需加载组件
> 每个路由依赖的代码单独打包成一个chunk文件

- 函数封装
``` js
// AsyncComponent.js

import React, { Component } from 'react'

// importComponent 是使用了import()的函数
export default function asyncComponent (importComponent) {
    class AsyncComponent extends Component {
        constructor (props) {
            super(props)
            this.state = {
                component: null
            }
        }
        
        componentDidMount () {
            importComponent()
                .then((mod) => {
                    this.setState({
                        component: mod.default ? mod.default : mod
                    })
                })
        }
        
        render () {
            const C = this.state.component
            return C ? <C {...this.props} /> : null
        }
    }
    
    return AsyncComponent
}
```
在AsyncComponent被挂在后，importComponent机会被调用，进而触发动态导入模块的动作。
- 示例代码
``` js
import React, { Component } from 'react'
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'
import asyncComponent from './AsyncComponent.js'

const AsyncHome = asyncComponent(() => import('./component/Home'))
const AsyncUser = asyncComponent(() => import('./component/User'))

class App extends Component {
    render () {
        <Router>
            <Switch>
                <Route exact path="/" component={AsyncHome} />
                <Route path="/user" component={AsyncUser} />
                <Route path="/home" component={AsyncHome} />
            </Switch>
        </Router>
    }
}

export default App
```
为什么asyncComponent不直接接收一个代表组件路径的字符串作为参数？因为在使用import()时，必须显式地声明要导入的组件路径，在webpack打包时，会根据这些显式的声明拆分代码，否则webpack无法获取足够的关于拆分代码的信息

    
