# 💻 Next-Tutorial-Redux-Scss
### Next.js에 Redux-Saga와 Sass 적용

<br />

## 🎥 참고 YouTube
### 🔖 https://www.youtube.com/watch?v=UXMGGI3TSs4&t=622s
<!-- ### 📺 Stack Navigation
<p align='center'>
    <img src='https://user-images.githubusercontent.com/64779472/114034632-50637880-98b9-11eb-98d8-a2111e389a09.PNG' width="400" height="730">
</p> -->

<br />

## 👨🏻‍💻 Install
- yarn add @zeit/next-sass node-sass@4.14.1
- yarn add next-redux-saga next-redux-wrapper react-redux redux redux-devtools-extension redux-saga
- 덤으로 yarn add redux-thunk

<br />

## 👨🏻‍💻 Sass
- next.js에 sass를 적용하려면 @zeit/next-sass 라이브러리를 추가로 Install해야 되며, .next.config.js를 생성해서 다음 코드를 추가해야 된다.
- 그리고 _app.js에 import "../scss/main.scss"; 를 하면 된다.

<br />

### 🏃 .next.config.js
```js
  const withSass = require("@zeit/next-sass");

  module.exports = withSass({
    cssModules: true,
  });
```

<br />

## 👨🏻‍💻 Redux-Saga
### 🏃 store.js
- composeWithDevTools: redux devtools를 사용하기 위한 메서드
- createSagaMiddleware: redux-saga 생성
- createStore: Store 생성
- applyMiddleware: 미들웨어 적용

<br />

```js
  import { createStore, compose, applyMiddleware } from "redux";
  import { composeWithDevTools } from "redux-devtools-extension";
  import createSagaMiddleware from "redux-saga";

  import rootReducer from "./reducers";
  import rootSaga from "./sagas";

  //redux-saga 생성
  const sagaMiddleware = createSagaMiddleware();

  //초기 initailState
  const initialState = {};

  //미들웨어 연결
  const middleware = [sagaMiddleware];

  //개발 모드라면 composeWithDevTools
  //배포 모드라면 compose
  const enhancer =
    process.env.NODE_ENV === "production"
      ? compose(applyMiddleware(...middleware))
      : composeWithDevTools(applyMiddleware(...middleware));

  //store 생성
  const store = createStore(rootReducer, initialState, enhancer);

  //store에 rootSaga를 넣은 sagaMiddleware를 실행
  store.sagaTask = sagaMiddleware.run(rootSaga);

  export default store;
```
<br />

### 🏃 _app.js
- Component를 Provider로 묶어주면 모든 컴포넌트에서 redux store를 사용할 수 있게 된다.
- createWrapper: getServerSideProps, getStaticProps 같은 next의 라이프 사이클에 redux를 결합시키는 역할. 따라서, wrapper.withRedux로 페이지를 감싸게 되면 redux가 결합된 라이프사이클을 사용할 수 있음

<br />

```js
  import "../styles/globals.css";
  import React from "react";
  import { Provider } from "react-redux";
  import { createWrapper } from "next-redux-wrapper";
  import store from "../store/store";
  import "../scss/main.scss";

  function MyApp({ Component, pageProps }) {
    return (
      <Provider store={store}>
        <Component {...pageProps} />;
      </Provider>
    );
  }

  const makestore = () => store;
  const wrapper = createWrapper(makestore);

  export default wrapper.withRedux(MyApp);
```

<br />

### 🏃 sagas/index.js
- all: 제너레이터 함수를 배열의 형태로 인자로 넣어주면, 제너레이터 함수들이 병행적으로 동시에 실행되고, 전부 resolve될 때까지 기다린다.
- call: 함수의 동기적인 호출을 할 때 사용
- fork: 함수의 비동기적인 호출을 할 때 사용(즉, call과 다르게 순서 상관없이 실행해야될 때 사용)

<br />

```js
  import { all, fork } from "redux-saga/effects";
  import postSaga from "./postSaga";

  //제너레이터
  export default function* rootSaga() {
    yield all([fork(postSaga)]);
  }
```

<br />

### 🏃 sagas/postSaga.js
- put: dispatch와 같은 역할을 한다.
- takeEvery: 들어오는 모든 액션에 대해 특정 작업을 처리한다.
- takeLatest: 기존에 진행 중이던 작업이 있다면 취소하고, 가장 마지막으로 실행된 작업만 수행

<br />

```js
  import { GET_POSTS, GET_REQ } from "../types";
  import { all, fork, put, takeEvery } from "redux-saga/effects";

  function* post() {
    yield put({
      type: GET_POSTS,
      payload: ["1st post", "2nd posts", "3 posts"],
    });
  }

  function* watchPost() {
    yield takeEvery(GET_REQ, post);
  }

  //postSaga() 여러 Saga 통합
  export default function* postSaga() {
    yield all([fork(watchPost)]);
  }

```

### 🏃 reducers/index.js
- combineReducers: 많은 Reducer들을 하나로 합쳐 하나의 Reducer로 관리할 수 있게 만들어준다.

<br />

```js
  import { combineReducers } from "redux";
  import { postReducer } from "../reducers/postReducer";

  export default combineReducers({
    post: postReducer,
  });

```

<br />

### 🏃 reducers/postReducer.js
```js
  import * as types from "../types";

  const initialState = {
    posts: [],
    post: {},
    loading: false,
    error: null,
  };

  export const postReducer = (state = initialState, action) => {
    switch (action.type) {
      case types.GET_REQ:
        return {
          ...state,
          posts: [],
          loading: true,
          error: null,
        };
      case types.GET_POSTS:
        return {
          ...state,
          posts: action.payload,
          loading: false,
          error: null,
        };
      default:
        return state;
    }
  };

```
<br />