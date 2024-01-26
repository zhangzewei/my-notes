# React Router 六步成形
> 作者：markzzw(zhangzewei) 日期：2024年1月25日
代码样例：[Codesandbox](https://codesandbox.io/p/sandbox/todolist-zttf9d?file=%2Fsrc%2Frouter.js%3A15%2C9)

读者将会通过本篇文章对`React Router`的基础有一定的认识，能够使用其基本的特性和功能进行小规模项目或者组件的开发。

本文所教学的React版本为`React 18`，React Router版本为`6.21`，将使用简单的 todolist 进行讲解。

## 安装npm包
```
npm install react-router-dom
```
## 创建根路由
创建文件 `router.js`, 这个文件是代表了整个系统路由的配置，我们先将 App 组件设置为根路由，这样子我们访问根路由就能够渲染App组件
```jsx
import App from "./App";
export const routerConfig = [
  {
    path: "/",
    element: <App />,
  },
];
```
然后我们将 `index.js` 修改为
```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router-dom";// 引入router的组件
import { routerConfig } from "./router"; // 引入路由配置

const rootElement = document.getElementById("app");
const root = createRoot(rootElement);

const router = createBrowserRouter(routerConfig); // 使用我们的路由配置创建路由

root.render(
  <StrictMode>
    <RouterProvider router={router} /> {/* 将路由配置传入到 RouterProvider 中 */}
  </StrictMode>
);
```
现在刷新界面，就能够看到和之前一样的 todo list 的界面了。

## 新增子路由
接下来修改 `router.js`:

```jsx
import App from "./App";
import AddTodo from "./AddTodo";
import Todos from "./Todos";
export const routerConfig = [
  {
    path: "/",
    element: <App />,
    children: [ // 子路由配置
      {
        path: "/",
        element: <Todos />,
      },
      {
        path: "/create",
        element: <AddTodo />,
      },
    ],
  },
];
```
在上面的代码中，发现了根路由出现了两次，React Router会根据层级一层层的匹配，所以当根路由时，React Router会先匹配 App 组件，然后是 Todos 组件，我们将给 App.js 中添加一个 `<Outlet />` 组件，这个组件是路由的一个占位符，即这个组件将会被渲染为最终匹配的组件，也就是说 React Router 先匹配到 App 然后里面有一个 outlet 然后 outlet 将会渲染为 Todos。
然后将 App.js 修改为：
```jsx
import { TodoProvider } from "./TodoProvider";
import { Outlet } from "react-router-dom";
import "./styles.css";
export default function App() {
  return (
    <TodoProvider>
      <Outlet />
    </TodoProvider>
  );
}
```
现在我们来访问根路由，就只会出现 Todos 组件，然后访问 '/create' 就只会出现 AddTodo 组件。

## 路由跳转
目前我们只能通过更改浏览器的路劲来访问对应的页面，接下来我们就给他加上跳转链接，让我们能通过点击某个链接进行页面的跳转，我们先给 Todos 组件添加一个按钮，点击之后就能跳转到 AddTodo 组件，这里需要使用到 `<Link to='path' />` 组件。

### Todos.js
```jsx
import { useContext } from "react";
import { Link } from "react-router-dom"; // 引入 Link 组件
import { TodoContext } from "./TodoProvider";
import { TODO_ACTIONS } from "./TodoReducer";

export default function Todos() {
  const { todos, dispatch } = useContext(TodoContext);
  const deleteTodo = (id) => {
    dispatch({
      type: TODO_ACTIONS.delete,
      payload: {
        id,
      },
    });
  };
  const handleDone = (item) => {
    let newItem = item;
    if (item.status === "active") {
      newItem.status = "done";
    } else {
      newItem.status = "active";
    }
    dispatch({
      type: TODO_ACTIONS.change,
      payload: newItem,
    });
  };
  return (
    <div>
      <h2>
        <span>Todo List</span>
        {/* 添加一个跳转链接，to 表示需要跳转到的组件的对应的链接 */}
        <Link className="btn btn-link" to="/create">
          Add Todo
        </Link>
      </h2>
      <ul className="list-group">
        {todos.map((item) => (
          <li className="list-group-item lits-item" key={item.id}>
            <span className={item.status}>{item.text}</span>
            <div className="btn-group" role="group" aria-label="...">
              <button
                onClick={() => deleteTodo(item.id)}
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
                {item.status === "active" ? "Done" : "Undone"}
              </button>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
现在点击 Add Todo 按钮就能够跳转到 AddTodo 组件了，但是现在跳转过去进行了添加操作，页面却不能够回到之前的页面，我们期望能够回到展示Todo list的页面，所以接下来我们将会更改 AddTodo 组件。

由于在AddTodo里面我们是在添加按钮的click事件中进行的数据添加，然后需要在事件处理的函数里面进行跳转，所以就不再是用 `<Link>` 组件跳转，这里我们可以使用React Router提供的hook `useNavigate` 进行跳转。

### AddTodo.js
```jsx
import { useContext, useState } from "react";
import { Link, useNavigate} from "react-router-dom";
import { TodoContext } from "./TodoProvider";
import { TODO_ACTIONS } from "./TodoReducer";
export default function AddTodo() {
  const navigate = useNavigate(); // 获取navigate实例
  const { dispatch } = useContext(TodoContext);
  const [inputValue, setInputValue] = useState("");
  const handleInput = (event) => {
    const value = event.target.value.trim();
    setInputValue(value);
  };
  const addTodo = () => {
    if (inputValue && confirm("Comfirm to add one todo")) {
      dispatch({
        type: TODO_ACTIONS.add,
        payload: {
          id: new Date().toString(),
          text: inputValue,
          status: "active",
        },
      });
      navigate("/"); // 通过navigate进行跳转
    }
  };
  return (
    <div>
      <h2>
        <span>Add One Todo</span>
        <Link className="btn btn-link" to="/">
          Back
        </Link>
      </h2>
      <div className="input-group">
        <input
          type="text"
          className="form-control"
          placeholder="Add some todo ..."
          value={inputValue}
          onChange={handleInput}
        />
        <span className="input-group-btn">
          <button onClick={addTodo} className="btn btn-default" type="button">
            Add
          </button>
        </span>
      </div>
    </div>
  );
}
```

## 获取路径参数
在这里我们需要新增一个 Todo Detail 页面用来展示一个单独的 Todo 的详情，通过在路径中放入todo的id来进行数据的获取。

新增页面的第一步，新增路由：
### router.js
```jsx
import App from "./App";
import AddTodo from "./AddTodo";
import Todos from "./Todos";
import TodoDetail from "./TodoDetail";
export const routerConfig = [
  {
    path: "/",
    element: <App />,
    children: [
      {
        path: "/",
        element: <Todos />,
      },
      {
        path: "/create",
        element: <AddTodo />,
      },
      {
        path: "/detail/:id", // 冒号代表参数的意思
        element: <TodoDetail />,
      },
    ],
  },
];
```
新增 TodoDetail 页面
### TodoDetail.js
```jsx
import { useContext } from "react";
import { useParams } from "react-router-dom";
import { TodoContext } from "./TodoProvider";

export default function TodoDetail() {
  const { todos } = useContext(TodoContext);
  const { id } = useParams();
  const todo = todos.find((todo) => todo.id === id);
  if (todo) {
    return (
      <div>
        <h2>Todo Detail</h2>
        <ul className="list-group">
          <li className="list-group-item lits-item">
            <strong>Todo ID</strong>
            {todo.id}
          </li>
          <li className="list-group-item lits-item">
            <strong>Todo Text</strong>
            {todo.text}
          </li>
        </ul>
      </div>
    );
  }
  return <h2>Can not match any todo with id: {id}</h2>;
}
```

现在就能访问 '/detail/testid'，但是现在只会渲染找不到匹配的提示语句，因为id不匹配，可是我们又不可能去记住每一个todo的id，所以我们还是需要使用到 <Link> 组件，转过头来修改 Todos 组件。
### Todos.js
```jsx
import { useContext } from "react";
import { Link } from "react-router-dom";
import { TodoContext } from "./TodoProvider";
import { TODO_ACTIONS } from "./TodoReducer";

export default function Todos() {
  const { todos, dispatch } = useContext(TodoContext);
  const deleteTodo = (id) => {
    dispatch({
      type: TODO_ACTIONS.delete,
      payload: {
        id,
      },
    });
  };
  const handleDone = (item) => {
    let newItem = item;
    if (item.status === "active") {
      newItem.status = "done";
    } else {
      newItem.status = "active";
    }
    dispatch({
      type: TODO_ACTIONS.change,
      payload: newItem,
    });
  };
  return (
    <div>
      <h2>
        <span>Todo List</span>
        <Link className="btn btn-link" to="/create">
          Add Todo
        </Link>
      </h2>
      <ul className="list-group">
        {todos.map((item) => (
          <li className="list-group-item lits-item" key={item.id}>
            {/* to 参数传入对应的路径，用id组合起来 */}
            <Link to={`/detail/${item.id}`} className={item.status}>
              {item.text}
            </Link>
            <div className="btn-group" role="group" aria-label="...">
              <button
                onClick={() => deleteTodo(item.id)}
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
                {item.status === "active" ? "Done" : "Undone"}
              </button>
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}
```
这样我们就能够通过点击每一个 todo 的文本跳转到对应的页面了。

## 报错页面
当我们输入一个与router config不匹配的路径时，会出现一个404的页面，这个页面是React router自带的默认报错页面，我们也可以自定义报错页面，用来展示一些我们需要展示的信息，因为默认的报错页面没有返回按钮，我们就在报错页面添加一个返回主页的按钮，方便用户在遇到报错的时候回到正确的页面。

### ErrorPage.js
```jsx
import { Link } from "react-router-dom";
import { useRouteError } from "react-router-dom";

export default function ErrorPage() {
  const error = useRouteError();
  return (
    <div className="container">
      <h1>Oops!</h1>
      <p>Sorry, an unexpected error has occurred.</p>
      <p>
        <i>{error.statusText || error.message}</i>
      </p>
      <Link to="/" className="btn btn-primary">
        Back To Home
      </Link>
    </div>
  );
}
```
然后将 error page 添加到router config中
### router.js
```jsx
import App from "./App";
import AddTodo from "./AddTodo";
import Todos from "./Todos";
import TodoDetail from "./TodoDetail";
import ErrorPage from "./ErrorPage";
export const routerConfig = [
  {
    path: "/",
    element: <App />,
    errorElement: <ErrorPage />, // 错误页面
    children: [
      {
        path: "/",
        element: <Todos />,
      },
      {
        path: "/create",
        element: <AddTodo />,
      },
      {
        path: "/detail/:id",
        element: <TodoDetail />,
      },
    ],
  },
];
```
此时，我们输入一个错误的路径，就能够看到我们自定义的错误界面了。


