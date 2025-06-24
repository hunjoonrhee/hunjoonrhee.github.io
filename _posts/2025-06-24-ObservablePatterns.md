---
layout: single
title: "🚗 Angular Observable 패턴 정리 (movies$ vs currentCar$)"
categories: [Angular]
tags: [Observable]
search: true
---

# ✅ 1️⃣ 목표
오늘은 Angular의 `Observable`에 대해서 공부한 것을 정리해보려고 한다. 두 가지 케이스를 통해서 `Observable`로 들어온 변수를 어떻게 필터링 하는지, 그리고 상태 관리는 어떻게 하는지 알아보겠다. 
- 서버에서 옵저버블로 넘어온 `movies$`를 인풋 필드에 입력된 텍스트로 필터링하는 과정
- 자동차 모델을 선택하면 해당 모델의 색상 리스트를 선택할 수 있는 드롭 다운 메뉴가 생기고, 색상 선택 시 자동으로 해당 모델의 이미지가 세팅되는 과정

---

## ✅ 2️⃣ `movies$` & `filteredMovies$` 예시

### 📌 흐름

```ts
readonly movies$ = this.moviesService.getMovies();
readonly titleControl = new FormControl('');
readonly yearControl = new FormControl('');

readonly filteredMovies$ = combineLatest([
  this.movies$,
  this.titleControl.valueChanges.pipe(startWith('')),
  this.yearControl.valueChanges.pipe(startWith('')),
]).pipe(
  map(([movies,title,year]) =>
    movies.filter(m => 
      (!title || m.title.toLowerCase().includes(title.toLowerCase())) &&
      (!year || m.release_date.includes(year))
    )
  )
);
```

여기서 어려웠던 점을 필터를 해야할 대상도 옵저버블이고, `FormControl`을 통해 들어온 `title`과 `year` 도 `valueChanges()`를 통해 옵저버블로 처리를 해야한다는 점이었다. 즉 세 개의 옵저버블이 있었고, 이걸 동시에 처리를 해야했다. 
이럴 때 필요한 것이 `combineLatest` 이다. 이 `combineLatest`는 옵저버블 변수의 최신 값들을 하나로 묶어서 처리를 할 수 있게 해준다. 따라서 이 `combineLatest`로 묶인 세 가지 옵저버블에 `.pipe()`와 `.map()` 을 동시에 적용할 수 있는 것이다.

또 여기서 알아야 했던 점이 `.valueChanges()`를 그냥 쓰지 않고 `.valueChanges.pipe(startWith(''))` 로 썼다는 점이다.
`startWith(value)` 는 해당 `Observable`이 구독되자마자, 원래 값이 발생하기 전에 `value`를 먼저 흘려보내도록 만드는 연산자이다.

예를 들어:
```ts
this.titleControl.valueChanges.pipe(startWith(''));
```
- `valueChanges`는 **기본적으로 폼 컨트롤이 바뀔 때만 값 발행** -> 즉, 아무 입력도 안하면 `valueChanges`는 값을 발행하지 않는다.
- `startWith('')`를 붙이면 -> `valueChanges`가 발행되기 전에 빈 문자열을 최초 한 번 흘려보낸다. 즉시 ''로 시작. 옵저버블이 항상 초기값을 가진다.

그렇다면 왜 쓰는 것일까?
`combineLatest` 또는 `switchMap` 할 때 **각 `Observable`이 최소 1개 값은 있어야 동작**하기 때문이다.
```ts
combineLatest([
  this.movies$,
  this.titleControl.valueChanges.pipe(startWith(''))
])
```
- 만약 `startWith` 없이 `titleControl.valueChanges` 가 아무 입력 없으면?
👉 `combineLatest`는 값을 기다리느라 아무것도 안 발행함!
- `startWith('')` 있으면?
👉 `movies$` 가 `emit` 되자마자 ' ' 도 같이 `emit` → 바로 필터 로직 수행 가능!

