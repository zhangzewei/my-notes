# 十分钟学会React的数据管理
> 作者：markzzw(zhangzewei) 日期：2024年1月16日
代码样例：[Codesandbox](https://codesandbox.io/p/sandbox/anything-sgxjhd?file=%2Fsrc%2FTodos.js%3A52%2C15)

读者将会通过本篇文章学会`React`的数据管理方案，即 `context` 和 `reducer` 的使用。 

本文所教学的React版本为`React 18`，将使用 react 进行一个简单的todolist的编写。

## useContext

在react中除了在父组件使用state进行数据管理，还可以将state通过context的方式注入，然后在子组件中通过 `useContext()` hook 获取，这样子就能够达到不需要使用props进行传递，而是在需要的组件中获取，这能够解决子组件层级太深，props透传多层的问题。

我们通过修改在[一小时学会React基础](./react-basic.md)中的todolist的代码来体验一下context的便捷。

**首先需要新建一个 `TodoProvider.js`**

```jsx
import { createContext, useState } from "react";

// 需要给外界调用useContext的地方使用
export const TodoContext = createContext(null);
// 在该组件内部进行state的管理
export default function TodoProvider({ children }) {
    const [todos, setTodos] = useState([]);
    useEffect(() => {
        // 从localstorage里获取存储的todos
        const todos = localStorage.getItem("todos");
        if (todos) {
            setTodos(JSON.parse(todos));
        }
    }, []);
    // 通过 TodoContext.Provider 的 value 将需要透传的数据传入
    return (
        <TodoContext.Provider
            value={{
                todos,
                setTodos,
            }}
        >
            {children}
        </TodoContext.Provider>
    );
}
```

**接下来将 `App.js` 改为**

```jsx
import "./styles.css";
import AddTodo from "./AddTodo";
import Todos from "./Todos";
import TodoProvider from "./TodoProvider";

export default function App() {
  return (
    <TodoProvider>
      <div className="container">
        <div className="row">
          <div className="page-header">
            <h1>TodoList</h1>
          </div>
          <AddTodo />
          <Todos />
        </div>
      </div>
    </TodoProvider>
  );
}
```
去除掉在父级元素中的 state 的相关代码，只留下html的结构，用 `<TodoProvider>` 包裹起来，使的 `<AddTodo />` 和 `<Todos />`在其作用域下。

**将 `AddTodo.js` 改为**

```jsx
import { useState } from "react";
import { TodoContext } from "./TodoProvider";
export default function AddTodo() {
  // 通过 useContext 拿到 setTodos 函数
  const { todos, setTodos } = useContext(TodoContext);
  const [inputValue, setInputValue] = useState("");

  const handleInput = (event) => {
    const value = event.target.value.trim();
    setInputValue(value);
  };
  // 修改 addTodo 函数
  const addTodo = () => {
    if (inputValue) {
        const newList = todos.concat({
            text: inputValue,
            status: "active",
        });
        setTodos(newList);
        localStorage.setItem("todos", JSON.stringify(newList));
        setInputValue("");
    }
  };
  return (
    <div className="input-group">
      <input
        type="text"
        className="form-control"
        placeholder="Search for..."
        value={inputValue}
        onChange={handleInput}
      />
      <span className="input-group-btn">
        <button onClick={addTodo} className="btn btn-default" type="button">
          Go!
        </button>
      </span>
    </div>
  );
}
```

**将 `Todos.js` 改为**

```jsx
import { useContext } from "react";
import { TODO_ACTIONS } from "./todo.store";
import { TodoContext } from "./TodoProvider";

export default function Todos() {
  // 通过 useContext 拿到 todos 和 setTodos
  const { todos, setTodos } = useContext(TodoContext);
  const handleDelete = (index) => {
    const newList = todos.filter((_, idx) => idx !== index);
    setTodos(newList);
  };

  const handleDone = (item) => {
    if (item.status === "active") {
      const newList = todos.map((item, index) => ({
        ...item,
        status: idx === index ? "done" : item.status,
      }));
      setTodos(newList);
    } else {
      const newList = todos.map((item, index) => ({
        ...item,
        status: idx === index ? "active" : item.status,
      }));
      setTodos(newList);
    }
  };
  return (
    <ul className="list-group">
      {
        todos.map((item, index) => (
          <li className="list-group-item list-item" key={index}>
            <span
              className={item.status}
            >
              {item.text}
            </span>
            <div className="btn-group" role="group">
              <button
                onClick={() => handleDelete(index)}
                type="button"
                className="btn btn-danger"
              >
                Delete
              </button>
              <button
                onClick={() => handleDone(item, index)}
                type="button"
                className="btn btn-primary"
              >
                {
                  item.status === "active" ? "Done" : "Undone"
                }
              </button>
            </div>
          </li>
        ))
      }
    </ul>
  );
}
```

经过修改，现在的 state 的管理全部就放在了 TodoProvider，并且子组件可以不通过props而是使用context去获取数据然后更新UI。

## Reducer

在多个事件处理程序中分布有许多状态更新的组件可能会变得不堪重负。对于这些情况，我们可以将组件外部的所有状态更新逻辑合并到一个称为reducer的函数中。

简单来说，就是目前todolist的业务逻辑处理代码分散在各个组件内部，如果我们需要修改业务逻辑，那么就要到对应的组件中去修改，如果这个业务逻辑处理很复杂，需要抽象更多组件，那么业务修改逻辑将会变得困难。

为了减少心智负担（查找需要修改的对应业务逻辑的组件），将业务逻辑都集中在一个函数内部去实现，就只需要在这一个函数中修改对应代码即可。

接下来我们就开始编写todolist的Reducer。

首先我们需要知道todolist的业务逻辑为：添加，删除，更改；在理想情况下我们不再从组件内部去处理所有的todolist，而是将需要处理的todo传给Reducer，让Reducer处理；所以上面三个业务逻辑分别改为：添加(新的todo)，删除(对应id的todo)，更改(对应id的todo的内容)。

**既然是Reducer，那么我们首先需要创建一个 `TodoReducer.js`**

```jsx
export const TODO_ACTIONS = {
  ADD: "add",
  DELETE: "delete",
  CHANGE: "change",
  GET_FROM_STORAGE: "getFromStorage",
};

export function todosReducer(currentState, action) {
  // action 会通过 dispatch 传送给 todosReducer 使用。
  let newState = currentState;
  switch (action.type) {
    case TODO_ACTIONS.ADD:
      newState = currentState.concat(action.payload);
      break;
    case TODO_ACTIONS.DELETE:
      newState = currentState.filter((todo) => todo.id !== action.payload.id);
      break;
    case TODO_ACTIONS.CHANGE:
      newState = currentState.map((todo) => {
        if (todo.id === action.payload.id) {
          return action.payload;
        }
        return todo;
      });
      break;
    case TODO_ACTIONS.GET_FROM_STORAGE:
      newState = action.payload;
      break;
    default:
      throw Error(`未知的行为代码：${action.type}`);
  }
  localStorage.setItem("todos", JSON.stringify(newState));
  return newState;
}
```

可以看到 `todosReducer` 的action.type是对应的 `TODO_ACTIONS` 的操作，使用策略模式，将不同的操作进行分隔，只需要传入对应的参数供逻辑代码使用即可，代码结构清楚明了。

**将 `TodoProvider.js` 改为**

```jsx
import { createContext, useEffect, useReducer } from "react";
import { todosReducer, TODO_ACTIONS } from "./TodoReducer";
export const TodoContext = createContext(null);

export default function TodoProvider({ children }) {
  // 通过 useReducer 创建 todo 的 Reducer
  // dispatch是一个能够把操作和操作数据传送给todosReducer函数的函数
  const [todos, dispatch] = useReducer(todosReducer, []);
  useEffect(() => {
    const todos = localStorage.getItem("todos");
    if (todos) {
      dispatch({
        type: TODO_ACTIONS.GET_FROM_STORAGE,
        payload: JSON.parse(todos),
      });
    }
  }, []);
    // 把修改函数改为 dispatch ，子组件可以通过 dispatch 将任务分发给 todosReducer 内部操作
  return (
    <TodoContext.Provider
      value={{
        todos,
        dispatch,
      }}
    >
      {children}
    </TodoContext.Provider>
  );
}
```

**将 `AddTodo.js` 改为**

```jsx
import { useContext, useState } from "react";
import { TODO_ACTIONS } from "./TodoReducer";
import { TodoContext } from "./TodoProvider";
export default function AddTodo() {
  // 在context中获取dispatch
  const { dispatch } = useContext(TodoContext);
  const [inputValue, setInputValue] = useState("");
  const handleInput = (e) => {
    const value = e.target.value.trim();
    setInputValue(value);
  };

  const handleAdd = () => {
    if (inputValue) {
      // 使用dispatch传递操作和操作的数据
      dispatch({
        type: TODO_ACTIONS.ADD,
        payload: {
          id: new Date().toString(), // 绑定id用于删除和修改操作
          text: inputValue,
          status: "active",
        },
      });
      setInputValue("");
    }
  };

  return (
    <div className="input-group">
      <input
        type="text"
        className="form-control"
        placeholder="Add todo ..."
        // 绑定state上的value
        value={inputValue}
        // 绑定函数
        onChange={handleInput}
      />
      <span className="input-group-btn">
        <button
          // 绑定函数
          onClick={handleAdd}
          className="btn btn-default"
          type="button"
        >
          Go!
        </button>
      </span>
    </div>
  );
}
```

**将 `Todos.js` 改为**

```jsx
import { useContext } from "react";
import { TODO_ACTIONS } from "./TodoReducer";
import { TodoContext } from "./TodoProvider";

export default function Todos() {
  // 从 context 中获取 todos 和 dispatch
  const { todos, dispatch } = useContext(TodoContext);
  const handleDelete = (id) => {
    // 将原有逻辑修改为通过dispatch传递
    dispatch({
      type: TODO_ACTIONS.DELETE,
      payload: {
        id,
      },
    });
  };

  const handleDone = (item) => {
    // 将原有逻辑修改为通过dispatch传递
    let newItem = item;
    if (item.status === "active") {
      newItem.status = "done";
    } else {
      newItem.status = "active";
    }
    dispatch({
      type: TODO_ACTIONS.CHANGE,
      payload: newItem,
    });
  };
  return (
    <ul className="list-group">
      {
        // 使用 {} 能够在jsx中书写js表达式，通过map返回html数组
        todos.map((item, index) => (
          // key 是返回数组html的必须参数，它能够帮助react进行数组html的更新，key必须是唯一的
          <li className="list-group-item list-item" key={index}>
            <span
              className={
                // 使用status对其样式进行不一样的渲染
                item.status
              }
            >
              {item.text}
            </span>
            <div className="btn-group" role="group">
              <button
                onClick={() => handleDelete(item.id)}
                type="button"
                className="btn btn-danger"
              >
                Delete
              </button>
              <button
                onClick={() => handleDone(item)}
                type="button"
                className="btn btn-primary"
              >
                {
                  // 通过status进行文案的重新渲染
                  item.status === "active" ? "Done" : "Undone"
                }
              </button>
            </div>
          </li>
        ))
      }
    </ul>
  );
}
```

## 结束
至此，react中的数据管理方案已经介绍完毕，总体思路为：
1. 通过 context 替换 state 管理全局数据
2. 通过 reducer 对全局 state 进行统一处理