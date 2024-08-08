---
layout: single
title:  "Code Review (RegisterForm.tsx)"
categories: Code Review
tags: [Next.js, Form-Component]
search: true
---

# Code Review

무심코 챗 지피티에게 "너 코드 리뷰도 할줄 아냐?" 라고 물었더니 가능하다고 했다. 한 번에 믿지 않고 나는 다시 한 번 물었다. "내가 너에게 어떤 역할 및 경력을 주면 거기에 맞게 코드 리뷰가 가능하냐?" 라고. 그랬더니 가능하다고 했다. 그래서 나의 코드를 한 번 넘겨줬다. 지피티의 피드백을 기록 해보려고 한다.

## RegisterForm.tsx

`Next.js` 버전 14 를 사용하며, `tailwind-css` 프레임워크를 사용하고 있다. 그리고 다음의 코드는 회원가입 페이지에 들어가는 회원가입 폼 (RegisterForm) 컴포넌트이다.

```typescript
'use client';

import { useState, ChangeEvent, useEffect, FormEvent } from 'react';

import Image from 'next/image';

const RegisterForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: '',
    confirmPassword: '',
    address: '',
    phone: '',
    policyyn: false,
  });

  const [confirmPasswordError, setConfirmPasswordError] = useState(false);
  const [checkPolicyError, setCheckPolicyError] = useState(false);
  const [nameError, setNameError] = useState(false);
  const [emailError, setEmailError] = useState(false);
  const [passwordError, setPasswordError] = useState(false);

  const handleOnChange = (event: ChangeEvent<HTMLInputElement>) => {
    const targetName = event.target.name;
    const targetValue = 
    event.target.type === 'checkbox' ? event.target.checked : event.target.value;
    setFormData({ ...formData, [targetName]: targetValue });
  };

  useEffect(() => {
    if (formData.confirmPassword && formData.password) {
      if (formData.password !== formData.confirmPassword) {
        setConfirmPasswordError(true);
      } else {
        setConfirmPasswordError(false);
      }
    }
  }, [formData.confirmPassword, formData.password]);

  const handleOnSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    setEmailError(false);
    setCheckPolicyError(false);
    setNameError(false);
    setPasswordError(false);

    if (!formData.email) {
      setEmailError(true);
      return;
    }
    if (!formData.name) {
      setNameError(true);
      return;
    }
    if (!formData.password) {
      setPasswordError(true);
      return;
    }
    if (!formData.confirmPassword) {
      setPasswordError(true);
      return;
    }
    if (!formData.policyyn) {
      setCheckPolicyError(true);
      return;
    }

    alert('register!');
    setFormData({
      name: '',
      email: '',
      password: '',
      confirmPassword: '',
      address: '',
      phone: '',
      policyyn: false,
    });
  };

  return (
    <div className="flex flex-col sm:flex-row items-center justify-center mr-2 ml-2 max-w-full">
      <div className="w-1/3 mr-0 sm:mr-3">
        <Image src={'/images/register.png'} alt="register" width={400} height={400} />
      </div>
      <form onSubmit={handleOnSubmit}>
        <div className="border-none border-gray-400 p-10 w-96 shadow-md rounded-lg ml-0 sm:ml-3">
          <h2 className="text-xl font-semibold leading-7 text-gray-900 text-center"> Register </h2>
          <div className="mt-2">
            <label htmlFor="name" className="block w-full text-sm font-medium leading-6 text-gray-900">
              Name
            </label>
            <div className="mt-2">
              <input
                type="text"
                name="name"
                placeholder="enter your name"
                value={formData.name}
                onChange={handleOnChange}
                className="w-full h-10 p-2 rounded-lg ring-1 ring-inset ring-gray-400 focus:ring-2
                          focus:ring-inset focus:ring-green-700 shadow-md placeholder-left"
              />
            </div>
          </div>
          {nameError && <p className="text-xs text-red-600 mt-0">This field is mandatory</p>}
          <div className="mt-2">
            <label htmlFor="email" className="block w-full text-sm font-medium leading-6 text-gray-900">
              Email
            </label>
            <div className="mt-2">
              <input
                type="email"
                name="email"
                placeholder="enter your email"
                value={formData.email}
                onChange={handleOnChange}
                className="w-full h-10 p-2 rounded-lg ring-1 ring-inset ring-gray-400 focus:ring-2 
                          focus:ring-inset focus:ring-green-700 shadow-md placeholder-left"
              />
            </div>
          </div>
          {emailError && <p className="text-xs text-red-600 mt-0">This field is mandatory</p>}
          ...
          <div className="mt-4 flex flex-row items-center justify-between">
            <input type="checkbox" name="policyyn" className="form-checkbox h-4 w-4" 
              checked={formData.policyyn} onChange={handleOnChange} />
            <label htmlFor="policyyn" className="text-gray-900">
              I agree to the terms and conditions.
            </label>
          </div>
          {checkPolicyError && <p className="text-xs text-red-600 mt-0">You should agree policy</p>}
          <div className="mt-4 flex justify-end">
            <button type="submit" className="border rounded-lg border-gray-400 p-2 
                    shadow-sm text-green-700 hover:bg-green-700 hover:text-white">
              Register
            </button>
          </div>
        </div>
      </form>
    </div>
  );
};
export default RegisterForm;
```

