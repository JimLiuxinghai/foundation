## 基础

### JSX

由于 JSX 会编译为 `React.createElement` 调用形式，所以 `React` 库也必须包含在 JSX 代码作用域内。

```react
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

编译为：

```javascript
React.createElement(
	MyButton,
  { color: 'blue', shadowSize: 2},
  'Click Me'
)
```

* 自定义组件开头必须是大写字母。

* `if` 语句以及 `for` 循环不是 [JavaScript 表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Expressions_and_Operators)，所以不能在 JSX 中直接使用。但是，你可以用在 JSX 以外的代码中。比如：

  ```react
  function NumberDescriber(props) {
    let description;
    if (props.number % 2 == 0) {    
      	description = <strong>even</strong>;  
   	} 
    else {    
      description = <i>odd</i>;  
    }  
    return <div>{props.number} is an {description} number</div>;
  }
  ```

* Props默认值为True

  ```react
  <MyTextBox autocomplete />
  <MyTextBox autocomplete={true} />
  {/*这两个JSX表达式是等价的*/}
  ```

  

* React 组件也能够返回存储在数组中的一组元素：

  ```react
  render () {
    return [
      <li key="A">First item</li>,
      <li key="B">Second item</li>,
      <li key="C">Third item</li>,
    ]
  }
  ```

* React DOM 在渲染所有输入内容之前，默认会进行[转义](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html)。它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 [XSS（cross-site-scripting, 跨站脚本）](https://en.wikipedia.org/wiki/Cross-site_scripting)攻击。

  ```javascript
  const title = response.potentiallyMaliciousInput;
  // 直接使用是安全的：
  const element = <h1>{title}</h1>;
  ```

  

* React中渲染HTML:

  使用`dangerouslySetInnerHTML`

  ```react
  <div style={{paddingLeft: 15, textIndent: 2}} dangerouslySetInnerHTML={{__html:item.helpContent}}>
  ```

  原理：

   * 1.dangerouslySetInnerHTML 是React标签的一个属性，类似angular的ng-bind。

   * 2.有2个`{{}}`,第一个`{}`代表`jsx`语法开始，第二个是代表`dangerouslySetInnerHTML`接收的是一个对象键值对。

   * 既可以插入DOM，又可以插入字符串。

   * `dangerouslySetInnerHTML`必须是一个对象 

     ```javascript
     function createEvent() {
       return {__html: 'First &nbsp;&nbsp;&nbsp; Second'};
     }
     
     function MyEvent() {
       return <div dangerouslySetInnerHTML={createEvent()}></div>;
     }
     ```

  

  ### Context

  * React.createContext

    ```react
    const MyContext = React.createContext(defaultValue)
    ```

    创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

    **只有**当组件所处的树中没有匹配到 Provider 时，其 `defaultValue` 参数才会生效。这有助于在不使用 Provider 包装组件的情况下对组件进行测试。注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效。

  * React.Provider

    ```react
    <MyContext.Provider value={/* 某个值 */}>
    ```

    每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

    Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

    当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

  * ### `Class.contextType`

    ```react
    class MyClass extends React.Component {
      componentDidMount() {
        let value = this.context;
        /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
      }
      componentDidUpdate() {
        let value = this.context;
        /* ... */
      }
      componentWillUnmount() {
        let value = this.context;
        /* ... */
      }
      render() {
        let value = this.context;
        /* 基于 MyContext 组件的值进行渲染 */
      }
    }
    MyClass.contextType = MyContext;
    ```

    挂载在 class 上的 `contextType` 属性会被重赋值为一个由 [`React.createContext()`](https://zh-hans.reactjs.org/docs/context.html#reactcreatecontext) 创建的 Context 对象。这能让你使用 `this.context` 来消费最近 Context 上的那个值。你可以在任何生命周期中访问到它，包括 render 函数中。

  * 子组件使用context

    * 1.static contexyType

      ```react
      class MyClass extends React.Component {
        static contextType = MyContext;
        render() {
          let value = this.context;
          /* 基于这个值进行渲染工作 */
        }
      }
      ```

    * 2.Context.Consumer

      ```react
      <MyContext.Consumer>
        	{this.context}
      </MyContext.Consumer>
      ```

### 高阶组件

具体而言，**高阶组件是参数为组件，返回值为新组件的函数。**

组件是将Props转化为UI，高阶组件是将组件转换为另一个组件。

* 属性代理

  属性代理是最常见的方式，我们可以在HOC中自定义一些属性，然后和新生成的属性一起传给被包裹的组件，如下：

  ```javascript
  const withHeader = title => WrappedComponent => {
    class HOC extends Component {
      static displayName = `HOC(${getDisplayName(WrappedComponent)})`
      render() {
        const newProps = {
          id: Math.random().toString(36).substring(2).toUpperCase()
        }
        return (
          <div>
            <h1 className='demo-header'>{title}</h1>
            <WrappedComponent {...this.props} {...newProps}/>
          </div>
        )
      }
    }
    return HOC
  }
  
  @withHeader('标题')
  class Demo extends Component {
    render() {
      return (
        <div style={this.props}>
          { this.props.children }
        </div>
      )
    }
  }
  
  class App extends Component {
    render() {
      return (
        <Fragment>
          <Demo color='blue'>我是一个普通组件</Demo>
        </Fragment>
      )
    }
  }
  ```

* 反向继承

  对上面的例子又做了一番修改：

  我们让HOC继承wrappedComponent，这样我们的HOC就拥有了wrappedComponent中定义的属性和方法了，如state，props，生命周期函数

  ```react
  const withHeader = title => WrappedComponent => {
    class HOC extends WrappedComponent {
      static displayName = `HOC(${getDisplayName(WrappedComponent)})`
      render() {
        return (
          <div>
            <h1 className='demo-header'>{title}</h1>
            { super.render() }
          </div>
        )
      }
      componentDidMount() {
        this.setState({
          innerText: '我的值变成了2'
        })
      }
    }
    return HOC
  }
  
  ```

  

### 事件

* 事件处理：
  - React 事件的命名采用小驼峰式（camelCase），而不是纯小写。
  - 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。



### Protals

通常来讲，当你从组建的render方法返回一个元素时，改元素将被挂载到DOM节点中离其最近的父节点：

```javascript
ReactDOM.createPortal(child, container)
```

第一个参数（`child`）是任何[可渲染的 React 子元素](https://zh-hans.reactjs.org/docs/react-component.html#render)，例如一个元素，字符串或 fragment。第二个参数（`container`）是一个 DOM 元素。

一个 portal 的典型用例是当父组件有 `overflow: hidden` 或 `z-index` 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框。

### Refs

**Ref 转发是一个可选特性，其允许某些组件接收 `ref`，并将其向下传递（换句话说，“转发”它）给子组件。**

```react
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// 你可以直接获取 DOM button 的 ref：
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```



以下是对上述示例发生情况的逐步解释：

1. 我们通过调用 `React.createRef` 创建了一个 [React ref](https://zh-hans.reactjs.org/docs/refs-and-the-dom.html) 并将其赋值给 `ref` 变量。
2. 我们通过指定 `ref` 为 JSX 属性，将其向下传递给 ``。
3. React 传递 `ref` 给 `forwardRef` 内函数 `(props, ref) => ...`，作为其第二个参数。
4. 我们向下转发该 `ref` 参数到 ``，将其指定为 JSX 属性。
5. 当 ref 挂载完成，`ref.current` 将指向 `` DOM 节点



回调Refs

```react
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    // 使用 `ref` 的回调函数将 text 输入框 DOM 节点的引用存储到 React
    // 实例上（比如 this.textInput）
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

### Render Props

术语 [“render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce) 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术

```react
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

组件是 React 代码复用的主要单元，但如何分享一个组件封装到其他需要相同 state 组件的状态或行为并不总是很容易。

记录鼠标位置:

```react
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        <h1>移动鼠标!</h1>
        <p>当前的鼠标位置是 ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}
```

如果想复用组件的鼠标位置使用render props：

```react
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>移动鼠标!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

关于 render prop 一个有趣的事情是你可以使用带有 render prop 的常规组件来实现大多数[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html) (HOC)。 例如，如果你更喜欢使用 `withMouse` HOC而不是 `` 组件，你可以使用带有 render prop 的常规 `` 轻松创建一个：

```react
// 如果你出于某种原因真的想要 HOC，那么你可以轻松实现
// 使用具有 render prop 的普通组件创建一个！
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```