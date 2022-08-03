# To-Do List with React App

[![yarn version](https://badgen.net/npm/v/yarn)](https://classic.yarnpkg.com/en/)
![React]("https://img.shields.io/badge/React-61DAFB?style=flat&logo=React&logoColor=white")

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Overview

To-Do List는 작업을 관리하는 매우 간단한 앱입니다. 쇼핑 목록이나 작업 태스크 등 모든 체크리스트로 사용할 수 있습니다.

App을 만드는 과정에서 아래의 React 개념들을 활용합니다.

1. styled-components를 통한 컴포넌트 스타일링

2. Context API를 사용한 전역상태 관리

3. 배열상태 관리

## Create Components

필요한 컴포넌트

- TodoTemplate

- TodoHead

- TodoList

- TodoItem

- TodoCreate

## Context API를 활용한 전역상태 관리

`useReducer`를 사용하여 상태를 관리하는 TodoProvider 라는 컴포넌트를 만든다.
`state`와 `dispatch`를 Context를 통하여 다른 컴포넌트에서 바로 사용 할 수 있게 해주는데, 하나의 Context를 만들어서 `state`와 `dispatch`를 함께 넣어주는 대신에, 두개의 Context를 만들어서 따로 따로 넣어준다.

```js
import React, { useReducer, createContext } from 'react';

...

function todoReducer(state, action) {
  switch (action.type) {
    case 'CREATE':
      return state.concat(action.todo);
    case 'TOGGLE':
      return state.map((todo) =>
        todo.id === action.id ? { ...todo, done: !todo.done } : todo
      );
    case 'REMOVE':
      return state.filter((todo) => todo.id !== action.id);
    default:
      throw new Error(`Unhandled action type: ${action.type}`);
  }
}

const TodoStateContext = createContext();
const TodoDispatchContext = createContext();

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialTodos);
  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        {children}
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}
```

### 커스텀 Hook 만들기

컴포넌트에서 `useContext`를 직접 사용하는 대신에, `useContext`를 사용하는 커스텀 Hook을 만들어서 내보내준다.

```js
import React, { useReducer, createContext, useContext } from 'react';

...

export function useTodoState() {
  return useContext(TodoStateContext);
}

export function useTodoDispatch() {
  return useContext(TodoDispatchContext);
}
```

### nextId 값 관리하기

`nextId`값을 위한 Context를 만들어준다. 여기서 `nextId`가 의미하는 값은 새로운 항목을 추가 할 때 사용 할 고유 ID 다. 이 값은 `useRef`를 사용하여 관리해준다.

```js
import React, { useReducer, createContext, useContext, useRef } from 'react';

...

const TodoNextIdContext = createContext();

export function TodoProvider({ children }) {
  const [state, dispatch] = useReducer(todoReducer, initialTodos);
  const nextId = useRef(5);

  return (
    <TodoStateContext.Provider value={state}>
      <TodoDispatchContext.Provider value={dispatch}>
        <TodoNextIdContext.Provider value={nextId}>
          {children}
        </TodoNextIdContext.Provider>
      </TodoDispatchContext.Provider>
    </TodoStateContext.Provider>
  );
}

...

export function useTodoNextId() {
  return useContext(TodoNextIdContext);
}
```

### 커스텀 Hook 에서 에러 처리

`seTodoState`, `useTodoDispatch`, `useTodoNextId` Hook을 사용하려면, 해당 컴포넌트가 TodoProvider 컴포넌트 내부에 렌더링되어 있어야 한다.
TodoProvider로 감싸져있지 않다면 에러를 발생시키도록 한다.

```js
...

export function useTodoState() {
  const context = useContext(TodoStateContext);
  if (!context) {
    throw new Error('Cannot find TodoProvider');
  }
  return context;
}

export function useTodoDispatch() {
  const context = useContext(TodoDispatchContext);
  if (!context) {
    throw new Error('Cannot find TodoProvider');
  }
  return context;
}

export function useTodoNextId() {
  const context = useContext(TodoNextIdContext);
  if (!context) {
    throw new Error('Cannot find TodoProvider');
  }
  return context;
}
```

### 컴포넌트 TodoProvider로 감싸기

프로젝트 모든 곳에서 Todo 관련 Context들을 사용 할 수 있도록, App 컴포넌트에서 TodoProvider를 불러와서 모든 내용을 TodoProvider로 감싸준다.

```js
import { TodoProvider } from './TodoContext';

...

function App() {
  return (
    <TodoProvider>
      <GlobalStyle />
      <TodoTemplate>
        <TodoHead />
        <TodoList />
        <TodoCreate />
      </TodoTemplate>
    </TodoProvider>
  );
}
```

## 기능 구현

Context와 연동을 하여 기능을 구현한다. Context에 있는 `state`를 받아와서 렌더링을 하고, 필요한 상황에는 특정 액션을 `dispatch` 한다.

### TodoHead

남은 할 일은 `done` 값이 false인 항목들의 개수를 화면에 보여준다.
날짜는 Date의 [toLocaleString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toLocaleString) 함수를 사용한다.

```js
...

function TodoHead() {
  const todos = useTodoState();
  const undoneTasks = todos.filter((todo) => !todo.done);

  const today = new Date();
  const dateString = today.toLocaleDateString('ko-KR', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });
  const dayName = today.toLocaleDateString('ko-KR', { weekday: 'long' });

  return (
    <TodoHeadBlock>
      <h1>{dateString}</h1>
      <div className="day">{dayName}</div>
      <div className="tasks-left">할 일 {undoneTasks.length}개 남음</div>
    </TodoHeadBlock>
  );
}
```

### TodoList

TodoList에서는 `state`를 조회하고, `map`함수를 사용하여 TodoItem 컴포넌트를 렌더링해준다.

```js
import { useTodoState } from '../TodoContext';

...

function TodoList() {
  const todos = useTodoState();

  return (
    <TodoListBlock>
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          id={todo.id}
          text={todo.text}
          done={todo.done}
        />
      ))}
    </TodoListBlock>
  );
}
```

### TodoItem

`dispatch`를 사용해서 토글 기능과 삭제 기능을 구현해준다.

```js
import { useTodoDispatch } from '../TodoContext';

...

function TodoItem({ id, done, text }) {
  const dispatch = useTodoDispatch();
  const onToggle = () => dispatch({ type: 'TOGGLE', id });
  const onRemove = () => dispatch({ type: 'REMOVE', id });
  return (
    <TodoItemBlock>
      <CheckCircle done={done} onClick={onToggle}>
        {done && <MdDone />}
      </CheckCircle>
      <Text done={done}>{text}</Text>
      <Remove onClick={onRemove}>
        <MdDelete />
      </Remove>
    </TodoItemBlock>
  );
}
```

### TodoCreate

`dispatch`를 사용해서 새로운 할 일을 등록하는 생성 기능을 구현해준다. onSubmit에서는 새로운 항목을 추가하는 액션을 `dispatch` 한 후, `value` 초기화 및
`open` 값을 false로 전환해주었다.

```js
import { useTodoDispatch, useTodoNextId } from '../TodoContext';

...

function TodoCreate() {
  const [open, setOpen] = useState(false);
  const [value, setValue] = useState('');

  const dispatch = useTodoDispatch();
  const nextId = useTodoNextId();

  const onToggle = () => setOpen(!open);
  const onChange = e => setValue(e.target.value);
  const onSubmit = e => {
    e.preventDefault(); // 새로고침 방지
    dispatch({
      type: 'CREATE',
      todo: {
        id: nextId.current,
        text: value,
        done: false
      }
    });
    setValue('');
    setOpen(false);
    nextId.current += 1;
  };

  return (
    <>
      {open && (
        <InsertFormPositioner>
          <InsertForm onSubmit={onSubmit}>
            <Input
              autoFocus
              placeholder="할 일을 입력 후, Enter 를 누르세요"
              onChange={onChange}
              value={value}
            />
          </InsertForm>
        </InsertFormPositioner>
      )}
      <CircleButton onClick={onToggle} open={open}>
        <MdAdd />
      </CircleButton>
    </>
  );
}
```
