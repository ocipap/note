## React 에서 Redux 사용하기

> https://velog.io/@velopert/Redux-3-%EB%A6%AC%EB%8D%95%EC%8A%A4%EB%A5%BC-%EB%A6%AC%EC%95%A1%ED%8A%B8%EC%99%80-%ED%95%A8%EA%BB%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-nvjltahf5e



**리덕스의 3가지 규칙**

1. 하나의 애플리케이션 안에는 하나의 스토어
2. 상태는 읽기 전용
   1. 불변성을 유지하면서 변경
   2. shallow equality 검사를 하기 때문
   3. Immer.js 나 Immutable.js 등의 불편성 유지 라이브러리 사용
3. 리듀서는 순수함수



### 실습

**액션 타입 정의**

리듀서에서 사용할 액션 타입을 정의한다.

```js
const CHANGE_INPUT = 'waiting/CHANGE_INPUT'; // 인풋 값 변경
const CREATE = 'waiting/CREATE'; // 명단에 이름 추가
const ENTER = 'waiting/ENTER'; // 입장
const LEAVE = 'waiting/LEAVE'; // 나감
```



**액션 생성 함수 정의**

리듀사에서 사용할 액션 생성 함수를 정의한다.
FSA 규칙도 적용시킨다. ( type 으로 액션 타입, payload에 값을 전달 )

```Js
export const changeInput = text => ({ type: CHANGE_INPUT, payload: text });
export const create = text => ({ type: CREATE, payload: text });
export const enter = id => ({ type: ENTER, payload: id });
export const leave = id => ({ type: LEAVE, payload: id });
```



**createAction 이용해 정형화되고 쉽게 액션 생성 함수 정의하기**

`redux-actions` 에서 제공하는 `createAction` 사용하여 refactoring

```js
import { createAction } from 'react-actions';


let id = 0;

export const changeInput = createAction(CHANGE_INPUT, text => text);
export const create = createAction(CREATE, text => ({ text, id: id++ }));
export const enter = createAction(ENTER, id => id);
export const leave = createAction(LEAVE, id => id);
```

createAction 함수는 첫번째 인자로 액션 타입, 두번째 인자로 payload 생성 함수를 받는다.

리듀서는 순수 함수이여야 하기 때문에 crete 함수의 id 값은 액션을 생성하는 단계에서 변화를 주어야 한다. 



**initialState와 리듀서 정의**

초기값과 리듀서를 정의한다.

리듀서는 switch case 문으로 작성해야하지만, handleActions 함수를 이용해 쉽게 있다.

```javascript
const initialState = {
  input: '',
  list: [
    {
      id: 0,
      name: '홍길동',
      entered: true,
    },
    {
      id: 1,
      name: '콩쥐',
      entered: false,
    },
    {
      id: 2,
      name: '팥쥐',
      entered: false,
    },
  ],
};
```



**handleActions로 쉽게 리듀서 정의하기**	

`react-actions` 의 `handleActions` 는 기존에 switch case 문으로 작성하던 reducer 를 좀 더 편하게 작성할 수 있도록 해준다.
불변성을 유지하면서 새로운 state 를 정의한다.

```javascript
export default handleActions(
  {
    [CHANGE_INPUT]: (state, action) => ({
      ...state,
      input: action.payload,
    }),
    [CREATE]: (state, action) => ({
      ...state,
      list: state.list.concat({
        id: action.payload.id,
        name: action.payload.text,
        entered: false,
      }),
    }),
    [ENTER]: (state, action) => ({
      ...state,
      list: state.list.map(
        item =>
          item.id === action.payload
            ? { ...item, entered: !item.entered }
            : item
      ),
    }),
    [LEAVE]: (state, action) => ({
      ...state,
      list: state.list.filter(item => item.id !== action.payload),
    }),
  },
  initialState
);
```





**conbineReducers로 한개의 리듀서로 관리**

`redux` 의 `combineReducers` 를 이용하여 리듀서를 하나로 합쳐준다.

```js
import { combineReducers } from 'redux';
import counter from './counter';
import waiting from './waiting';

export default combineReducers({
  counter,
  waiting,
});
```



**스토어 생성 후 Provider 로 리액트 프로젝트 스토어 연동**

`redux` 의 `createStore` 를 이용해 store 를 생성한다.

```js
import { createStore } from 'redux';
import rootReducer from './store/modules';

const store = createStore(rootReducer);
```



`react-redux` 에 있는 `Provider` 를 이용해 스토어를 연동

App 을 Provider 로 감싸고, Provider에 스토어 전달

```react
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import rootReducer from './store/modules';

const store = createStore(rootReducer);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```



**connect 함수를 이용해 컴포넌트에 스토어 연동**

react-redux 에 있는 connect 함수를 이용해 컴포넌트에 스토어를 연동할 수 있다.

connect 함수는 호출 시 특정 컴포넌트를 받아 props 를 전달하는 함수를 반환

첫번째 실행시 스토어를 mapStateToProps 로 props, 액션 생성함수를 mapDispatchToProps 로 props 로 전달할 수 있다.



mapStateToProps 는 state 를 받아 해당 state 에서 필요한 값을 객체 형식으로 바꿔 props로 전달해준다.

mapDispatchToProps 도 dispatch 를 받아 지정해둔 액션 생성함수를 dispatch 하는 함수들을 객체 형식으로 만들어 props 로 전달해준다.



```javascript
const mapStateToProps = state => ({
  color: state.counter.color,
});

const mapDispatchToProps = dispatch => ({
  changeColor: color => dispatch(changeColor(color)),
});

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(PaletteContainer);
```



**bindActionCreators 이용해 쉽게 Action 전달하기**

`redux` 의 `bindActionCreactors` 는 mapDispatchToProps 에서 작성한 액션생성함수를 좀더 간편하게 작성할 수 있도록 해준다.

`actionCreator: (...params) => dispatch(actionCreator(...param))`

위와 같은 꼴로 작성되어 있다면 bindActionCreators 를 적용 가능하다.

```js
// bindActionCreators 적용 전 
const mapDispatchToProps = dispatch => ({
  increment: () => dispatch(increment()),
  decrement: () => dispatch(decrement()),
});

// bindActionCreators 적용 후
const mapDispatchToProps = dispatch => 
	bindActionCreators({ increment, decrement }, dispatch);
```



또 다른 방법으로는 mapDispatchToProps 자체를 액션 생성함수 객체 형태로 넘겨주면 connect 가 발생 시 bindActionCreators 를 자동으로 붙여준다.

```js
const mapDispatchToProps = { increment, decrement };

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(CounterContainer);
```