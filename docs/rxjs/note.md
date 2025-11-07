# RxJS

## Observable

`Observable`은 여러 값을 모아놓은 Lazy Push 컬렉션입니다. 시간에 따라 여러 값을 emit 할 수 있습니다.

- of()

  - 여러 값들을 순차적으로 발행하고 완료

- from()

  - `Array`, `Promise`, `Iterable`을 `Observable`로 반환

- interval() / timer()

  - 주기적으로 값을 발행

## Subject

- **Subject**

  - `Observable`과 `Observer` 두 가지 역할을 동시에 하는 객체
  - `Multi Casting` : 여러 구독자에게 동시에 같은 값을 전달
  - 초기값이 없으며 구독 이후에 발행된 값만 받을 수 있음
  - ex) 이벤트 버스, 알림

- **BehaviorSubject**

  - 초기값이 필수이고 현재값을 항상 저장함
  - 새 구독자는 즉시 현재값만 받음
  - ex) 상태 관리, 설정값

- **ReplaySubject**

  - 지정된 개수만큼 이전 값을 저장함
  - 새 구독자는 저장된 모든 값을 즉시 받음
  - ex) 히스토리, 로그, 캐시

- **AsyncSubject**

  - 완료 시에만 마지막 값을 발행
  - 완료되기 전까지는 아무 값도 발행하지 않음
  - `Promise`와 유사하게 동작
  - ex) 최종 결과, Promise 대체

---

- #### formEvent

  DOM 이벤트 리스너를 Observable Stream으로 변환하는 함수

  - subscribe()
  - unsubscribe()
  - pipe()

- #### rxjs/operators

  pipe() 메소드 안에서 사용하는 흐름 제어 연산자들

  ##### 핵심 변환 연산자

  - **map()**

    - 값을 계산해서 반환
    - `map(v => v + 10)`

  - **mergeMap()**

    - 각 값을 Observable로 변환하고 병합 (순서 보장 x)
    - `mergeMap(v => of(v * 10).pipe())`

  - **switchMap()**

    - 새로운 값이 들어오면 이전 Observable을 취소하고 새로운 것으로 전환
    - `switchMap(term => this.http.get('/api/search?q=${term}'))`

  - **concatMap()**

    - 순서를 보장하며 이전 Observable이 완료될때까지 대기
    - `concatMap(v => this.http.post('/api/data', value))`

  ##### 필터링 연산자

  - **filter()**

    - 조건에 맞는 값만 통과
    - `filter(v => v > 2)`

  - **debounceTime()**

    - 마지막 이벤트 후 지정된 시간이 지난 후에만 값을 발행
    - `debounceTime(300)`

  - **distinctUntilChanged()**

    - 이전 값과 다를때만 발행
    - `distinctUntilChanged()`

  - **take() / takeUntil()**

    - 지정된 개수만큼 또는 특정 조건까지만 값을 받음
    - `take(3)`
    - `takeUntil(this.destory$)`

  ##### 결합 연산자

  - **combineLatest()**

    - 모든 Observable의 최신 값을 결합

      ```Typescript
      combineLatest([
          this.userService.getUser(),
          this.settingsService.getSettings()
      ])
      ```

  - **forkJoin()**

    - 모든 Observable이 완료될 때까지 대기 후 마지막 값들을 배열로 반환

      ```Typescript
      forkJoin({
          users: this.http.get('/api/users'),
          posts: this.http.get('/api/posts'),
          comments: this.http.get('/api/commnets')
      })
      ```

  - **merge()**

    - 여러 Observable을 하나로 병합

      ```Typescript
      merge(
        button1Click$,
        button2Click$
      )
      ```

### Angular Pattern

#### 1. Http 요청 처리

```Typescript
@Component({
    selector: "",
    templateUrl: ""
})
export class ExampleComponent implements OnInit {

    users$: Observable<User[]>

    constructor(
        private http: HttpClient
    ){}

    ngOnInit(){

        this.users$ = this.http.get<User[]>('/api/users').pipe(
            retry(3), // 실패 시 재시도 횟수
            map(users => users.filter(user => user.active)),
            catchError(error => {
                console.error('Error: ', error),

                return of([]);
            })
        )
    }
}
```

#### 2. 검색 기능 구현

```Typescript
@Component({
    selector: "",
    templateUrl: ""
})
export class SearchComponent implements OnInit {

    searchControl = new FormControl();

    results$: Observable<any[]>;

    constructor(private searchS: SearchService){}

    ngOnInit(){

        this.results$ = this.searchControl.valueChanges.pipe(
            debounceTime(300),          // 300ms 디바운스
            distinctUntilChanged(),     // 이전 값과 같으면 무시
            switchMap(term =>
                term ? this.searchS.search(term) : of([]);
            )
        )
    }
}
```

#### 3. 메모리 누수 방지 패턴

```Typescript
@Component({
    selector: ""
})
export class DataDisplayComponent implements OnInit, OnDestroy {

    private destroy$ = new Subject<>();

    data: any;

    constructor(private dataS: DataService){}

    ngOnInit(){

        // 1. takeUntil을 사용해서 관리
        this.dataS.getData().pipe(
            takeUntil(this.destroy$)
        ).subscribe(data => this.data = data);

        // 여러 개의 구독이 있어도 하나의 destroy$로 관리
        this.dataS.getOtherData().pipe(
            takeUntil(this.destroy$)
        ).subscribe(data => console.log(data));
    }

    ngOnDestory(){
        this.destroy$.next();
        this.destroy$.complete();
    }
}
```

#### 4. Subject를 활용한 Component 간 통신

```Typescript
// Service
@Injectable({
    providedIn: 'root'
})
export class StateService{
    private userSubject = new BehaviorSubject<User>();

    user$: Observable<User> = this.userSubject.asObservable();

    setUser(user: User){
        this.userSubject.next(user);
    }

    getUser(): User{
        return this.userSubject.value;
    }
}

// Component
export class HeaderComponent implements OnInit {
    user$: Observable<User>;

    constructor(private stateS: StateService){}

    ngOnInit(){
        this.user$ = this.stateS.user$;
    }
}
```

#### 5. 캐싱 패턴

```Typescript
@Injectable({
    providedIn: 'root'
})
export class DataService {
    private cache$: Observable<any[]>;

    constructor(private http: HttpClient){}

    getData(): Observable<any[]> {
        if(!this.cache$){
            this.cache$ = this.http.get<any[]>('/api/data').pipe(
                shareReplay(1) // 결과를 Cache하고 새 구독자와 공유
            )
        }
    }

    clearCache(){
        this.cache$ = null;
    }
}

```
