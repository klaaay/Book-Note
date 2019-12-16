# everything-you-should-know-about-react(关于 React 你需要知道的一切)

- [everything-you-should-know-about-react(关于 React 你需要知道的一切)](#everything-you-should-know-about-react%e5%85%b3%e4%ba%8e-react-%e4%bd%a0%e9%9c%80%e8%a6%81%e7%9f%a5%e9%81%93%e7%9a%84%e4%b8%80%e5%88%87)
  - [Design Patterns and Techniques](#design-patterns-and-techniques)
    - [Conditional in JSX(JSX 中的状态分支)](#conditional-in-jsxjsx-%e4%b8%ad%e7%9a%84%e7%8a%b6%e6%80%81%e5%88%86%e6%94%af)
    - [Async Nature of setState(setState 函数的异步性)](#async-nature-of-setstatesetstate-%e5%87%bd%e6%95%b0%e7%9a%84%e5%bc%82%e6%ad%a5%e6%80%a7)
    - [Dependency Injection(依赖注入)](#dependency-injection%e4%be%9d%e8%b5%96%e6%b3%a8%e5%85%a5)
    - [Event Handler(事件处理)](#event-handler%e4%ba%8b%e4%bb%b6%e5%a4%84%e7%90%86)
    - [Presentational vs Container](#presentational-vs-container)
    - [Reaching Into A Component (深入某个组件内部)](#reaching-into-a-component-%e6%b7%b1%e5%85%a5%e6%9f%90%e4%b8%aa%e7%bb%84%e4%bb%b6%e5%86%85%e9%83%a8)
  - [Anti - Patterns](#anti---patterns)
    - [Props In Initial State(根据 props 去初始化 state)](#props-in-initial-state%e6%a0%b9%e6%8d%ae-props-%e5%8e%bb%e5%88%9d%e5%a7%8b%e5%8c%96-state)
    - [Mutating State(不使用 setState 去操作 state)](#mutating-state%e4%b8%8d%e4%bd%bf%e7%94%a8-setstate-%e5%8e%bb%e6%93%8d%e4%bd%9c-state)

https://hateonion.me/books/react-bits-cn/patterns/22.event-handlers.html

## Design Patterns and Techniques

### Conditional in JSX(JSX 中的状态分支)

存在控制的三元符使用&&符号的表达式简写会是一种更好的实践

```javascript
const sampleComponent = () => {
  return isTrue ? <p>True!</p> : <none />;
};
const sampleComponent = () => {
  return isTrue && <p>True!</p>;
};
```

有很多分支的状态控制，提取函数并考虑使用 swith 或者使用立即执行函数 hack

```javascript
// Y soo many ternary??? :-/
const sampleComponent = () => {
  return (
    <div>
      {flag && flag2 && !flag3 ? (
        flag4 ? (
          <p>Blah</p>
        ) : flag5 ? (
          <p>Meh</p>
        ) : (
          <p>Herp</p>
        )
      ) : (
        <p>Derp</p>
      )}
    </div>
  );
};

const sampleComponent = () => {
  return (
    <div>
      {(() => {
        if (flag && flag2 && !flag3) {
          if (flag4) {
            return <p>Blah</p>;
          } else if (flag5) {
            return <p>Meh</p>;
          } else {
            return <p>Herp</p>;
          }
        } else {
          return <p>Derp</p>;
        }
      })()}
    </div>
  );
};

const sampleComponent = () => {
  const basicCondition = flag && flag2 && !flag3;
  if (!basicCondition) return <p>Derp</p>;
  if (flag4) return <p>Blah</p>;
  if (flag5) return <p>Meh</p>;
  return <p>Herp</p>;
};
```

### Async Nature of setState(setState 函数的异步性)

**概述**

> 在一般情况下 React 考虑到性能的问题，可能会将 state 更新合并成一次更新，但是在 addEventListener, setTimeout 函数或者发出 AJAX call 的时候的情况下，这些调用不存在于 React 本身的上下文中，所以 React 并不能够像控制其他存在与其上下文中的函数一样，将多次 state 更新合并成一次，React 在此时采用的策略就是及时更新，确保在这些函数执行之后的其他代码能拿到正确的数据。  
> 所以大部分情况下 setState 表现出来的性状是异步的，在特殊情况下是同步的。

**解决方案**

> setState 提供了第二个参数 callback 函数可以获取到 state 的最新状态

**源码解读**

```javascript
// 其实setState作为一个函数，本身是同步的。只是因为在setState的内部实现中，使用了React updater的enqueueState 或者 enqueueCallback方法，才造成了异步。
ReactComponent.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === "object" ||
      typeof partialState === "function" ||
      partialState == null,
    "setState(...): takes an object of state variables to update or a " +
      "function which returns an object of state variables."
  );
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, "setState");
  }
};
// 而updater的这两个方法，又和React底层的Virtual Dom(虚拟DOM树)的diff算法有紧密的关系，所以真正决定同步还是异步的其实是Virtual DOM的diff算法。
```

### Dependency Injection(依赖注入)

可以使用 HOC(高阶组件的)方式注入你需要的额外数据

```javascript
// inject.jsx
var title = "React Dependency Injection";
export default function inject(Component) {
  return class Injector extends React.Component {
    render() {
      return <Component {...this.state} {...this.props} title={title} />;
    }
  };
}

// Title.jsx
export default function Title(props) {
  return <h1>{props.title}</h1>;
}

// Header.jsx
import inject from "./inject.jsx";
import Title from "./Title.jsx";

var EnhancedTitle = inject(Title);
export default function Header() {
  return (
    <header>
      <EnhancedTitle />
    </header>
  );
}
```

### Event Handler(事件处理)

**概述**

> 使用 function 的写法, 会在 function 初始化时生成一个 this. 比如我们在\_handleButtonClick 里面使用 this, 此时的 this 是\_handleButtonClick 生成出来的, 和 Switcher 这个 class 的 this 没有任何关系, 如果我想访问类似于 this.props 或者 this.state 这样的对象, 代码便会报错.

```javascript
lass Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
  }
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }

  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
    // 将导致
    // Uncaught TypeError: Cannot read property 'state' of null
  }
}
```

**解决方法**

> 在 constructor 函数中使用 bind， 使用箭头函数自动绑定

```javascript
constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
    this._buttonClick = this._handleButtonClick.bind(this);
  }

const _handleButtonClick = ()=>{
    //....
}
```

### Presentational vs Container

**概述**

> UI 和业务逻辑和数据混杂在一起

**解决办法**

> 我们将组件拆分成容器(container)组件和 UI(presentation)组件

**容器组件**

> 容器组件关心数据(包括数据的格式和数据的来源等). 容器组件关心具体的业务逻辑, 它接收数据并将数据整理成我们的 UI 组件需要的格式传递给 UI 组件. 我们常使用高阶组件去建立容器组件. 一般情况下, 容器组件的 render 方法里面包含的只会是 UI 组件.

**UI 组件**

> UI 组件关心组件展示出来是什么样子. UI 组件一般由基本的 html 标签为基础, 用以在页面上展示. 理想的 UI 组件应该被设计为没有外部依赖的组件. 常用的实现是使用没有内部 state 的无状态组件(stateless function)

**总结**

> 容器组件封装了封装了业务逻辑, 并且可以灵活的讲不同的数据注入不同的 UI 组件中, 这是使用容器组件带来明显的好处. 常见使用容器组件的方法是我们不去在容器组件内部规定哪个 UI 组件将被渲染, 而是建立一个接收一个 UI 组件的函数, 这样非常灵活, 让我们的容器组件可以包裹任意 UI 组件

```javascript
export default function(Component) {
  return class Container extends React.Component {
    render() {
      return <Component />;
    }
  };
}
```

### Reaching Into A Component (深入某个组件内部)

**概述**

> 通过父组件去访问子组件. 比如一个能自动 focus 的输入框(通过父组件控制自动 focus)

**方法**

> Ref 和 useRef

## Anti - Patterns

### Props In Initial State(根据 props 去初始化 state)

**概述**

> 使用 props 去在 getInitialState 中生成初始 state(或者在 constructor 中初始化)很容易导致多个数据源的问题, 也会给使用者带来这样的疑问: 我们的真正的数据源到底来自哪? 这是因为 getInitialState 只在组件第一次初始化的时候被调用一次.

**问题**

> 有可能组件的 props 发生了改变但是组件却没有被更新

**坏的实践**

```javascript
class SampleComponent extends Component {
  // constructor function (or getInitialState)
  constructor(props) {
    super(props);
    this.state = {
      flag: false,
      inputVal: props.inputValue
    };
  }

  render() {
    return <div>{this.state.inputVal && <AnotherComponent />}</div>;
  }
}
```

**好的实践**

```javascript
class SampleComponent extends Component {
  // constructor function (or getInitialState)
  constructor(props) {
    super(props);
    this.state = {
      flag: false
    };
  }

  render() {
    return <div>{this.props.inputValue && <AnotherComponent />}</div>;
  }
}
```

### Mutating State(不使用 setState 去操作 state)

**导致的问题**

1. 在 state 改变时组件不会重新渲染.
2. 在未来某个时候如果通过 setState 改变了 state, 那么这次未通过 setState 去改变的 state 将会同样生效.