내가 이 코드를 쓰면서 나 스스로 느낀점은 상태 관리 useState가 너무 많이 반복된다는 점과, 에러 메시지가 동일하게 들어간다는 점, 그리고 그 에러메시지가 컴포넌트에서 하드코딩이 된다는 점이었다. 그리고 `tailwind` 의 같을 클래스 네임이 너무 반복된다는 점이었다.

다음은 지피티의 피드백이다.

#### 전체적인 평가
코드의 전반적인 구조와 스타일은 잘 짜여져 있으며, 최신 `React` 및 `Next.js` 기능을 사용하고 있다. 특히, 상태 관리를 `useState` 훅을 통해 잘 처리하고 있으며, `useEffect` 를 사용하여 비밀번호 확인 로직을 구현한 점도 좋다. 그러나 몇 가지 개선할 수 있는 부분들이 있다.

#### 개선 사항
1. 상태 관리 최적화
   - 여러 개의 `useState` 훅을 사용하여 에러 상태를 관리하고 있는데
     ```typescript
      const [confirmPasswordError, setConfirmPasswordError] = useState(false);
      const [checkPolicyError, setCheckPolicyError] = useState(false);
      const [nameError, setNameError] = useState(false);
      const [emailError, setEmailError] = useState(false);
      const [passwordError, setPasswordError] = useState(false);
     ```
   이는 코드가 복잡해지고 유지보수가 어려워질 수 있다. 모든 에러 상태를 하나의 객체로 관리하는 것이 좋다.
   ```typescript
      const [errors, setErrors] = useState({
            confirmPassword: false,
            checkPolicy: false,
            name: false,
            email: false,
            password: false,
         });
   ```
2. 중복 코드 제거
   - 에러 메시지 출력 부분에서 중복된 코드가 많다. 이를 함수화하여 코드의 중복을 줄일 수 있다.
   ```typescript
   const renderError = (error) => error && <p className="text-xs text-red-600 mt-0">This field is mandatory</p>;
   ```
3. 유효성 검사 함수 분리
   - 유효성 검사 로직을 `handleOnSubmit` 함수 내에 직접 작성하는 대신 별도의 함수로 분리하면 가독성과 재사용성을 높일 수 있다.
  
   <img src="../../images/2024-08-07-CodeReview/codereview1.png" alt="codereview1" style="zoom:50%;" />
4. 스타일링 최적화
   - 클래스 네임이 중복되는 부분은 변수를 관리해 사용하는걸 추천한다. 이는 스타일 변경 시 일관성을 유지하는데 도움이 된다.
   ```typescript
   const inputClassName = "w-full h-10 p-2 rounded-lg ring-1 ring-inset ring-gray-400 focus:ring-2 focus:ring-inset focus:ring-green-700 shadow-md placeholder-left";
   ```
5. 불필요한 `useEffect` 호출 방지
   - 유효성 검사를 따로 함수로 빼서 관리를 하게 되면 현재 `useEffect` 는 불필요하게 되므로 삭제를 하는것이 좋다.
  
## 결론
이 코드에서 내가 찜찜하게 느꼈던 부분들을 지피티가 고쳐주었다. 반복되는 부분들은 최소화를 시키고, 중복 코드를 제거하고 `useEffect`가 불필요한지 한 번 더 확인해야겠다.
