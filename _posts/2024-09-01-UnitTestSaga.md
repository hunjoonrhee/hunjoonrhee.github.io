---
layout: single
title:  "Jest로 Saga 테스트하기"
categories: [Frontend]
tags: [Unit-Test, Jest, Saga, Redux-Saga]
search: true
---

# Introduction
[Next.js에서 Jest로 유닛테스트 만들기](https://hunjoonrhee.github.io/frontend/UnitTestNextJS/)에서 다음과 같은 내용을 다뤘다.
- 컴포넌트에 HTML 요소들이 렌더링이 되는가?
- 사용자 이벤트들이 제대로 동작하는가?
- 액션 요청이 디스패치가 잘 되는가?

위의 내용들을 설명하기 위해 나는 `RegisterForm.test.tsx` 의 테스트 코드를 예를 들어 설명했다. 리덕스 사가를 사용하기 때문에 스토어 설정과 사가 미들웨어 설정, 디스패치를 모킹하는 법 등이 상세하게 설명이 되어있다. 만약 저 글을 보지 않았다면 저 글을 먼저 보고 이 글을 읽는 것을 추천한다.

오늘은 액션 요청을 가로챈 사가의 요청 처리를 테스트 하는 법에 대해서 설명하고자 한다. 사가는 미들웨어로써 컴포넌트에서 날라온 디스패치를 중간에 가로채서 처리를 한다 ([Redux Saga](https://hunjoonrhee.github.io/frontend/Saga/#saga) 참조). 이때 요청 처리가 성공했을 때 성공 액션이 디스패치가 잘 되는지, 또는 실패했을 때 실패 액션이 디스패치가 잘 되는지 테스트를 하는 것도 중요하다.

# Test Scenario
오늘 예로 들 기능은 로그인 기능이다. 로그인 버튼을 누르게 되면 `handleOnSubmit` 함수가 실행되면서 `{email, password}` 객체가 `loginRequest` 의 인자로 사용되면서 로그인 액션 요청이 디스패치가 된다:

```typescript
const handleOnSubmit = (event: FormEvent<HTMLFormElement>) => {
    event?.preventDefault();
    dispatch(loginRequest({ email: formData.email, password: formData.password }));

    setFormData({
      email: '',
      password: '',
    });
  };
```

이렇게 되면 사가에선 다음과 같은 함수들이 순서대로 실행이 된다.
1. 워쳐 사가 (`watcher Saga`) 인 `watchLogin` 사가가 실행
2. `watchLogin`은 워커 사가 (`worker Saga`)인 `login` 사가를 호출
3. `login` 사가는 loginAPI를 실행하고 성공 시 `LOGIN_SUCCESS` 액션을, 실패 시 `LOGIN_FAILURE` 액션을 `yield put()` 으로 디스패치.

우리가 테스트를 할 부분은 바로 두번째와 세번째 순서가 된다.  
아래는 `login` 사가 코드다.

```typescript
export function* login(action: LoginRequestAction): SagaIterator {
  try {
    const response: any = yield call(loginAPI, action.data);
    yield put({
      type: LOGIN_SUCCESS,
      payload: response.data,
    });
  } catch (err: any) {
    yield put({
      type: LOGIN_FAILURE,
      error: err.response.data.message,
    });
  }
}
```
이제 우리는 `loginAPI`가 
1. 성공을 했을 때 `LOGIN_SUCCESS` 액션이 디스패치 된다는 것과 
2. 실패 했을 때 `LOGIN_FAILURE` 액션이 디스패치 된다는 것을 테스트 하면 된다.

# Login is successful
이 테스트 케이스를 코딩하기 시작할 때 중요한 부분은 **조건** 을 확실하게 아는 것 이라고 생각한다. 이 테스트 케이스의 조건은 *로그인이 성공적이다* 이다. 다시 말하자면 위의 `login` 사가 코드의  `response` 에 `response.data`에 데이터가 들어왔다는 뜻이다. 우리는 이 조건을 *모킹, mocking* 을 먼저 해야한다.

```typescript
const mockResponse = {data: { name: 'joon2', grade: 'bronze'}}
```

그 다음에 우리가 모킹을 해줘야 하는 함수는 `loginAPI` 함수이다. 이 함수에서 `axios.put` 이 실행되는데, 우리는 이 함수가 실행됐을 때 무조건 위에 정의 해놓은 `mockResponse`가 반환되도록 해야한다. 그래야 우리가 이 테스트 케이스를 테스트 하기 위한 *전제 조건* 이 완성되는 것이다.

```typescript
(axios.post as jest.Mock).mockResolvedValueOnce(mockResponse)
```

그리고 마지막으로 한 가지가 더 남았다. `login` 워커 사가가 실행되기 위해서는 이 워커 사가를 호출하는 *워커 사가* 가 필요하고 이 워커 사가는 리퀘스트 액션을 가로챈다. 따라서 이 리퀘스트 액션도 모킹을 해줘야 비로소 이 워커 사가를 실행시키기 위한 준비가 끝나는 것이다.

```typescript
const fakeAction = loginRequest({email: 'joon2@gmail.com', password: 'password123'})
```

자, 이제 워커 사가를 실행하면 된다. 워커 사가를 실행하는 함수는 `redux-saga` 모듈에서 제공하는 `runSaga()` 함수이다.  
>이 함수는 실제 Redux Store 와 연결하지 않고도 사가 로직을 독립적으로 실행하고 사가 함수의 동작을 검증할 수 있게 해준다. Redux Store 의 의존성을 제거하고, 특정 사가가 올바르게 액션을 처리하는지 그리고 디스패치 하는지 테스트를 하기 위한 도구 이다.

이 함수는 다음과 같은 인자를 필요로 한다.
- dispatch 객체 함수 `{dispatch: (action) => dispatched.push(action)}`
  - 이 함수는 `runSaga` 에서 디스패치 된 액션을 수집해주는 함수이다. 디스패치된 액션들을 수집하기 때문에, `dispatched` 라는 배열이 빈 배열로 초기 설정이 되어있어야 한다.
- 워커 사가 함수: 실행되어야 하는 워커 사가 함수가 들어간다.
- 모킹된 요청 액션: 워커 사가 함수를 실행할 때 필요한 요청 액션

이 runSaga 함수는 비동기로 실행되기 때문에 `await` 키워드를 앞에 붙여주고 반환 값을 프로미스로 변환해줘야 한다. 

이를 종합하면 코드가 다음과 같이 완성된다.
```typescript
await runSaga(
    {
        dispatch: (action) => dispatched.push(action)
    },
    login,
    fakeAction
).toPromise();
```

이렇게 되면 dispatched의 배열에 추가된 액션은 `LOGIN_SUCCESS` 액션이 이 될 것이다. 이것은 실제로 디스패치 될 `loginSuccess(mockResponse.data)` 와 같아야 하기 때문에 우리는 다음과 같은 `assertion` 코드를 작성할 수 있다.

```typescript
expect(dispatched[0]).toEqual(loginSuccess(mockResponse.data))
```

# Login failed
로그인이 실패 했을 시는 로그인이 성공했을 시와 거의 비슷하다. 이 경우의 조건은 loginAPI 함수가 에러를 발생시킨다는 것이다. 따라서 우리는 이 경우에는 `mockErrorResponse`를 정의해줘야 한다.
```typescript
const mockErrorResponse = {
    response: {
        data: {
            message: 'user not found'
        }
    }
}
```

여기에 들어갈 메세지는 임의로 정해주면 된다. 나는 잘못된 이메일을 넣었을 경우를 생각했기 때문에 백엔드에서 *user not found* 라는 메세지를 날릴 것을 알고 있기 때문에 저렇게 적었다.

그 다음에 성공 케이스와 같이 `axios.post`를 모킹해줘야 한다. 하지만 여기서 주의해야 할 점은 모킹 된 `axios.post`는 실패를 해야하기 때문에 `mockResolvedValueOnce` 가 아닌 `mockRejectedValueOnce`를 사용해야 한다는 것이다.

```typescript
(axios.post as jest.Mock).mockRejectedValueOnce(mockErrorResponse)
```

`fakeAction`과 `runSaga`는 성공했을 때와 똑같이 해줘도 된다. 그렇게 되면 `dispatched` 배열에 추가된 액션은 이제 `LOGIN_FAILURE` 이 될 것이고 이것은 실제 디스패치될 액션인 `loginFailure(mockErrorResponse.response.data.message)` 와 같을 것이다. 따라서 우리는 다음과 같은 `assertion` 코드를 작성할 수 있을 것이다.

```typescript
expect(dispatched[0]).toEqual(loginFailure(mockErrorResponse.response.data.message))
```

# Conclusion
오늘은 실제 사가에서 디스패치되는 액션들에 대한 테스트에 대해 알아보았다. 사가가 액션을 어떻게 디스패치 하는지는 [Redux Saga](https://hunjoonrhee.github.io/frontend/Saga) 를 보면 자세히 설명이 나와있다.  
사가에서 디스패치하는 액션을 테스트 하기 위해서는 실제 요청 액션을 처리하는 *워커 사가* 를 테스트에서 실행시켜야 하고 이것을 실행시키는데 사용하는 테스트 도구가 `runSaga` 함수이다.  
이 함수는 *워커 사가* 를 요청 액션과 함께 실행시키고, 그때 발생하는 액션들을 수집을 한다. 그래서 우리는 그 수집된 액션이 실제 디스패치 되는 액션과 같다는 사실을 테스트 하면 되는 것이다. 이때 요청 액션은 *fakeAction*으로 모킹된 액션을 사용하면 된다.