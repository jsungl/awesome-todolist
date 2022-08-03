# To-Do List with React App

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
