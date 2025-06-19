---
layout: single
title: "Angular: FormControl과 Signal을 함께 쓸 때 겪는 문제들 정리 (2)"
categories: [Angular]
tags: [FormControl, Signal]
search: true
---

# 1️⃣ RouterLink
주어진 코드에 `routerLink`는 있었고, 추가적으로 `<route-outlet />`를 사용해야해서 `RouterOutlet` 을 import 했는데, 계속 routerLink가 작동이 안되는 문제가 있었다. 당연했다. RouterLink에 대한 모듈은 임포트 하지 않았었으니까. RouterOutlet과 RouterLink를 둘다 포함하는 모듈인 `RouterModule`을 임포트 했어야 했다.

**질문:**  
RouterLink가 작동하려면 RouterModule이 꼭 imports 되어야 하나?

**핵심:**  
✔️ 맞음 — RouterLink directive는 RouterModule이 export 한다.  
`imports: [RouterModule]`이 없으면 `<a routerLink="...">` 에서 에러남.

**예시:**

```ts
import { RouterModule } from '@angular/router';

@Component({
  standalone: true,
  selector: 'app-header',
  imports: [RouterModule],
  template: \`
    <a routerLink="/home">Home</a>
    <a routerLink="/details">Details</a>
  \`
})
export class HeaderComponent {}
```

---

## 2️⃣ Checkbox 상태 관리
체크박스나 input 같은 form 의 상태관리는 `formControl`을 사용하는 방법과 signal + [checked] + (change)를 사용하는 방법이 있다.

✅ (a) FormControl 버전
```html
<div>
  @if(options?.towHitch) { Tow hitch?
  <input type="checkbox" [formControl]="towHitchControl" />
  }
</div>
```

```ts
export class OptionsComponent {
  towHitchControl = new FormControl<boolean>(false);

  ngOnInit() {
    this.towHitchControl.valueChanges.subscribe((tH) => {
      if (tH) {
        this.service.currentCarHitchYokeOptions.update((prev) => ({
          ...prev,
          towHitch: tH,
        }));
      }
    });
  }
}
```
✍️ `valueChanges.subscribe()` 를 `ngOnInit()` 에 넣는 이유
`ngOnInit()`은 **컴포넌트 인스턴스가 처음 생성되고 렌더링되기 전에 1회 호출**된다. 즉, 초기 구독, 초기 데이터 fetch, 초기 상태 세팅에 사용된다. 이후에 컴포넌트가 파괴되기 전까지는 다시 실행되지 않는다.

❓ valueChanges.subscribe가 왜 ngOnInit에 맞는가?
- `FormControl`은 초기값만 있으면 `.valueChanges`스트림은 언제든지 동작한다.
- `ngOninit()`에서 `subscribe` 하면:
1️⃣ 컴포넌트가 초기화되자마자 구독 시작
2️⃣ 이후에 사용자가 FormControl을 변경할 때마다 콜백 작동
3️⃣ 새로운 값이 들어올 때마다 Service로 전달
즉, **렌더링과 상관없이, 구독은 한 번만 연결하면 계속 작동**하기 때문에 `ngOnInit` 이 적절하다.

✅ (b) Signal + [checked] + (change) 버전
```html
<input
    type="checkbox"
    [checked]="service.currentCarHitchYokeOptions().yoke"
    (change)="toggleYoke($event)"
  />
  }
```
```ts
toggleYoke(event: Event) {
    const checked = (event.target as HTMLInputElement).checked;
    this.service.currentCarHitchYokeOptions.update((prev) => ({
      ...prev,
      yoke: checked,
    }));
  }
```
기본적인 형태로, `(change)` 함수를 통해 `checked`의 값을 지정하고 지정된 값을 `[checked]`에 넣어주는 것이다.

---

## 3️⃣ Step 이동 후 상태 유지 안됨

문제는 다음과 같았다. 
Step1 페이지에서 모델 및 색상을 선택하면 Step 2로 이동이 되고, Step 2에서 옵션을 선택하면 Step 3의 최종 페이지로 이동할 수 있게 된다. Step 3에서 다시 Step 1이나 Step 2로 갔을 때 선택 했던 값들이 유지가 되지 않고 초기화가 되었다. 

문제의 핵심은 `FormControl`에만 값을 저장하면 컴포넌트가 새로 생성될 때 값이 날라간다. 그래서 Step 3 에서 이 전 스텝으로 갔을 때 선택했던 상태가 유지되지 않았던 것이다. 이럴 때는 상태를 `FormControl` 저장할 것이 아니라 Service에 저장을 하고 `FormControl` 초기화 될 때 초기 값을  `Service` 상태로 세팅해야 한다.

**예시:**

```ts
export class ConfiguratorService {
  readonly selectedCar = signal<string>(''); // model code
  readonly selectedColor = signal<string>(''); // color code
}

export class Step1Component {
  configurator = inject(ConfiguratorService);

  carModelControl = new FormControl(this.configurator.selectedCar());
  carColorControl = new FormControl(this.configurator.selectedColor());

  constructor() {
    this.carModelControl.valueChanges.subscribe(v =>
      this.configurator.selectedCar.set(v)
    );
    this.carColorControl.valueChanges.subscribe(v =>
      this.configurator.selectedColor.set(v)
    );
  }
}
```

---

## 4️⃣ FormControl 숫자 비교 안됨
FormControl에서 선택된 값의 타입이 number 였는데, 그 값을 서비스에서 타입이 number 인 다른 값과 비교를 하는데 자꾸 비교가 안되는 상황이 생겼다. 알고보니 `<select>`의 `[value]`는  HTML 표준상 무조건 string이었다. 그래서 비교를 하려면 FormControl의 value 를 number 로 변환해주거나 [ngValue] 를 사용해서 nubmer 타입을 유지해줘야 한다.

**예시:**

```ts
// 틀림
if (formControl.value === config.id) {
  // false (string === number)
}

// 올바름
if (Number(formControl.value) === config.id) {
  // true
}
```

또는 `<option>`에서 `[ngValue]` 사용해서 number 타입 유지:

```html
<option [ngValue]="config.id">{{ config.name }}</option>
```

---

## 5️⃣ [value] vs [ngValue]

**질문:**  
`[value]`와 `[ngValue]` 차이?

**핵심:**

| 속성 | `[value]` | `[ngValue]` |
|------|----------------|----------------------------|
| HTML 표준 | O | X (Angular 전용) |
| 타입 | 항상 string | 타입 그대로 유지 |
| FormControl 저장값 | string | number/object 가능 |
| 추천 | 간단한 string | 숫자 ID, 객체 넘기기 |

**예시:**

```html
<!-- [value]: string -->
<option [value]="1">One</option>

<!-- [ngValue]: 타입 그대로 -->
<option [ngValue]="1">One</option>
<option [ngValue]="someObject">Obj</option>
```
---

# ✅ 전체 핵심 요약

- RouterLink → RouterModule 필수
- Checkbox → FormControl or Signal 사용
- Step 상태 유지 → Service + FormControl 초기화
- Select 비교 안 맞음 → [ngValue] or Number()
👍 이 내용은 실전에서 자주 겪는 문제이니, 앞으로 비슷한 상황에서 바로 떠올릴 수 있도록 기억해두자.