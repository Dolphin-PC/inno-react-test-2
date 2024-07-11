### [과제] 숙련주차 과제 답

## 1. 추가하기 버튼을 클릭해도 추가한 아이템이 화면에 표시되지 않음.

### 원인

1. List.jsx는 `todos`라는 redux를 참조하여, todos 데이터를 출력하고 있음.
2. Form.jsx의 `onSubmitHandler`는 `todos` redux에 저장하는 로직이 없음.

### 해결

1. Form.jsx에서의 `onSubmitHandler`함수 내부에 `addTodo`함수를 dispatch를 통해 호출

```jsx
// src/features/todos/components/Form.tsx
const Form = () => {
  const id = nextId();

  const dispatch = useDispatch();

  const [todo, setTodo] = useState({
    id: 0,
    title: "",
    body: "",
    isDone: false,
  });

  const onChangeHandler = (event) => {
    const { name, value } = event.target;
    setTodo({ ...todo, [name]: value });
  };

  const onSubmitHandler = (event) => {
    event.preventDefault();
    if (todo.title.trim() === "" || todo.body.trim() === "") return;

    const payload = {
      id: id,
      title: todo.title,
      body: todo.body,
      isDone: false,
    };
    dispatch(addTodo(payload));

    setTodo({
      id: 0,
      title: "",
      body: "",
      isDone: false,
    });
  };

  ...
}
```

## 2. 추가하기 버튼 클릭 후 기존에 존재하던 아이템들이 사라짐.

### 원인

1. redux module `todos.js`, `ADD_TODO`reducer함수 로직 오류

- todos배열의 값이 `[action.payload]`으로 치환됨

```js
// src/redux/modules/todos.js
case ADD_TODO:
    return {
        ...state,
        todos: [action.payload],
    };
```

### 해결

1. todos의 배열에 추가하기 위해, spread연산자 활용

```js
// src/redux/modules/todos.js
case ADD_TODO:
    return {
        ...state,
        todos: [...state.todos, action.payload],
    };
```

## 3. 삭제 기능이 동작하지 않음.

### 원인

1. `todos reducers` 중 `DELETE_ADD`타입에 맞는`reducer`함수가 없음

```js
// src/redux/modules/todos.js
const todos = (state = initialState, action) => {
  switch (action.type) {
    case ADD_TODO: ...

    case TOGGLE_STATUS_TODO: ...

    case GET_TODO_BY_ID: ...
    default:
      return state;
  }
};
```

### 해결

1. DELETE_ADD타입의 reducer 함수 작성

```js
// src/redux/modules/todos.js
const todos = (state = initialState, action) => {
  switch (action.type) {
    case DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter((todo) => {
          return todo.id !== action.payload;
        }),
      };
    default:
      return state;
  }
};
```

## 4. 상세 페이지에 진입 하였을 때 데이터가 업데이트 되지 않음.

### 원인

1. Detail.jsx에서 참조하고 있는 `todo`는, `todos` store내부의 `todo`를 참고하고 있음

```jsx
// Detail.jsx
const todo = useSelector((state) => state.todos.todo);
```

```js
// src/redux/modules/todos.js
const initialState = {
  todos: [
    {
      id: "1",
      title: "리액트",
      body: "리액트를 배워봅시다",
      isDone: false,
    },
  ],
  todo: {
    id: "0",
    title: "",
    body: "",
    isDone: false,
  },
};
```

### 해결

1. Detail에서 참조하고 있는 `todo`값을 업데이트하기 위해서는, `GET_TODO_BY_ID` reducer를 실행하여 `todo`객체를 업데이트 해야함

```jsx
// Detail.jsx

const Detail = () => {
  const dispatch = useDispatch();
  const todo = useSelector((state) => state.todos.todo);

  const { id } = useParams();
  const navigate = useNavigate();

  useEffect(() => {
    dispatch(getTodoByID(id));
  }, []);

  ...
}
```
