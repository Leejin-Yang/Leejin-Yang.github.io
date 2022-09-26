---
layout: post
title: Redux | middleware & thunk
description: 리덕스 미들웨어와 thunk
summary: 리덕스 미들웨어 개념과 redux-thunk
tags: redux
---

이전 Redux 개념에서 보면 reducer는 순수 함수여야 한다는 원칙이 있다. side effect를 발생시키는 작업(비동기 로직 등 함수 외부의 상태 또는 동작을 변경하는 작업)이 reducer에서 이루어지면 안된다. 이 작업을 수행하기 위해 middleware 개념이 존재한다.

_Redux Middleware는 side effect가 발생하는 로직을 작성할 수 있도록 설계되었다._

```tsx
type MiddlewareAPI = { dispatch: Dispatch; getState: () => State }
type Middleware = (api: MiddlewareAPI) => (next: Dispatch) => Dispatch
```

Redux Middleware는 콘솔을 찍거나, 비동기 타이머, 비동기 API 호출 등 여러가지 작업을 할 수 있다. Action의 정보를 가로채 정보에 따라 특정 작업을 수행한 뒤 reducer에게 전달해주거나 정보에 따라 아예 무시할 수 있다. middleware를 다음과 같은 형태로 정의할 수 있다.

```jsx
// Middleware written as ES5 functions

// Outer function:
function exampleMiddleware(storeAPI) {
  return function wrapDispatch(next) {
    return function handleAction(action) {
      // Do anything here: pass the action onwards with next(action),
      // or restart the pipeline with storeAPI.dispatch(action)
      // Can also use storeAPI.getState() here

      return next(action)
    }
  }
}

// 화살표 함수로 표현시

const anotherExampleMiddleware = (storeAPI) => (next) => (action) => {
  // Do something in here, when each action is dispatched

  return next(action)
}

// 출처 https://redux.js.org/tutorials/fundamentals/part-4-store
```

- exampleMiddleware: middleware 자체이다. applyMiddleware로 호출이 되고, storeAPI를 받는데, 이 storeAPI는 store의 {dispatch, getState} 함수를 가진다.
- wrapDispatch: next는 다음 미들웨어 또는 store.dispatch 역할을 한다. next(action)으로 다음 middleware나 reducer로 action을 넘긴다. 또는 storeAPI.dispatch(action)으로 현재 middleware를 다시 처리할 수 있다.
- handleAction: 현재 action을 받아 작업이 전달될 때마다 호출된다.

_여러 로직을 수행 후 next(action)으로 action을 넘기는 것을 기억하면 된다._

간단한 로그 예시

```jsx
const loggerMiddleware = (storeAPI) => (next) => (action) => {
  console.log('current state', storeAPI.getState())
  console.log('dispatching', action)
  let result = next(action) // action을 다음 middleware나 reducer로 전달한다.
  console.log('next state', storeAPI.getState()) // action 처리 후 state 값을 가져온다.
  return result // result는 store.dispatch(action)이 적용된 결과를 가진다.
}

// middleware를 적용할 때, store 생성 시 applyMiddleware를 추가하면 된다.
const store = createStore(rootReducer, applyMiddleware(loggerMiddleware))
```

<br>

## Redux-Thunk

redux-thunk는 middleware로 action 대신 함수를 반환하는 action creator를 사용할 수 있게 한다. 이를 통해 action을 지연시키거나 상태에 따라 action을 실행하지 않게 할 수 있다. 비동기 로직 처리, 비동기 API 호출을 사용할 때 사용한다. Redux에서 비동기 로직을 처리할 때 권장하는 공식 middleware이기도 하다.

thunk 사용 방법

```jsx
import { createStore, applyMiddleware } from 'redux'
// thunk를 redux-thunk에서 import
import thunk from 'redux-thunk'

// middleware 추가
const store = createStore(modules, applyMiddleware(thunk))
```

thunk middleware 구현 코드를 보면 다음과 같다.

```jsx
// standard middleware definition, with 3 nested functions:
// 1) Accepts `{dispatch, getState}`
// 2) Accepts `next`
// 3) Accepts `action`
const thunkMiddleware =
  ({ dispatch, getState }) =>
  (next) =>
  (action) => {
    // If the "action" is actually a function instead...
    if (typeof action === 'function') {
      // then call the function and pass `dispatch` and `getState` as arguments
      return action(dispatch, getState)
    }

    // Otherwise, it's a normal action - send it onwards
    return next(action)
  }
```

action의 타입이 함수인 경우 dispatch와 getState를 받아 action을 실행한다. 아닌 경우 action을 다음 middleware나 reducer로 넘겨준다.

thunk 함수 예시

```jsx
function fetchData() {
  return async (dispatch, getState) => {
    // 비동기 API 호출
    const response = await axios.get('.../data')

    // 비동기 작업이 마치고 dispatch
    dispatch({ type: 'FETCH_DATA', payload: response.data })
  }
}

store.dispatch(fetchData())
```

함수로 받은 경우 함수 실행 → 비동기 작업 후 dispatch 실행 → action 타입 체크 → 함수가 아닌 경우 next(action)을 실행

_해당 middleware에서 next(action)이 아닌 dispatch를 수행하면 해당 middleware가 재시행_

Promise.then().catch()나 try/catch 구문으로 에러 처리를 할 수 있다. 상황에 따른 action을 수행할 수 있다.

```jsx
function fetchData(someValue) {
  return async (dispatch, getState) => {
    dispatch(requestStarted())

    // Have to declare the response variable outside the try block
    let response

    try {
      response = await myAjaxLib.post('/someEndpoint', { data: someValue })
    } catch (error) {
      // Ensure we only catch network errors
      dispatch(requestFailed(error.message))
      // Bail out early on failure
      return
    }

    // We now have the result and there's no error. Dispatch "fulfilled".
    dispatch(requestSucceeded(response.data))
  }
}
```

기존 Redux 흐름에서 middleware가 추가된 다이어그램이다.

![redux-async-data-flow](/assets/img/redux-async-data-flow.gif)

<br>

## 결론

간단하게 middleware의 필요성과 비동기 로직 처리할 때 이용하는 redux-thunk에 대해 정리해보았다.

- _Redux Middleware는 side effect가 발생하는 로직을 작성할 수 있도록 설계되었다._
- _여러 로직을 수행 후 next(action)으로 action을 넘긴다._

Redux Toolkit에는 기본적으로 thunk가 내장되어 있다. 추가 라이브러리 설치 없이 이용할 수 있다.
_Redux를 사용해야 한다면 Redux Toolkit을 사용하자_

<br>

## Reference

[https://redux.js.org/usage/writing-logic-thunks](https://redux.js.org/usage/writing-logic-thunks)

[https://redux.js.org/tutorials/fundamentals/part-6-async-logic](https://redux.js.org/tutorials/fundamentals/part-6-async-logic)
