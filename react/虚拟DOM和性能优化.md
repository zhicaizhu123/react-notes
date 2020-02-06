### 虚拟DOM
> React之所以执行效率高，一个重要原因是它的虚拟DOM机制。

虚拟DOM使用普通的Javascript对象描述DOM元素，React元素本身就是一个虚拟DOM节点。下面一个DOM结构：
``` html
<div class="container">
    <h1>Hello React</h1>
</div>
```
可以使用一个Javascript对象来表达
``` js
{
    type: 'div',
    props: {
        className: 'container',
        children: {
            type: 'h1',
            props: {
                children: 'Hello React'
            }
        }
    }
}
```
有了虚拟DOM这一层，我们需要操作DOM时，就可以操作虚拟DOM，而不操作正式DOM，虚拟DOM是普通的Javascript对象，访问Javascript对象比访问真实DOM要快的多。

### Diff算法
当组件执行render的时会返回一个新的虚拟DOM对象，React会通过比较新旧虚拟
DOM的结构的变化找出差异部分，更新到真实DOM上，减少最终要在真实DOM上执行的操作，提高程序执行效率。 、

**两个假设**：
- 如果两个元素的类型相同，那么它们将生成两颗不同的树
- 为列表中的元素设置key值，用key标识对应的元素在多次render过程中是否发生变化    

基于这两个假设，React实现了在O(N)时间复杂度内完成两棵虚拟DOM树的比较。  

**如何比较**
- 实现假设一
    1. 当根节点时不同类型时     
    React会认为新的树和旧的树完全不同，不会再继续比较其他属性和子节点，而是把整棵树拆掉重建。虚拟DOM的节点类型分为两类：DOM元素类型和React组件类型。
    2. 当根节点时相同的DOM元素类型时    
    React会保留根节点，而比较根节点的属性，然后只更新哪些变化了的属性。
    3. 当根节点时相同的组件类型时   
    如果两个根节点时相同类型的组件，对应的组件实例不会销毁，只是执行更新操作，同步变化的属性到虚拟DOM树上，这一过程组件实例的componentWillReceiveProps()和componentWillUpdate()会被调用。    
    比较完根节点后，React会以同样的原则继续递归比较子节点。
- 实现假设二    
    React提供了key属性，这个key为了帮助React提高DIff算法的效率，当一个子节点定义了一个key，React会根据key来匹配子节点，之哟子节点的key值没有变化，React就认为这个同一个节点。尽量不要使用元素再列表的索引值作为key，因为列表中的顺序一旦改变，就可能导致大量的key失效，进而引起大量的修改操作。

### 性能优化
- 使用生产环境版本的库
- 避免不必要的组件渲染  
    - 当组件props或者state发生变化时，组件的render就会被重新调用，返回新的虚拟DOM对象。React组件生命周期方法提供一个shouldComponentUpdate发方，这个方法默认返回true，如果返回false，组件此次更新将会停止，也就是后续的componentWillUpdate、render等方法都不会执行。如果对对象层层比较比较耗性能，所以折中方案时，值比较对象的第一层级的属性，也就是执行浅比较。    
    - React提供了一个PureComponent组件，这个组件使用浅比较来比较新旧props和state。但是使用浅比较很容易直接修改数据而产生错误。
- 使用key

### 性能检测工具
- React Developer Tools for Chrome
- Chrome Perfomance Tab
- why-did-you-update    
why-did-you-update 会比较组件的state和props的变化，从而实现组件render方法不必要的调用。
``` sh
# 安装
npm install why-did-you-update -D
```
``` js
// 使用
import React from 'react'

if (process.env.NODE_ENV !== 'production') {
    const { whyDidYouUpdate } = require('why-did-you-update')
    whyDidYouUpdate(React)
}
```