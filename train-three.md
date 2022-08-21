### lesson3-给组件添加响应式

![image-20200908160328366](https://tva1.sinaimg.cn/large/007S8ZIlly1gijayj5424j30z80u07wh.jpg)



给组件添加响应式有以下三种办法：

😎 [observer HOC](https://mobx-react.js.org/observer-hoc)

🤓 [observer component](https://mobx-react.js.org/observer-component)

🧐 [useObserver hook](https://mobx-react.js.org/observer-hook)

```jsx
import { observable } from 'mobx'
import { Observer, useObserver, observer } from 'mobx-react' // 6.x or mobx-react-lite@1.4.0
import ReactDOM from 'react-dom'

const person = observable({
  name: 'John',
})

// named function is optional (for debugging purposes)
const P1 = observer(function P1({ person }) {
  return <h1>{person.name}</h1>
})

const P2 = ({ person }) => <Observer>{() => <h1>{person.name}</h1>}</Observer>

const P3 = ({ person }) => {
  return useObserver(() => <h1>{person.name}</h1>)
}

ReactDOM.render(
  <div>
    <P1 person={person} />
    <P2 person={person} />
    <P3 person={person} />
  </div>,
)

setTimeout(() => {
  person.name = 'Jane'
}, 1000)
```



##### observer HOC

注意：mobx-react与mobx-react-lite中都有observer，但是后者不支持类组件。

以下是参数，如果组件是函数式组件，可以使用options，类组件则没有。

```tsx
observer<P>(baseComponent: React.FC<P>, options?: IObserverOptions): React.FC<P>

interface IObserverOptions {
    // Pass true to use React.forwardRef over the inner component. It's false by the default.
    forwardRef?: boolean
}
```

observer把组件转换成响应式组件，它可以自动追踪哪个可观察量被使用了以及当值改变的时候自动重新渲染这个组件。

```jsx
import { observer, useLocalStore } from 'mobx-react' // 6.x or mobx-react-lite@1.4.0

export const Counter = observer<Props>(props => {
  const store = useLocalStore(() => ({
    count: props.initialCount,
    inc() {
      store.count += 1
    },
  }))

  return (
    <div>
      <span>{store.count}</span>
      <button onClick={store.inc}>Increment</button>
    </div>
  )
})
```

observer内部使用了React.memo，所以使用的时候我们可以不用自己再封装memo，但值得注意的是，observer内部封装的memo只负责默认的浅比较，因为mobx-react认为，observed组件很少需要基于复杂的props进行更新渲染。以下是mobx-react-lite中的源码部分截图，参考行61-63的注释。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gircs01emmj30ut0u0e81.jpg" alt="image-20200915151056480" style="zoom:50%;" />



##### Observer component

```ts
<Observer>{renderFn}</Observer>
```

接收一个匿名无参数的函数作为children，并且返回React元素。

```jsx
import { Observer, useLocalStore } from 'mobx-react' // 6.x or mobx-react-lite@1.4.0

export function ObservePerson() {
  const person = useLocalStore(() => ({ name: 'John' }))
  return (
    <div>
      {person.name} <i>I will never change my name</i>
      <div>
        <Observer>{() => <div>{person.name}</div>}</Observer>
        <button onClick={() => (person.name = 'Mike')}>
          I want to be Mike
        </button>
      </div>
    </div>
  )
}
```

注意：只能使得它自己的返回组件是响应式的，如果你里面还嵌套了别的组件，那这个里面的组件得靠自己变成响应式~

看下面的例子：

```jsx
import { Observer } from 'mobx-react' // 6.x or mobx-react-lite@1.4.0

function ObservePerson() {
  return (
    <Observer>
      {() => (
        <GetStore>{store => <div className="self">{store.wontSeeChangesToThis}</div>}</GetStore>
      )}
    </Observer>
  )
}
```

在上面这个例子里，这个className为div就不是响应式，如果想变成响应式，需要再额外添加，比如说像下面这样：

```jsx
import { Observer } from 'mobx-react' // 6.x or mobx-react-lite@1.4.0

function ObservePerson() {
  return (
    <GetStore>
      {store => (
        <Observer>{() => <div>{store.changesAreSeenAgain}</div>}</Observer>
      )}
    </GetStore>
  )
}
```

##### useObserver hook

函数签名：

```tsx
useObserver<T>(fn: () => T, baseComponentName = "observed", options?: IUseObserverOptions): T

interface IUseObserverOptions {
    // optional custom hook that should make a component re-render (or not) upon changes
    useForceUpdate: () => () => void
}
```



这个自定义hook方法只存在于mobx-react-lite，或者是mobx-react@6。

这个hook方法也可以使组件变得响应式，但是关于一些优化兼容手段如memo或者forwardRef，则需要你自己手动添加。

```jsx
import { useObserver, useLocalStore } from 'mobx-react' // 6.x or mobx-react-lite@1.4.0

function Person() {
  const person = useLocalStore(() => ({ name: 'John' }))
  return useObserver(() => (
    <div>
      {person.name}
      <button onClick={() => (person.name = 'Mike')}>No! I am Mike</button>
    </div>
  ))
}
```

这个方法有个优点，就是任何的hook改变observable，组件都不会重复渲染。

这个方法有个缺点，就是就算你是在组件的某一部分使用，但是却能引起组件的全部更新~，所以慎用，如果想要精细控制的话，还是要选择<Observer/>或者useForceUpdate。
