最近在新项目中用到了react hooks，趁热乎总结一下。
@[TOC](初始React Hooks)
# 1. React Hooks是什么&为什么要用React Hooks?
以下引入官方文档中的简单介绍：Hook是React16.8的新特性，它可以让你在不编写class的情况下使用state以及其他React特性。

新特定的出现总归是要解决以往开发过程中存在的一些问题，它和之前的版本对比主要有以下优势：
	1.  可以再非class的情况下使用更多React特性。（可以不用再使用class来写react代码啦）
	2.  函数更加专一化。（一个函数只负责一块逻辑的处理，而并非将很多不相关的逻辑都堆在一个函数）
	3.  复用组件的状态逻辑。这也是它主要要解决的问题，但只是**状态逻辑**的复用，会共享数据的处理逻辑，而非数据本身。

# 2. React Hooks的分类
 根据官方文档的介绍，React Hooks主要有以下几种hook：
- **基础hook**
	- **useState**：返回一个state和一个更新state的函数，功能类似于class中的state和this.setState。
	- **useEffect**: 接收一个包含命令式、且可能有副作用代码的函数。它主要可以实现componentDidmount、componentWillUnmount、componentDidUpdate的功能。同时也可以在我们自己封装的hook中使用useEffect。
	- **useContext**:  接收一个context对象，并返回该context的当前值。调用了 useContext 的组件总会在 context 值变化时重新渲染。
**useContext(MyContext)只是相当于<MyContext.Consumer>**，只是能接收到context的值并订阅其变化。我们仍然需要在上层组件中用<MyContext.Provider> 来传递变化。

- **额外的hook**
	- **useReducer**： 它接收一个形如 (state, action) => newState 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。**可提供类似redux的功能**。
	- **useCallback**：它接收一个回调函数和一个依赖数组作为参数，返回一个**缓存的回调函数**。
	- **useMemo**：它接收一个回调函数和一个依赖数组作为参数，返回一个**缓存的值**。
	- **useRef**：返回一个可变的 ref 对象，并且在组件的整个生命周期内保持不变，即可**跨周期来获取数据**。
	- **useImperativeHandle**：可以在使用 ref 时自定义暴露给父组件的实例值。
	- **useLayoutEffect**：如果在useEffect中会操作dom，那就用useLsyoutEffect来代替。它里边的**回调函数会在dom更新完成之后执行，但在浏览器绘制前运行完成**。
	- **useDebugValue**： 可在React 开发者工具中显示自定义 hook 的标签。

# 3. 详解React Hooks
## (1) useState
const [count, setCount] = useState(0);其中0作为count的初始值，后续调用count来取值，调用setCount()来修改值。
- 可根据状态之间的关联性来选择将其设置为一个状态还是多个状态。例如
```javascript
const [name, setName] = useState('susan'); 
const [school, setSchool] = useState('school-one');
//由于name和school都属于个人信息，所以可以将其放在一个state中。
 const [user_info, setUserInfo] = useState({name: 'susan', school: 'school-one'}); 
```
- useState的调用必须放在顶层调用，不能在循环、条件语句、嵌套函数中。

```javascript
//正确调用
export const Basic = () => {
	const [count, setCount] = useState(0);
}
//错误调用
export const Basic = () => {
	const flag = 1;
	if(flag){
		const [count, setCount] = useState(0);	
	}
}
```
必须放在顶层调用的原因是**react对hook的存储是是按顺序的**，react会在第一次渲染时将state按useState的顺序逐个放到全局的数组中，后边每次更改state都会根据顺序去取state。如果不在非顶层调用的话，就会导致根据初始的序号取不到对应的state。

## (2) useEffect：
写法如下：useEffect(()=>{}, [])
- 第二个参数传与不传影响很大（第二个参数为依赖项，可选）
```javascript
import React, { useState, useEffect } from 'react';
export const BasicInfo = () => {
	const [count, setCount] = useState(0);
	//不写第二个参数，
	useEffect(()=>{
		document.title = `You clicked ${count} times`;
	})

	//第二个参数为空
	useEffect(()=>{
		document.title = `You clicked ${count} times`;
	}, [])

	//第二个参数有值
	useEffect(()=>{
		document.title = `You clicked ${count} times`;
	}, count)
}
```
以上三种写法所导致的结果分别是：
1. 不写第二个参数时：useEffect中第一个函数参数在每次渲染后都会执行一次。
2. 第二个参数为空时：函数只有在第一次渲染后会执行，相当于componentDidMount。
3. 第二个参数不为空时，函数只有在该参数发生变化时，才会执行。（可取，避免了不必要的执行）
***注意：useEffect会在每次渲染后都会执行，只是第一个回调函数会依据情况来执行***

- 依赖项是否发生了变化？

```javascript
import React, { useState, useEffect } from 'react';
export const BasicInfo = () => {
	const [count, setCount] = useState(0);
	const [item_list, setItemList] = useState([]);
	//第二个参数为基本数据类型（趁机回想一下基本数据类型都有什么来着？）
	useEffect(()=>{
		console.log('count...', count);
	}, [count])
	
	//第二个参数为为引用数据类型
	useEffect(()=>{
		console.log('item_liist...', item_list)
	}, [item_list])

	//第二个参数每次都是一个新的数组
	useEffect(()=>{
		console.log('item_liist...', item_list)
	}, [...item_list])
}
```
1. 当第二个参数为基本数据类型时，那么上述写法没有任何问题；
2. 当第二个参数为引用数据类型时，由于**react比较依赖是否发生变化的方法是简单的比较**，所以当还是同一个数组时，它并不会继续判断数组中的内容是否发生变化，就会直接认定该依赖没有变化。
	解决办法就是将依赖项每次都换成一个新的数组，如第三种写法。

## (3) useContext:
- 只是替代了consumer，provider该怎么写还是得怎么写

```javascript
// 创建 Context
const Context = React.createContext();//context包含consumer和provider

function Father() {
  return (
  //使用provider给子组件提供数据
    <Context.Provider data={'我是父组件的数据'}>
      <Son />
    </Context.Provider>
  );
}

// 使用 Consumer 从上下文中获取数据
function Son() {
  return (
    <Context.Consumer>
      {data => <div>{data}</div>}
    </Context.Consumer>
  );
}

//使用useContext从上下文中获取数据
function Son() {
	const data = useContext(Context);
	return <div>{data}</div>;
}
```

好啦，暂时就以基础hook作为本次分享的结尾啦，因为发现太长的文章可能看到一半就不太想看了，或者没有那么长连续的时间来仔细看。以上可能在有些地方理解的有出入，欢迎大家来讨论哦。下一篇将继续分享额外的hook的一些用法和注意的点。

希望大家能持续关注哦，留一些个人信息方便大家找到我哦。[知乎](https://www.zhihu.com/people/wen-xiao-yan-5-20/posts)   [github](https://github.com/wenjinhua?tab=repositories) 
附上个人公众号哦，希望大家前来骚扰（也会经常写一些随笔来记录生活，毕竟人生漫漫）
![](https://img-blog.csdnimg.cn/20200325120613452.jpg#pic_center)
