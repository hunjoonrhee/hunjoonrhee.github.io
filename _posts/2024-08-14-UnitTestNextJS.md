---
layout: single
title:  "Next.js에서 Jest로 유닛테스트 만들기"
categories: [Frontend]
tags: [Unit-Test, Jest, Next.js]
search: true
---
# Unit-Test
유닛 테스트, Unit Test, 란 **"단위"** 별 테스트를 뜻한다. 백엔드에서 **단위** 는 한 클래스의 매서드를 의미할 수 있고, 프론트엔드에선 하나의 컴포넌트를 가리킨다.  
유닛 테스트의 목적은  
>*코드의 특정 단위 (메서드나 모듈)가 기대한대로 작동하는지 확인*  

하는 것이다.

## Motivation
유닛 테스트를 하는 이유는 다음과 같다.
- 코드 품질 향상:
  - 버그 발견: 코드의 버그를 조기에 발견하고 고칠 수 있다.
  - 안정성 보장: 코드가 변경되더라도 기존의 기능이 올바르게 작동하는지 확인할 수 있다.
- 안전한 리팩토링: 코드 구조를 개선하거나 성능을 최적화할 때 유닛테스트가 있으면 기존 기능이 깨지지 않음을 보장할 수 있다.
- 개발 속도 향상: 자동화된 테스트 (CI / CD) 파이프라인에서 유닛 테스트를 자동으로 실행하여 코드 변경마다 자동으로 품질을 검증할 수 있다.
- 유지보수성 향상:
  - 변경 용이성: 코드 베이스가 커질수록 유지보수하기가 어려워진다. 유닛 테스트가 있으면 코드 변경 시 예상치 못한 부작용을 줄일 수 있다.
  - 신뢰성 확보: 새로운 기능 추가나 버그 수정시 기존 기능이 의도한대로 작동하는지 확인할 수 있다.

유닛 테스트의 주요 기여도는
>*코드가 변경될 때 기존의 기능이 의도한대로 작동하는지 확인할 수 있음*  

에 있다는걸 알 수 있다.  

## Unit-Test Library
유닛 테스트를 지원하는 라이브러리는 프로그래밍 언어마다 종류가 다양하게 있다.  
`Java`를 위한 보편적인 라이브러리에는 `JUnit` 이 있고, `JavaScript` 또는 `TypeScript` 를 사용한 코드를 테스트 할 때 보편적으로 사용되는 라이브러리에는 `Jest` 가 있다.  
다음은 유닛 테스트 라이브러리를 사용 언어별로 나타낸 표이다.

| 언어 | 유닛 테스트 라이브러리 | 설명 |
|-------|-------|-------|
| `JavaScript(Node.js)` | `Jest` | Facebook에서 만든 테스트 프레임워크. 설정이 간편하고 모킹, 스타이, 코드 커버리지 지원. |
| | `Mocha` | 유연하고 확장 가능한 테스트 프레임워크. 비동기 테스트에 강력함. |
| | `Chai` | `Mocha`와 함께 사용되는 assertion 라이브러리. 다양한 assertion 스타일 지원. |
| | `Sinon` | 스파이, 스터브, 모킹 기능을 제공하는 라이브러리. |
| `Python` | `unittest` | 표준 라이브러리. `JUnit` 스타일의 테스트 프레임워크.
| | `pytest` | 사용하기 쉽고 확장 가능한 테스트 프레임워크. 풍부한 플러그인 지원.
| | `nose` | `pytest` 와 유사한 기능을 제공하는 테스트 프레임워크.
| `Java` | `JUnit` | 가장 널리 사용되는 `Java` 테스트 프레임워크.
| | `TestNG` | `JUnit` 과 유사하지만 더 많은 기능과 유연성을 제공.
| | `Mockito` | `Java`용 모킹 라이브러리. 단위 테스트에서 의존성을 모킹할 때 사용.

