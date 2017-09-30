---
title: 【翻译】React模式 
---

[原文链接](http://reactpatterns.com/)

## 1. 无状态函数（Stateless function）
无状态函数是一种创建高度可复用组件的牛逼闪闪的方法，它自己不管理状态，他只是函数。
```jsx
const Greeting = () => <div>Hi there!</div>
```
可以传递`props`和`context`。
```jsx
const Greeting = (props, context) =>
  <div style={{color: context.color}}>Hi {props.name}!</div>
```
也可以定义局部变量。
```jsx
const Greeting = (props, context) => {
  const style = {
    fontWeight: "bold",
    color: context.color,
  }
  return <div style={style}>{props.name}</div>
}
```
当然也可以不定义局部变量，改为函数。
```jsx
const getStyle = context => ({
  fontWeight: "bold",
  color: context.color,
})

const Greeting = (props, context) =>
  <div style={getStyle(context)}>{props.name}</div>
```
无状态函数也可以拥有`defaultProps`，`propTypes`和`contextTypes`。
```jsx
Greeting.propTypes = {
  name: PropTypes.string.isRequired
}
Greeting.defaultProps = {
  name: "Guest"
}
Greeting.contextTypes = {
  color: PropTypes.string
}
```

## 2. JSX展开属性（JSX Spread Attributes）
展开属性是JSX的一个特性，一种语法糖，用来将一个对象的所有属性作为JSX的属性传递。
以下两个例子是等价的
```jsx
// props written as attributes
<main className="main" role="main">{children}</main>
```
```jsx
// props "spread" from object
<main {...{className: "main", role: "main", children}} />
```
用它可以方便地将属性转发给底层组件。
```jsx
const FancyDiv = props =>
  <div className="fancy" {...props} />
```
这时我给可以`FancyDiv`组件添加他关心和他不关心的属性。
```jsx
<FancyDiv data-id="my-fancy-div">So Fancy</FancyDiv>

// output: <div className="fancy" data-id="my-fancy-div">So Fancy</div>
```
注意属性顺序很重要，如果外部传入`className`属性，那么`FancyDiv`定义的`className`将会被覆盖。
```jsx
<FancyDiv className="my-fancy-div" />

// output: <div className="my-fancy-div"></div>
```
也可以让`FancyDiv`定义的`className`永远生效，只需要将它放在展开属性（{...props}）后面。
```jsx
// my `className` clobbers your `className`
const FancyDiv = props =>
  <div {...props} className="fancy" />
```
你应该优雅地处理这类情形，这种情况下我会合并使用者定义的`className`和组件自身的`className`。
```jsx
const FancyDiv = ({ className, ...props }) =>
  <div
    className={["fancy", className].join(' ')}
    {...props}
  />
```

## 3. 参数解构（Destructuring Arguments）
参数解构是ES2015的特性，它能够很好的配合无状态函数的参数。
以下两个例子是等价的。
```jsx
const Greeting = props => <div>Hi {props.name}!</div>

const Greeting = ({ name }) => <div>Hi {name}!</div>
```
[剩余参数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)（[中文链接](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Rest_parameters)）语法可以将剩余的参数手机到一个新对象中。
```jsx
const Greeting = ({ name, ...props }) =>
  <div>Hi {name}!</div>
```
反过来，这个新对象可以通过展开属性将属性转发给底层组件。
```jsx
const Greeting = ({ name, ...props }) =>
  <div {...props}>Hi {name}!</div>
```
应该避免将非DOM属性转发给原生组件，通过解构可以创建一个不包含高阶组件特有属性的新对象，因此解构可以让这个工作更加简单。

## 4. 条件渲染（Conditional Rendering）
组件定义内部是不能使用if/else条件语句的，但是可以使用[条件表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)。

if
```jsx
{condition && <span>Rendered when `truthy`</span> }
```
else
```jsx
{condition || <span>Rendered when `falsey`</span> }
```
if-else (tidy one-liners)
```jsx
{condition
  ? <span>Rendered when `truthy`</span>
  : <span>Rendered when `falsey`</span>
}
```
if-else (big blocks)
```jsx
{condition ? (
  <span>
    Rendered when `truthy`
  </span>
) : (
  <span>
    Rendered when `falsey`
  </span>
)}
```

## 5. Children类型（Children types）
React中children有好几种类型，常见的有数组和字符串。

字符串
```jsx
<div>
  Hello World!
</div>
```
array
```jsx
<div>
  {["Hello ", <span>World</span>, "!"]}
</div>
```
children也可以是函数，但是必须和父组件协作才能用。

function
```jsx
<div>
  {() => { return "hello world!"}()}
</div>
```

## 6. 数组类型的children（Array as children）
数组类型的children是非常常见的，列表就是这么渲染出来的。使用`map`函数就可以创建React元素数组。
```jsx
<ul>
  {["first", "second"].map((item) => (
    <li>{item}</li>
  ))}
</ul>
```
和下面这个数组字面量方式等价
```jsx
<ul>
  {[
    <li>first</li>,
    <li>second</li>,
  ]}
</ul>
```
为了更加简洁，可以结合解构，JSX展开属性，其他组件一起使用。
```jsx
<ul>
  {arrayOfMessageObjects.map(({ id, ...message }) =>
    <Message key={id} {...message} />
  )}
</ul>
```

## 7. 函数类型的children（Function as children）
函数类型的children不是天然有用的。
```jsx
<div>{() => { return "hello world!"}()}</div>
```
这种技术通常被称为渲染回调，可以给组件创作带来更多空间和便利。比如[ReactMotion](https://github.com/chenglou/react-motion)使用这种高能技术以后，渲染逻辑可以由使用者提供，而不是被类库写死。更多细节，请参考下一章渲染回调。

## 8. 渲染回调（Render callback）
下面这个组件使用了渲染回调技术，它没什么用，但它是一个很好的开端。
```jsx
const Width = ({ children }) => children(500)
```
该组件将children当做函数来调用，并传递了一个数字类型值为500的参数。

下面我们将使用该组件，并给它传递一个函数类型的children.
```jsx
<Width>
  {width => <div>window is {width}</div>}
</Width>
```
我们将得到以下结果。
```html
<div>window is 500</div>
```
有了这些设置，我们可以根据宽度来决定渲染什么。
```jsx
<Width>
  {width =>
    width > 600
      ? <div>min-width requirement met!</div>
      : null
  }
</Width>
```
如果这个逻辑会被多次使用，我们可以创建一个新组件来封装可重用逻辑。
```jsx
const MinWidth = ({ width: minWidth, children }) =>
  <Width>
    {width =>
      width > minWidth
        ? children
        : null
    }
  </Width>
```
很明显这对于一个有着固定宽度的组件没有什么意义，但对一个监听浏览器窗口宽度的组件就有意义了，以下是示例代码。
```jsx
class WindowWidth extends React.Component {
  constructor() {
    super()
    this.state = { width: 0 }
  } 

  componentDidMount() {
    this.setState({width: window.innerWidth});
    window.addEventListener("resize", ({target})=>{
      this.setState({width: target.innerWidth})
    });
  }

  render() {
    return this.props.children(this.state.width)
  }
}
```
很多开发者更喜欢高阶组件完成类似功能，这是个人偏好问题。

## 9. Children值传（Children pass-through）
有时候你可能会创建一个组件，只用来处理上下文并且直接渲染其children.
```jsx
class SomeContextProvider extends React.Component {
  getChildContext() {
    return {some: "context"}
  }

  render() {
    // how best do we return `children`?
  }
}
```
现在你需要作出决定，将`children`包裹在一个`<div />`中，还是直接返回`children`。第一种做法多了一层标签（可能导致样式失效），第二种做法将会导致一个错误。
```jsx
// option 1: extra div
return <div>{children}</div>

// option 2: unhelpful errors
return children
```
最好的做法是将`children`看做一个不透明的数据类型，`React`提供了`React.Children`来合理的处理`children`。
```jsx
return React.Children.only(this.props.children)
```

## 10. 组件代理（Proxy component）
（我不确定这个名字是否有意义）
按钮（Button）在网页应用中随处可见，每一个按钮都必须有一个`type`属性并设成`button`。
```jsx
<button type="button">
```
书写次数多了，也就容易导致错误，我们可以创建一个高阶组件代理该低阶组件。
```jsx
const Button = props =>
  <button type="button" {...props}>
```
这时我们可以使用`Button`代替`button`，确保`type`属性总被正确使用。
```jsx
<Button />
// <button type="button"><button>

<Button className="CTA">Send Money</Button>
// <button type="button" class="CTA">Send Money</button>
```

## 11. 使用样式（Style component）
这是一种使用样式的组件代理。
假设我们通过使用`class`将一个`button`装饰成主要（primary）按钮。
```jsx
<button type="button" className="btn btn-primary">
```
我们可以通过两个单一职责组件达到此目的。
```jsx
const PrimaryBtn = props =>
  <Btn {...props} primary />

const Btn = ({ className, primary, ...props }) =>
  <button
    type="button"
    className={classnames(
      "btn",
      primary && "btn-primary",
      className
    )}
    {...props}
  />
```
便于理解，请看下面的图示。
```
PrimaryBtn()
  ↳ Btn({primary: true})
    ↳ Button({className: "btn btn-primary"}, type: "button"})
      ↳ '<button type="button" class="btn btn-primary"></button>'
```
通过这些组件，以下代码是等价的。
```jsx
<PrimaryBtn />
<Btn primary />
<button type="button" className="btn btn-primary" />
```
对于样式维护来说真是一大福音，它将样式问题封装在单一组件中。

## 12. 事件切换（Event switch）
在写事件回调时通过采用`handle{EventName}`规则。
```jsx
handleClick(e) { /* do something */ }
```
对于一个需要处理多种事件事件的组件来说，这些函数名显得非常啰嗦。函数名中也不会带有更多信息，因为他们一般直接调用其他`action`或`function`。
```jsx
handleClick() { require("./actions/doStuff")(/* action stuff */) }
handleMouseEnter() { this.setState({ hovered: true }) }
handleMouseLeave() { this.setState({ hovered: false }) }
```
下面只给组件写一个事件处理函数，并通过`event.type`区分。
```jsx
handleEvent({type}) {
  switch(type) {
    case "click":
      return require("./actions/doStuff")(/* action dates */)
    case "mouseenter":
      return this.setState({ hovered: true })
    case "mouseleave":
      return this.setState({ hovered: false })
    default:
      return console.warn(`No case for event type "${type}"`)
  }
}
```
或者，对于简单组件，你可以通过胖箭头函数方式直接调用`action`或`function`。
```jsx
<div onClick={()=>someImportedAction({action:"DO_STUFF" })}>
</div>
```
不要担心性能问题，知道性能问题爆发。一定不要过早进行性能优化。

## 13. 布局组件（Layout component）
布局组件会产生一些静态DOM元素，他们可能不会有任何改变，即使改变了也不会很频繁。
下面是一个并排显示两个子组件的组件。
```jsx
<HorizontalSplit
  leftSide={<SomeSmartComponent />}
  rightSide={<AnotherSmartComponent />}
/>
```
我们可以尽量去优化这个组件。
虽然`HorizontalSplit`是两个组件的父组件，但是它绝不是这两个组件的所有者。我们可以让它永不更新，不影响组件的生命周期。
```jsx
class HorizontalSplit extends React.Component {
  shouldComponentUpdate() {
    return false
  }

  render() {
    <FlexContainer>
      <div>{this.props.leftSide}</div>
      <div>{this.props.rightSide}</div>
    </FlexContainer>
  }
}
```

## 14. 容器组件（Container component）
>“容器负责获取数据并渲染其子组件，这就够了”
>—Jason Bonta
假设我们已经有了可复用的`CommentList`组件。
```jsx
const CommentList = ({ comments }) =>
  <ul>
    {comments.map(comment =>
      <li>{comment.body}-{comment.author}</li>
    )}
  </ul>
```
接下来我们可以创建一个新组件负责获取数据并渲染无状态的`CommentList`组件。
```jsx
class CommentListContainer extends React.Component {
  constructor() {
    super()
    this.state = { comments: [] }
  }

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: comments =>
        this.setState({comments: comments});
    })
  }

  render() {
    return <CommentList comments={this.state.comments} />
  }
}
```
我们可以给不同的应用上下文创建不同的容器组件。

## 14. 高阶组件（Higher-order component）
[高阶函数](https://en.wikipedia.org/wiki/Higher-order_function)是一个接受函数类型的参数或返回一个新函数的函数。那么什么是高阶组件呢？
如果你已经开始使用容器组件，它们都是包裹在一个函数中的通用容器。
下面我们从一个无状态的`Greeting`组件开始。
```jsx
const Greeting = ({ name }) => {
  if (!name) { return <div>Connecting...</div> }

  return <div>Hi {name}!</div>
}
```
如果`Greeting`组件接到`props.name`，它就回去渲染这个数据，否则他会说正在连接。现在我们创建一个高阶组件。
```jsx
const Connect = ComposedComponent =>
  class extends React.Component {
    constructor() {
      super()
      this.state = { name: "" }
    }

    componentDidMount() {
      // this would fetch or connect to a store
      this.setState({ name: "Michael" })
    }

    render() {
      return (
        <ComposedComponent
          {...this.props}
          name={this.state.name}
        />
      )
    }
  }
```
它就是一个函数，返回一个渲染作为参数传递进去的组件的新组件。
最后，我们需要用`Connect`组件将`Greeting`组件包裹起来，如下：
```jsx
const ConnectedMyComponent = Connect(Greeting)
```
高阶组件是一个功能很强的模式，可以用来获取数据并给其他无状态组件提供数据。

## 15. 状态提升（State hoisting）
无状态组件并不持有状态，正如它名称暗示的那样。

Events are changes in state. Their data needs to be passed to stateful container components parents.

This is called “state hoisting”. It’s accomplished by passing a callback from a container component to a child component.
```jsx
class NameContainer extends React.Component {
  render() {
    return <Name onChange={newName => alert(newName)} />
  }
}

const Name = ({ onChange }) =>
  <input onChange={e => onChange(e.target.value)} />
```
`Name`组件从`NameContainer`组件中获得`onChange`回调并在事件中调用。
上面的`alert`只是简单演示并不修改状态，下面的代码将会修改`NameContainer`组件的状态。
```jsx
class NameContainer extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <Name onChange={newName => this.setState({name: newName})} />
  }
}
```
通过回调，状态被提升到维护局部状态的容器组件中。这给无状态函数一个清晰的边界和最大限度的可重用性。

这个模式并不局限于无状态函数，因为无状态函数没有生命周期事件，该模式同样适用于无状态组件。

受控的input就是一个使用了状态提升的重要模式。

## 16. 受控的input（Controlled input）
直接讨论受控的input比较困难，我们先从不受控的input谈起。
```jsx
<input type="text" />
```
当你在浏览器中输入框中输入时，你会看到输入框的值发生变化，这很正常。

受控的input禁用DOM突变，它的值只能被组件修改，不能被DOM修改。
```jsx
<input type="text" value="This won't change. Try it." />
```
上面的输入框有着固定值没有什么意义，下面输入框的值将会从`state`中获取。
```jsx
class ControlledNameInput extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <input type="text" value={this.state.name} />
  }
}
```

接着，修改输入框的值就是修改组件状态。
```jsx
    return (
      <input
        value={this.state.name}
        onChange={e => this.setState(e.target.value)}
      />
    )
```
这就是受控的input，只有当组件的状态改变了才能改变DOM，对于创建一致的UI，有着非常大的作用。