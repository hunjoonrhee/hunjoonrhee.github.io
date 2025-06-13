---
layout: single
title: "Angular: FormControl과 Signal을 함께 쓸 때 겪는 문제들 정리"
categories: [Angular]
tags: [FormControl, Signal]
search: true
---

# Introduction
최근 Angular Level 2 시험을 준비하며 코딩 과제를 하다 다음과 같은 문제들을 겪었다.
- 차 모델을 선택하면 해당 색상 리스트가 동적으로 바뀜
- 모델과 색상 리스트는 `Signal`로 관리됨
- `FormControl`을 통해 `<select>` 를 바인딩 함.

그런데...
✅ **색상이 초기값으로 선택되어도 드롭다운에 표시되지 않는 현상 발생!**  
✅ **드롭다운 값은 들어가는데 화면엔 아무것도 안 뜸!**

# 문제 1: `<select>`에 값이 있는데 드롭다운엔 표시 안됨
```html
<select [formControl]="carColorControl">
  <option *ngFor="let color of colors" [value]="color.code">
    {{ color.description }}
  </option>
</select>
```

```ts
this.carColorControl.setValue('Deep Blue Metallic'); // 분명히 값은 들어감
```
### 📌 원인
`[value]="color.code"`인데, `setValue('Deep Blue Metallic')`처럼 **description 값**을 넣어서 드롭다운과 일치하지 않음.

### ✅ 해결
```ts
this.carColorControl.setValue(color.code); // value와 정확히 일치시켜야 함
```

---

## 문제 2: 모델 선택 시 색상 자동 초기화가 안됨
차 모델이 선택되고 나면 선택된 모델의 색상 리스트가 드롭다운으로 나와야했다. 그리고 차의 색상은 모델이 바뀌면 항상 첫번째 색상이 선택이 되도록 하는것이 과제였다. 
```ts
const carModelControl = new FormControl<string>('');

this.carModelControl.valueChanges.subscribe((m) => {
  const currentModel = this.allModels.find(
        (model) => model.description === m
      );
})
```
으로 모델을 찾고 난 뒤 모델이 찾아지면, 즉 `currentModel` 이 `undefined`가 아니면 `carColorControl`의 값을 정해줘야한다. 
즉 코드는 다음과 같아야 한다.
```ts
this.carModelControl.valueChanges.subscribe(index => {
  const car = this.configuratorService.allModels()[index];
  this.configuratorService.setCurrentCar(index);

  // ✅ 자동 초기화
  const firstColor = car.colors[0];
  this.carColorControl.setValue(firstColor.code);
  this.configuratorService.setCurrentColorCode(firstColor.code);
});
```

## 문제 3: Signal과 FormControl이 자동으로 연결되지 않음
마지막 문제는 `carColorControl`에서 초기화된 값이 UI에는 나오지 않았다. 의외로 간단하게 해결할 수 있었다. 문제 2에서 나는 초기 세팅 값을 색상 코드, `color.code`로 하였다. 
원인은 `<select>` 안에 있는 `<option>`에 `[value]` 값을 주지 않아서 선택된 값과 초기화된 색상 코드가 일치하지 않았기 때문이었다.
옵션에 `[value]`를 주지 않으면, 자동으로 `[value]`는 UI 에 나오는 옵션 텍스트 값으로 지정이 된다. 
```html
<select [formControl]="carColor">
  @if(configuratorService.currentCar()) { @for(color of
  configuratorService.currentCar()!.colors; track color.code) {
  <option>{{ color.description }}</option>
  } }
</select>
```
이런 상황이라면 `[value]`는 색상의 description이 된다. 하지만 초기 세팅값을 색상 코드로 하였기 때문에 차 모델이 바뀔 때마다 초기 세팅된 색상을 찾지 못하는 상황이 생긴 것이다.
결론은 `FormControl` 에 세팅되는 값의 타입과 `<option>` 또는 `<input>` 에 사용되는 `value`의 타입은 일치해야 한다는 것이다.
문제는 다음과 같은 코드로 해결이 되었다.
```html
<select [formControl]="carColor">
  @if(configuratorService.currentCar()) { @for(color of
  configuratorService.currentCar()!.colors; track color.code) {
  <option [value]="color.code">{{ color.description }}</option>
  } }
</select>
```
---

## 요약: 자주 틀리는 5가지 실전 포인트

| 문제 | 원인 | 해결 |
|------|------|------|
| 드롭다운에 값이 안 보임 | `[value]`와 `setValue()` 값이 다름 | `code`를 기준으로 통일 |
| 모델 바뀔 때 색상 초기화 안 됨 | 수동 초기화 코드 없음 | `setValue()`로 첫 값 세팅 |
| `FormControl`에 값이 들어와도 바인딩 안 됨 | `<option [value]>` 불일치 | 정확히 일치시키기 |

---

## 결론

Angular에서 Signal과 FormControl을 함께 쓸 땐 아래를 꼭 기억하자:

- `[value]`와 `setValue()`의 값은 **완벽하게 일치**해야 한다
- 모델 변경 시 종속된 값들(FormControl, Signal)은 **직접 초기화**
- Signal과 FormControl은 자동 동기화되지 않음 → `subscribe()` 또는 `effect()` 사용 필요

---

👍 이 내용은 실전에서 자주 겪는 문제이니, 앞으로 비슷한 상황에서 바로 떠올릴 수 있도록 기억해두자.