위의 예시는 combineLatest만 썼지만, 다른 버전도 있다. 아래의 버전은 combineLatest 와 switchMap을 같이 쓴 경우이다.
```ts
filteredMovies$ = combineLatest([
    this.titleControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      distinctUntilChanged()
    ),
    this.yearControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      distinctUntilChanged()
    ),
  ]).pipe(
    map(
      ([title, year]) =>
        [title?.trim(), year?.toString().trim()] as [string, string]
    ),
    switchMap(([title, year]) =>
      this.movies$.pipe(
        map((movies) => {
          const hasTitle = title ? title.length > 0 : false;
          const hasYear = year ? year.length > 0 : false;
          if (!hasTitle && !hasYear) {
            return null;
          }
          return movies.filter(
            (m) =>
              m.title.toLowerCase().includes(title?.toLowerCase() || '') &&
              m.release_date.includes(year || '')
          );
        })
      )
    )
  );
```
이 코드에서는 일단 두개의 `formControl` 을 `combineLatest`로 묶어서 가장 최신 값을 `.pipe()`와 `.map()`으로 뽑아냈다. 추출된 값은 다시 한 번 `trim()`을 통해서 빈 공간을 없애줬다. 
이 값들이 이제 movies$ 라는 옵저버블 필터에 쓰여야 한다. 즉, `.pipe()`와 `.map()`을 쓰는 주된 옵저버블이 바뀌어야 한다는 셈이다. 이 때 우리는 `switchMap`을 쓸 수 있다. `formControl()`의 `pipe()` 에서 `title`과 `year`를 갖고 `movies$` 옵저버블로 스위치를 한다. 이때 `map()`과 `switchMap`은 ',' 로 같이 쓰면 된다. `switchMap`에서 우리는 `title`과 `year`를 갖고 주된 옵저버블을 `movies$`로 바꿔준다. 그 다음에는 위의 예와 똑같이 `.pipe()` 와 `.map()`으로 `movies`를 필터해주면 된다.

### ✔️ 핵심

- 여러개의 옵저버블을 동시에 처리해야 할 때는`combineLatest` + `map`
- 주 옵저버블을 바꾸고 싶을 때는 현재 옵저버블 파이프 안에서 `switchmap`
- `combineLatest`를 쓸 때 `valueChanges`가 있으면 꼭 `.pipe(startWith(''))`로 `formControl` 에 초기값

---

## ✅ 3️⃣ `currentCar$` & `currentCarColor$`
이번 케이스는 앞의 케이스와는 약간 다르다. 
먼저 차 모델을 고르는 드롭다운 메뉴가 있다. 여기서 차 모델을 선택하게 되면 해당 모델의 색상 메뉴가 드롭다운으로 나온다. 모델의 색상까지 선택하게 되면 선택된 색상과 모델의 이미지가 나오게 된다.

이걸 만약에 signal로 처리를 한다고 했을 때, valueChanges()에서 나온 값을 구독해서 시그널로 상태를 저장할 것이다. 하지만 우리는 이것을 옵저버블로 처리를 해야한다.
옵저버블로 처리를 할 때 시그널과 비슷하게 상태를 처리할 수 있게 해주는 것이 BehaviorSubject()이다. 

서비스 단에 BehavorSubject()로 차 모델의 상태를 저장할 수 있는 "저장소"를 만들어준다. 
```ts
currentCarSubject = new BehaviorSubject<CarModel | null>(null);
```
위의 코드를 보면 알 수 있듯이 BehaviorSubject를 만들 때는 반드시 초기값을 줘야한다. 타입스크립트라면 해당 변수의 타입까지 같이 지정해야 한다.
BehaviorSubject는 가장 최근의 값을 항상 기억한다. 그리고 값을 지정할 때, 즉 setState를 하고 싶을 땐 setCurrentCar() 함수를 만들어주고 BehaviorSubject의 next() 메서드로 값을 갱신한다. 갱신하면 자동으로 구독자에게 알려준다. 

따라서 우리는 서비스단에 다음과 같이 상태관리 변수와 상태 세팅 함수들을 만들어 놓을 수 있다.
```ts
@Injectable({ providedIn: 'root' })
export class ConfiguratorService {
  private currentCarSubject = new BehaviorSubject<CarModel | null>(null);
  readonly currentCar$ = this.currentCarSubject.asObservable();

  private currentCarColorSubject = new BehaviorSubject<Color | null>(null);
  readonly currentCarColor$ = this.currentCarColorSubject.asObservable();

  private currentCarImageSubject = new BehaviorSubject<string | null>(null);
  readonly currentCarImage$ = this.currentCarImageSubject.asObservable();

  setCurrentCar(car: CarModel) {
    this.currentCarSubject.next(car);
  }

  setCurrentCarColor(color: Color) {
    this.currentCarColorSubject.next(color);
  }

  setCurrentCarImage(url: string) {
    this.currentCarImageSubject.next(url);
  }
}
```

이걸 이제 컴포넌트와 연동을 해야한다.
### 📌 컴포넌트 연동

