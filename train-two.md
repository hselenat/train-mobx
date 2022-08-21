### lesson2-MobX API 概念

#### Observable state(可观察的状态)

MobX 为现有的数据结构(如对象，数组和类实例)添加了可观察的功能。 通过使用 [@observable](http://cn.mobx.js.org/refguide/observable-decorator.html) 装饰器(ES.Next)来给你的类属性添加注解就可以简单地完成这一切。

```jsx
import {observable, computed} from "mobx";

class TodoList {
  @observable todos = [
    {id: "0", title: "任务0", finished: false},
    {id: "1", title: "任务1", finished: false}
  ];
  @computed get unfinishedTodoCount() {
    return this.todos.filter(todo => !todo.finished).length;
  }
}
```



#### Computed values(计算值)

定义在相关数据发生变化时自动更新的值。如上段代码，当添加了一个新的todo或者某个todo的 `finished` 属性发生变化时，MobX 会确保 `unfinishedTodoCount` 自动更新。



#### Reactions(反应)

Reactions 和计算值很像，但它不是产生一个新的值，而是会产生一些副作用，比如打印到控制台、网络请求、递增地更新 React 组件树以修补DOM、等等。

##### React 组件

如果你用 React 的话，可以把你的(无状态函数)组件变成响应式组件，方法是在组件上添加 [`observer`](http://cn.mobx.js.org/refguide/observable-decorator.html) 函数/ 装饰器. `observer`由 `mobx-react` 包提供的。

```jsx
import {observer} from "mobx-react";

@observer
class TodoListView extends React.Component {
  render() {
    const {todos, unfinishedTodoCount} = this.props.todoList;
    return (
      <div>
        <ul>
          {todos.map(todo => (
            <TodoView todo={todo} key={todo.id} />
          ))}
        </ul>
        Tasks left: {unfinishedTodoCount}
      </div>
    );
  }
}

const TodoView = observer(({todo}) =>
    <li>
        <input
            type="checkbox"
            checked={todo.finished}
            onClick={() => todo.finished = !todo.finished}
        />{todo.title}
    </li>
)
const store = new TodoList();

ReactDOM.render(
  <TodoListView todoList={store} />,
  document.getElementById("root")
);
```



##### 自定义 reactions

使用[`autorun`](http://cn.mobx.js.org/refguide/autorun.html)、[`reaction`](http://cn.mobx.js.org/refguide/reaction.html) 和 [`when`](http://cn.mobx.js.org/refguide/when.html) 函数即可简单的创建自定义 reactions，以满足你的具体场景。

例如，每当 `unfinishedTodoCount` 的数量发生变化时，下面的 `autorun` 会打印日志消息:

```javascript
autorun(() => {
    console.log("Tasks left: " + todos.unfinishedTodoCount)
})
```



##### MobX 会对什么作出响应?

*MobX 会对在执行跟踪函数期间读取的任何现有的可观察属性做出反应*。



#### Actions(动作)

任何应用都有动作。动作是任何用来修改状态的东西。

使用MobX你可以在代码中显式地标记出动作所在的位置。 动作可以有助于更好的组织代码。 建议在任何更改 observable 或者有副作用的函数上使用动作。 结合开发者工具的话，动作还能提供非常有用的调试信息。 注意: 当启用**严格模式**时，需要强制使用 `action`，参见 `enforceActions`。

```jsx
import {configure} from "mobx";

// 不允许在动作外部修改状态
configure({enforceActions: "observed"});
```



#### `enforceActions`

也被称为“严格模式”。

在严格模式下，不允许在 [`action`](https://cn.mobx.js.org/refguide/action.html) 外更改任何状态。 可接收的值:

- `"never"` (默认): 可以在任意地方修改状态
- `"observed"`: 在某处观察到的所有状态都需要通过动作进行更改。在正式应用中推荐此严格模式。
- `"always"`: 状态始终需要通过动作来更新(实际上还包括创建)。