백엔드 (`Java`)에서의 유닛테스트는 [백엔드 유닛테스트](https://hunjoonrhee.github.io/backend/frontend/UnitTest/)에서 어느정도는 확인할 수 있다. 그래서 이번에는 프론트엔드에서 유닛 테스트 하는 방법을 소개하려고 한다.

# Unit-Testing in Frontend
## Configuration Jest
현재 내가 작업하고 있는 프로젝트는 `Next.js` 프레임워크와 `TypeScript`를 사용하고 있다. 그리고 상태 관리를 위한 라이브러리는 `redux-saga`를 사용한다. 이 안에서 `Jest` 를 사용한 유닛테스트를 하기 위한 초기 설정방법을 소개하겠다.  
1. 필요한 패키지 설치
   >npm install --save-dev jest @testing-library/react @testing-library/jest-dom @testing-library/user-event babel-jest @babel/preset-typescript @babel/preset-react @babel/preset-env @babel/core @types/jest jest-environment-jsdom redux-mock-store

2. `Babel` 설정
   `Next.js`는 기본적으로 `Babel`을 사용하므로, `Babel`을 `Jest`와 함께 사용하기 위한 설정이 필요하다. 프로젝트 루트에 `.babelrc` 파일을 추가하고 다음과 같이 설정한다.
  ```json
    {
      "presets": ["next/babel"]
    }
  ```
3. `Jest` 설정 파일
   `Jest`를 설정하기 위해 프로젝트 루트에 `jest.config.js`파일을 생성한다.
   ```javascript
   const nextJest = require('next/jest');
   const createJestConfig = nextJest({
    dir: './',
    });
    
    const customJestConfig = {
        testEnvironment: 'jest-environment-jsdom',
        moduleNameMapper: {
            '^@/components/(.*)$': '<rootDir>/components/$1',
            '^@/pages/(.*)$': '<rootDir>/pages/$1',
            },
            setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
            };
    
    module.exports = createJestConfig(customJestConfig);
    ```
    또한 추가적으로 `jest.setup.js`도 같은 위치에 생성한다.
    ```javascript
    import '@testing-library/jest-dom';
    ```
이로써 `Jest` 를 사용하여 유닛테스트를 만들기 위한 초기 설정이 완료된다.

## Making Unit-Test
이해를 돕기 위해 회원가입 페이지에 렌더링 되는 회원가입 폼 컴포넌트를 예로 들어 설명하겠다.  
회원가입 페이지는 아래와 같다.  
<img src="../../images/2024-08-14-UnitTestFE/RegisterForm.png" alt="RegisterForm" style="zoom:50%;" />

회원가입 폼은 다음과 같이 구성되어있다.
- 이름, 이메일, 비밀번호, 비밀번호 확인, 주소, 전화번호를 입력할 수 있는 입력 필드 (Input Field)
- 이용 약관에 동의한다는 체크박스 
- 회원가입 버튼

이 중에 이름, 이메일, 비밀번호, 비밀번호 확인과 체크박스는 필수 항목이고 주소와 전화번호 필드는 선택 항목이다. 필수 항목을 빠뜨리고 회원가입 버튼을 누르면 빠뜨린 필드에 경고 메시지가 뜬다.  
필수 항목들이 다 입력 및 체크가 되고 회원가입 버튼을 누르게 되면 회원가입 요청을 날리는 디스패치가 호출된다.

위와 같은 User Story에 적합한 테스트 케이스는 다음과 같이 정의될 수 있다.
1. 회원가입 폼 컴포넌트가 렌더링 되면 6개의 Input Field, 1 개의 체크박스 그리고 1 개의 버튼이 화면에 나타난다.
2. 필수 항목들이 입력이 되고 체크 박스가 체크된 후 회원가입 버튼을 누르게 되면 회원 가입 요청을 날리는 디스패치가 호출된다.
3. 필수 항목들 중 하나라도 입력이 안된 상태에서 회원가입 버튼을 누르게 되면 *This is a mandatory field* 라는 경고 메시지가 화면에 나타난다.
4. 체크 박스를 체크하지 않은 채로 회원가입 버튼을 누르게 되면 *You should agree the Policy* 라는 경고 메시지가 화면에 나타난다.

이제 각각의 테스트 케이스를 다루는 테스트 파일을 만들어보자.

### RegisterForm.test.tsx
먼저 `RegisterForm.test.tsx` 라는 이름의 파일을 `/test` 폴더 안에 생성한다.  
그리고 다음과 같은 패키지를 `import` 한다.
#### import packages
```typescript
import '@testing-library/jest-dom';
import { registerRequest } from '@/app/actions';
import RegisterForm from '@/app/components/RegisterForm';
import rootReducer from '@/app/reducers';
import rootSaga from '@/app/sagas';
import { fireEvent, render, screen } from '@testing-library/react';
import { Provider } from 'react-redux';
import configureStore from 'redux-mock-store';
import createSagaMiddleware from 'redux-saga';
```
- `@testing-library/jest-dom`: 이 라이브러리는 `jest`와 함께 사용되는 도구이다. `DOM`에 대한 맞춤형 `matchers`를 제공한다. 예를 들어, 어떠한 한 요소 (Element)가 문서에 존재하는지 확인하거나, 특정 텍스트를 포함하고 있는지 테스트 할때 유용하다.
- `registerRequest`: 리덕스 액션 생성자 (action creator)이다. 회원가입 요청과 관련된 액션을 생성하는데 사용하고, 테스트에서는 이 액션 생성자는 실제 액션 생성자로써 나중에 모킹된 액션과 비교할 때 사용된다.
- `RegisterForm`: 회원가입 폼 컴포넌트이다. UI 요소로서 이 컴포넌트가 렌더링 된 후에 테스트가 시작된다.
- `rootReducer`: 애플리케이션의 전체 `Redux` 리듀서를 의미한다. 나중에 `mockStore`를 만들때 사용한다.
- `rootSaga`: 애플리케이션의 전체 사가를 의미한다. 사가 미들웨어를 실행할 때 사용한다.
- `fireEvent, render, screen`: 리액트 컴포넌트를 테스트할 때 사용하는 유틸리티이다. `render`는 컴포넌트를 렌더링하고, `screen`은 `DOM`에 접근하도록 도와주며 `fireEvent`는 사용자 이벤트 (클릭 또는 입력)을 시뮬레이션하도록 도와준다.
- `Provider`: 리덕스 스토어를 리액트 컴포넌트 트리에 제공하는 컴포넌트이다. `Provider`를 통해 애플리케이션의 모든 하위 컴포넌트들이 리덕스 스토어에 접근할 수 있게 된다.
- `configureStore from 'redux-mock-store'`: `redux-mock-store`는 리덕스 스토어의 목 (mock) 버전을 생성할 때 사용한다. 이 mock 스토어는 테스트 환경에서 리덕스의 동작을 시뮬레이션할 때 사용한다.
- `createSagaMiddleware`: 리덕스 애플리케이션에서 사가 미들웨어를 생성할 때 사용한다. 이 미들웨어는 사가를 실행하고 리덕스 스토어와 비동기 작업을 관리하는데 필요하다.

#### configure reducer, store and saga for testing
패키지 import 가 끝나면 리듀서와 스토어 그리고 사가 설정을 한다. 이것을 하는 이유는 테스팅하는 컴포넌트가 리덕스 스토어를 사용해서 설정을 하는것이다. 만약 리덕스를 사용하지 않는다면 이 설정은 할 필요가 없다.  

```typescript
const sagaMiddleware = createSagaMiddleware();
const mockStore = configureStore([sagaMiddleware]);
const store = mockStore({
reducer: rootReducer,
});
const mockDispatch = jest.fn();

jest.mock('react-redux', () => ({
...jest.requireActual('react-redux'),
useDispatch: () => mockDispatch,
}));

sagaMiddleware.run(rootSaga);
```

`sagaMiddleware`와 `rootSaga` 그리고 `rootReducer`들은 `mocking` 하지 않는다. 우리는 스토어와 디스패치만 `mocking` 해주면 된다. 먼저 `configureStore` 함수로 비어있는 목(mock) 스토어를 만들어주고, 그 안에 `rootReducer`를 포함시켜 목 스토어안에 우리가 사용하게될 리듀서를 넣어준다.  
중요한건 `Dispatch` 를 모킹 (mocking)하는것이다. 우리는 컴포넌트에서 디스패치를 사용할 때 다음과 같이 디스패치 함수를 상수 (`const`)로 정의하고 사용한다.
>const dispatch = useDispatch();

따라서 디스패치 함수를 `jest.fn()` 으로 모킹을 해도, 디스패치 함수를 정의해주는 `useDispatch()` 훅 (`hook`)도 같이 모킹을 해주지 않는다면 충분하지 않다.  
`useDispatch()` 훅은 `react-redux` 모듈에서 제공한다. `useDispatch()` 이것만 모킹을 하는 방법도 있을 수 있겠지만, 모듈이 모킹되어있지 않은 상태에서 `useDispatch()`만 모킹을 시도했더니 갖은 에러가 떠서 조사를 하다 다음과 같은 해결방안을 찾게 되었다.

*전체 모듈을 모킹하되, 그 안에서 `useDispatch()` 훅만 우리가 모킹한 함수인 `mockDispatch`를 반환하게 하고 나머지는 실제 모듈로 사용하게 한다*

이 부분이 위의 코드의 `jest.mock('react-redux')...` 부분이다.
이로써 테스팅에 필요한 리덕스 세팅이 완료가 된다.

#### start testing, beforeEach
이제 실제 테스트 코드를 짤 준비가 되었다. 테스트의 시작을 알리는 `Syntax`는 `describe()`이다. 이 `describe()`함수 안에 테스트 이름을 적고, 그 안에 하위 함수를 정의 함으로써 테스트 코드가 작성된다.

```typescript
describe('RegisterForm', () => {
  // Redux-Saga Middleware
  beforeEach(() => {
    render(
      <Provider store={store}>
        <RegisterForm />
      </Provider>,
    );

    // get all input fields
    nameInput = screen.getByLabelText('name');
    emailInput = screen.getByLabelText('email');
    passwordInput = screen.getByLabelText('password');
    confirmPasswordInput = screen.getByLabelText('confirmPassword');
    addressInput = screen.getByLabelText('address');
    mobileInput = screen.getByLabelText('mobile');
    // checkbok
    policy = screen.getByRole('checkbox');
    // button
    registerButton = screen.getByRole('button', { name: 'Register', hidden: true });

    store.clearActions();
    jest.clearAllMocks();
  });
```

여기에 `beforeEach`라는 함수가 쓰였는데, 이 함수는 이름에서 알 수 있듯이 각각의 테스트 케이스가 시작되기 전에 `jest`가 작동을 시켜준다. 따라서 테스트 케이스가 공통적으로 공유하는 설정들은 이렇게 `beforeEach` 함수로 따로 떼어내서 정의를 해주면 테스트 케이스의 코드도 간결해지고 가독성도 올라가게 된다.  

우리는 회원가입 폼 컴포넌트를 테스트 하기 때문에, 각각의 테스트 케이스를 실행시키기 위해선 회원가입 폼 컴포넌트가 렌더링이 되어있어야 한다. 그리고 앞서 말한 네가지 테스트 케이스 모두 입력 필드, 체크박스 그리고 버튼이 찾아져 있어야 하기 때문에, 컴포넌트 렌더링과 입력필트, 체크박스 및 버튼을 찾는 코드가 `beforeEach` 로 정의된 것이다. 이 밖에도 `store.clearActions()`, `jest.clearAllMocks()`는 스토어와 모킹된 모든 것들을 하나의 테스트 케이스가 끝나면 초기화가 되어야 하기 때문에 `beforeEach`안에 넣어줬다.

#### it should render all input fields, checkbox and button for register
각각의 테스트 케이스는 `it('test name', () => {})` 으로 정의한다.  
보통 함수 이름인 `it`과 'test name'을 합쳐서 이 테스트 케이스의 목적을 하나의 문장으로 만드는게 통상적이다. 이 테스트 케이스는 *회원가입 폼 컴포넌트가 렌더링이 되면 모든 입력 필드, 체크박스 그리고 버튼이 화면에 나타난다* 가 목적이다. 따라서 다음과 같이 이름을 지어주면 좋다.

```typescript
it('should render all input fields, checkbox and button for register', () =>{})
```

우리가 원하는 요소들이 화면에 나타나는걸 테스트하기 위한 가장 좋은 방법은 `toBeInTheDocument()` 함수를 사용하는 것이다. 

> expect(nameInput).toBeInTheDocument()

앞서 `beforeEach` 에서 정의한 nameInput 이라는 입력 필드가 화면에 존재한다면 `toBeInTheDocument()` 가 `true`를 반환할 것이고 아니라면 `false`를 반환하고 해당 테스트 케이스는 fail된다.

#### it should dispatch register request when register-button is clicked
이 케이스가 이번 테스트의 하이라이트 이다. 이 케이스는 회원가입 버튼이 눌렸을 때 회원가입 리퀘스트 액션이 디스패치 되는것을 테스트 한다.  

먼저 각각의 입력 필드에 이제 텍스트가 입력이 되어야 한다. 입력이나 클릭같은 사용자 이벤트는 `fireEvent` 라는 함수를 사용할 수 있다. 

> fireEvent.change(nameInput, {target: {value: 'Joon'}});

이렇게 하면 이름을 입력하는 필드에 'Joon'이라는 텍스트가 입력이 된다. `{target: {value: }}`로 넣어주는 이유는 `fireEvent`가 입력된 텍스트에 접근하기 위한 경로를 알려주기 위함이다. 우리가 일반 컴포넌트에서 필드에 입력된 값을 추출할 때에 `event.target.value`를 사용하는 것과 같은 원리이다.

그리고 클릭 이벤트는 다음과 같이 쓰면 된다.

> fireEvent.click(policy);

이렇게 모든 필수 사항들이 완료가 되고 `fireEvent.click(registerButton)`이 실행이 되면 디스패치가 호출 되어야 한다. 실제 회원가입 폼에서 버튼이 눌릴 때 `handleOnSubmit()`이라는 함수가 실행되는데 거기에는 하나의 디스패치가  한 번 호출이 된다. 그래서 우리가 기대하는 것도 *디스패치는 한 번 호출된다* 이다. 이것은 `jest` 에서 제공하는 함수인 `toHaveBeenCalledTimes(1)`로 테스트 할 수 있다.

```typescript
// Check that mockDispatch was called
    expect(mockDispatch).toHaveBeenCalledTimes(1);
```

우리가 모킹한 `mockDispatch`가 호출이 되고 나면 우리가 다음으로 기대하는 것은 그 모킹된 디스패치 안에 있는 리퀘스트 액션이 디스패치 되는 것이다.  

`mockDispatch`라는 함수 안에 `.mock().calls` 라는 함수를 불러올 수 있는데, `mockDispatch`를 실행했을 때 호출되는 디스패치들을 불러오는 함수이다. 디스패치들은 배열로 저장이 된다. 
위에 *"하나의 디스패치가 한 번 호출된다"* 라고 적혀있다. 따라서 호출되는 디스패치는 첫번째, 즉 배열의 0 번째 요소가 되는것이다: `mockDispatch.mock().calls[0]`.  

그리고 디스패치 안에는 여러 인자들이 있을 수 있는데, 그 중 첫번째 인자는 항상 액션이 들어간다.  
이제 우리가 테스트 해야할 것은 액션이 제대로 디스패치가 되는가 이다. 그러기 위해선 모킹한 디스패치의 액션에 접근을 해야한다: `mockDispatch.mock().calls[0][0]` 이것이 모킹한 디스패치에 들어가는 액션이다.

이 액션은 실제 컴포넌트에서 디스패치 되는 액션인 `registerRequest()`와 일치해야하기 때문에 그다음 테스트 코드는 다음과 같이 작성할 수 있다.

```typescript
const dispatchedAction = mockDispatch.mock.calls[0][0];

expect(dispatchedAction).toEqual(
    registerRequest({
    name: 'Joon',
    email: 'joon@gmail.com',
    password: 'password123',
    address: '',
    mobile: '',
    }),
);
```

이렇게 하면 우리의 두번째 테스트 케이스 작성이 완료된다.

#### it should render a warning message
세번째와 네번째 테스트 케이스는 묶어서 다뤄보겠다. 이 두 테스트 케이스는 경고 메시지에 관한 내용이다. 여기서 중요한 핵심은 *인풋 필드가 작성이 안되었을 때 해당 필드의 속성중 어떤것이 바뀌는가* 이다.  

다음 두가지 그림을 보자.

<img src="../../images/2024-08-14-UnitTestFE/input_normal.png" alt="input_normal" style="zoom:50%;" />

<img src="../../images/2024-08-14-UnitTestFE/input_error.png" alt="input_error" style="zoom:50%;" />

첫번째 그림은 아무일도 일어나지 않는 필드이다. 이 필드는 `aria-invalid`라는 속성을 갖고 있고, 이 속성은 기본적으로 `false` 값을 갖고 있다. 하지만 두번째 사진처럼 이 필드가 입력이 되지 않은 상태에서 버튼이 눌려져 경고 메시지가 뜨게 되면 이 `aria-invalid` 값이 `true`가 된 것을 알수 있다.  

우리는 이 특징을 가지고 다음과 같은 코드를 작성해주면 된다.

```typescript
expect(nameInput).toHaveAttribute('aria-invalid', 'true');
```

이 코드가 통과가 되었다는 뜻은 `nameInput` 이 입력이 되지 않았다는 뜻이고, 이렇게 되면 우리는 다음 결과인, *This is a mandatory field*라는 텍스트가 화면에 나타나야함을 기대하게 된다.

```typescript
expect(screen.getByText('This is a mandatory field.')).toBeInTheDocument();
```

체크박스가 체크가 되지 않음을 테스트 하는 코드는 다음과 같다.

```typescript
expect(policy).not.toBeChecked();
```

이 코드가 통과가 되었다는 뜻은 `policy` 체크박스가 체크가 안된 채로 버튼이 눌렸다는 뜻이고, 이렇게 되면 우리는 다음 결과인, *You should agree the Policy.*라는 텍스트가 화면에 나타나야함을 기대하게 된다.

```typescript
expect(screen.getByText('You should agree the Policy.')).toBeInTheDocument();
```

# Conclusion
우리는 `Next.js` 프로젝트에서 `Jest`를 사용하여 테스트 케이스를 코딩하는 방법에 대해서 알아보았다. 이 글이 테스팅하는 모든 케이스를 설명해주진 않겠지만 다음과 같은 핵심적인 내용을 내포하고 있다.
- `Jest` 초기 설정
- 테스트 코드를 시작하는 법
- `dispatch`를 `mocking` 하는 법
- `HTML` 요소 (Input field, checkbox, button)를 찾는 법

