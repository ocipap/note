## Vanilla JS 에서 Redux 사용하기

**액션 타입 정의**

리듀서와 액션 함수에서 해당 액션 타입의 값을 이용, 상수 느낌

```js
const TOGGLE_SWITCH = 'TOGGLE_SWITCH';
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';
```



**액션 생성 함수 정의**

액션 타입에서 정의한 상수를 타입으로 하는 함수들을 생성

이때 type 값은 필수이고, type을 제외한 나머지 값은 선택, 추후 리듀서에서 action 부분에서 사용할 수 있음

```js
const toggleSwitch = () => ({ type: TOGGLE_SWITCH });
const increment = diff => ({ type: INCREMENT, diff });
const decrement = () => ({ type: DECREMENT });
```



**초기값 설정**

state 의 초기 값

```js
const initialState = {
  light: true,
  counter: 713,
};
```



**리듀서 함수 정의**

state 에 변화를 주는 함수, 액션 함수에서 넘겨받은 actions 를 기반으로 새로운 state 반환

```js
function reducer(state = initialState, action) {
  switch (action.type) {
    case TOGGLE_SWITCH:
      return {
        ...state,
        light: !state.light,
      };
    case INCREMENT:
      return {
        ...state,
        counter: state.counter + action.diff,
      };
    case DECREMENT:
      return {
        ...state,
        counter: state.counter - 1,
      };
    default:
      return state;
  }
}
```



**스토어 생성**

store 는 `createStore` 함수에 reducer 를 넘겨주어 생성함

state 와 reducer를 포함하고, 유용한 내장 메서드들이 있음

```js
const store = createStore(reducer);
```



**render 함수 선언**

state 에 따라 각기 다른 view 를 렌더하는 함수를 선언

`store.getState()` 를 통해 현재 state 를 가져올 수 있음

```js
const render = () => {
  const state = store.getState();
  const { light, counter } = state;

  if (light) {
    $light.style.background = 'green';
    $switch.innerText = '끄기';
  } else {
    $light.style.background = 'black';
    $switch.innerText = '켜기';
  }
  $counter.innerText = counter;
};
```



**스토어 구독하기**

store 의 내장 함수인 subscribe 에 함수를 넘겨, 액션 생성 함수가 dispatch 되었을 때 전달받은 함수를 실행함



**이벤트 별로 디스패치 하기**

store 의 내장 함수로, 액션 생성 함수로 생성된 액션을 넘겨받아, reducer 를 실행시킴



### 상태 관리 플로우 정리

store 정의 -> dispatch 를 통해 액션 생성 후 -> reducer 에 전달, 액션에 따라 새로운 state 생성 -> 구독중인 함수에서 변경된 state 를 기반으로 로직 처리

