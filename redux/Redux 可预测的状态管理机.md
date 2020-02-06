### 基本概念
Redux通过一系列约定的规范将修改应用状态的步骤标准化，让应用状态的管理不再错综复杂，而是如一条长线般清晰。
### 三大原则
> Redux应用需要遵循三大原则，否则程序很容易出现难以察觉的问题

- 唯一的数据源     
    Redux应用只维护一个全局的状态对象，存储在Redux的store中。唯一数据源是一种集中式管理应用状态的方式，便于监控任意时刻用用的状态和调试应用，减少出错的可能性
- 保持应用状态可读  
    在任何时候不能直接修改应用状态，当需要修改应用状态时，必须发送一个action，有这个action描述如何修改应用状态。
- 应用状态的改变通过纯函数完成  
    action表明修改应用状态的意图，真正对应用状态修改的时reducer。**reducer必须时纯函数**，所以reducer接收到action时，不能直接修改原来的状态对象，而是要创建一个新的状态对象返回。

### 主要组成
- action    
**action是Redux得信息载体，是store唯一的信息来源。把action发送到store必须通过store的dispatch方法**。action是普通的Javascript对象，但是**每个action必须有一个type属性描述action的类型，type是一般定义未字符串常量**，除了type属性外，action的结构完全可以有自己决定。一般通过action creator来创建action，action creator是返回action的函数。
``` js
// a addTodo action creator
function addTodo (text) {
    return {
        type: 'ADD_TODO',
        text
    }
}
```
- reducer   
action用于描述引用发生了什么操作，reducer则根据action做出响应，决定如何修改应用的状态state，那么就应该在编写reducer前设计好state
``` js
// todo state
{
    todos: [
        {
            id: 1,
            text: 'study Vue',
            completed: true
        },
        {   
            id: 2,
            text: 'study React',
            completed: true
        }
    ]
}
```
有了state，再为state编写reducer，**reducer为一个纯函数，它接收两个参数，当前的state和action，返回新的state**，
``` js
(oldState, action) => newState
```
创建一个基本的reducer
``` js
const initState = {
    todos: []
}

// reducer
function todos (state = initState, action) {
    switch (action.type) {
        case 'ADD_TODO':
            return {
            ...state,
            todos: [
                ...state.todos, 
                { id: action.id, text: action.text, completed: false }
            ]
            }
        case 'TOGGLE_TODO':
            return {
                ...todos,
                todos: state.todos.map(item => {
                    if (item.id === action.id) {
                        return { ...item, completed: !item.completed }
                    }
                })
            }
        default: 
            return state
    }
}
```
**当应用比较复杂时，可以拆分出多个reducer保存到独立的文件中，每个recuder处理state中的部分状态**
``` js
function todos (state = [], action) {
    switch (action.type) {
        case 'ADD_TODO':
            return [
                ...state, 
                { text: action.text, completed: false }
            ]
        case 'TOGGLE_TODO':
            return state.map(item => {
                if (item.id === action.id) {
                    return { ...item, completed: !item.completed }
                }
            })
        default: 
            return state
    }
}
```
**Redux还提供了combineReducers函数，用于合并多个reducer。**
``` js
import { combineReducer } from 'redux'
const todos = combineReducers({
    todos
})
```
等价于
``` js
function todos(state = {}, action) {
    return {
        todos: todos(state.todos, action)
    }
}
```
还可以为combineReducers接收的参数对象指定和recuder的函数名不同的key
``` js
import { combineReducer } from 'redux'
const todos = combineReducers({
    mytodos: todos
})
```
等价于
``` js
function todos(state = [], action) {
    return {
        mytodos: todos(state.mytodos, action)
    }
}
```
**combineReducers传递给每个reducer的state中的属性取决于它的参数对象的key值**

- store     
store时Redux中的一个对象（一个Redux应用中只有一个store）,是action和reducer之间的桥梁。store主要有以下工作
    - 保存应用状态
    - 通过方法getState()方法访问应用状态
    - 通过dispatch(action)发送更新状态的意图
    - 通过方法subscribe(listener)注册监听函数、监听应用状态的改变   

store保存了唯一数据源。store通过createStore()函数创建，创建时需要传递第一个参数为reducer，也可以传递第二个参数作为应用的初始状态
``` js
import { createStore } from 'redux'
import todos = './reducers'

// 创建store未设置初始状态
const store = createStore(todos)

// 创建store设置了初始状态
//const store = createStore(todos, initState)
```
创建store后，可以通过getState()获取当前应用的状态state
``` js
const state = store.getState()
```
当需要修改state时，可以通过store.dispatch()方法发送action
``` js
// 定义action
function addTodo ({ id, text }) {
    return {
        type: 'ADD_TODO',
        id,
        text
    }
}

// 发送action
store.dispatch(addTodo({ id: 4, text: 'study Vuex' }))
```
当reducer处理外action时，应用状态会被更新，为了能准确的知道应用状态的更新的时间，需要向store注册一个监听函数
``` js
const unsubscribe = store.subscribe(() => {
    console.log(store.getState())
})
```
store.subscribe()方法返回的是一个可以取消监听的函数
``` js
unsubscribe()
```

### Redux的数据流过程
1. 调用store.dispatch(action)，其中action是一个用于描述"发生了什么"的对象
2. Redux的store调用reducer函数。store传递两个参数给reducer：当前应用状态和action。reducer必须是一个纯函数，它的唯一职责及时计算下一个应用的状态。
3. 根reducer会把多个子reducer的返回结果结合成最终的应用状态。
4. Redux的store保存了根reducer返回的完整的应用状态。此时，应用状态才完成更新，对于React应用而言，可以再这个时候调用组件的setState方法，根据新的应用状态更新数据