```ts
export class Step1Component implements OnInit {
  readonly configuratorService = inject(ConfiguratorService);
  readonly allModels$ = this.configuratorService.allModels$;

  readonly carModelControl = new FormControl('');
  readonly carColorControl = new FormControl('');

  ngOnInit() {
    // 모델 선택
    this.carModelControl.valueChanges.subscribe(modelDescription => {
      this.allModels$.subscribe(models => {
        const selected = models.find(m => m.description === modelDescription);
        if (selected) {
          this.configuratorService.setCurrentCar(selected);
          const firstColor = selected.colors[0];
          if (firstColor) {
            this.carColorControl.setValue(firstColor.code);
            this.configuratorService.setCurrentCarColor(firstColor);
          }
        }
      });
    });

    // 색상 선택
    this.carColorControl.valueChanges.subscribe(code => {
      const car = this.configuratorService.currentCarSubject.getValue();
      const color = car?.colors.find(c => c.code === code);
      if (color && car) {
        this.configuratorService.setCurrentCarColor(color);
        const url = `https://site.com/${car.code}/${color.code}.jpg`;
        this.configuratorService.setCurrentCarImage(url);
      }
    });
  }
}
```
앞의 movies 예시와 다른 점은 드롭다운 메뉴로 선택된 값을 우리는 저장하여 다른 값을 만들어 내기 위해 써야 한다는 것이다. 그래서 따로 combineLatest와 같은 메서드는 필요가 없고, valueChantes를 곧바로 구독을 하면 된다. 그래서 ngOnInit() 메서드에 넣어놓고 쓰면 된다.
첫 번째 모델 선택에서는 모델이 선택되고 나면, 그 선택된 모델을 `setCurrentCar` 메서드에 넣어서 상태를 저장해준다. 그리고 모델이 선택되자마자 해당 색상 리스트의 가장 첫번째 색이 자동 선택이 되어야 했기 때문에 `firstColor` 를 추출하여 carColorControl의 값을 setValue로 세팅해주고 setCurrentCarColor 메서드에 넣어서 색상 상태도 정해준다.

그 다음 색상을 고를 때 `formControl`에서 색상 코드만 구독이 된다는 사실을 주의해야 했다. 색상 코드만 오기 때문에, 우리는 현재 선택된 차 모델을 갖고와서 차 모델의 색상 리스트 중에서 선택한 코드와 맞는 색을 찾아내야 했다. 그리고 그 색을 다시 `setCurrentCarColor` 메서드로 세팅을 다시 해줘야 했다. 이때 필요했던 것이 `.getValue()` 라는 메서드였다. `BehaviorSubject`에서 현재 색상을 옵저버블 없이 값을 가져올때 사용하는 메서드가 `.getValue()` 이다. 이것을 통해서 현재 선택된 차 모델을 갖고오고, 그리고 색상을 추출해서 `imageUrl`를 만들어 `currentCarImage`에 저장을 할 수 있었던 것이다.


---

# ✅ 전체 핵심 요약

✅ 핵심 요약

1️⃣ 여러 Observable 동기 처리는 `combineLatest`
- `movies$`와 `FormControl`의 `valueChanges`를 하나의 스트림으로 묶어 최신값을 동시에 사용.
- `startWith('')`로 초기값 보장해서 `combineLatest`가 항상 동작하도록 설계.

2️⃣ 실시간 필터링은 `map` 또는 `switchMap`으로
- `combineLatest` + `map` → 클라이언트 필터링.
- `combineLatest` + `switchMap` → 값 변화 시마다 주 스트림(`movies$`)로 전환해서 처리.
- `debounceTime`, `distinctUntilChanged`로 불필요한 연산 방지.

3️⃣ 상태 관리는 `BehaviorSubject`로
- `BehaviorSubject`는 초기값 필수.
- 가장 최근 값을 메모리에 유지, `next()`로 갱신, `getValue()`로 현재 값 동기 조회 가능.
- 시그널 대신 `Observable` + `BehaviorSubject` 패턴으로 전환 가능.

4️⃣ `FormControl`과 상태를 수동 동기화
- 드롭다운 선택 변화 → `FormControl.setValue()`로 UI 값 동기화.
- 선택한 모델과 색상은 `BehaviorSubject`로 상태 저장.
- 저장된 상태를 이미지 URL 생성 등 파생 데이터에 활용.

5️⃣ 구독 위치와 구조
- 여러 값이 필요하면 `combineLatest`로 묶어 한 번에 처리.
- 드롭다운 같은 단일 선택은 `valueChanges`를 `ngOnInit`에서 직접 구독.
- 필요에 따라 `switchMap`으로 스트림 전환 설계.

👍 이 내용은 실전에서 자주 겪는 문제이니, 앞으로 비슷한 상황에서 바로 떠올릴 수 있도록 기억해두자.