---
layout: post
title: TypeScript | Generics
description: 타입스크립트 제네릭 개념과 사용 예시
summary: 타입스크립트 제네릭 개념과 사용 예시
tags: typescript
---

## 제네릭이란?

단일 타입이 아닌 다양한 타입에서 작동하는 컴포넌트를 생성하는데 사용된다. 제네릭을 통해 여러 타입의 컴포넌트나 자신만의 타입을 사용할 수 있다.

<br>

## 제네릭을 사용하는 이유

```tsx
function insertAtBeginning(array: any[], value: any) {
  const newArray = [value, ...array]
  return newArray
}

const demoArray = [1, 2, 3]
const updatedArray = insertAtBeginning(demoArray, -1) // any[]

// any로 타입을 지정해주었기에 아래 문자열 메소드에서 에러가 나지 않는다
updatedArray[0].split('')
```

위의 예제에서 보면 `updatedArray`의 추론된 타입은 `any[]` 이다. 함수 선언시 타입을 any로 지정했기 때문에 타입스크립트가 배열에 `number` 타입만 들어있다는 것을 인식하지 못한다. 그 결과 문자열 메소드인 `split`에서 에러가 발생하지 않는다. 함수 호출한 다음 타입스크립트로부터 어떤 지원도 받을 수 없다.

```tsx
function insertAtBeginning(array: number[], value: number) {
  const newArray = [value, ...array]
  return newArray
}

const demoArray = [1, 2, 3]
const updatedArray = insertAtBeginning(demoArray, -1) // number[]
```

위와 같이 선언 시 타입을 지정해 줄 수 있지만, 지정 타입 이외의 다른 타입에서는 사용이 불가능하다. 유틸리티 함수와 같이 여러 타입이 들어 올 가능성이 있다면 그 함수는 재사용하기 어렵다.

<br>

_타입의 안정성과 유연성을 주기 위해 제네릭을 사용한다.!_

<br>

## 제네릭 사용 방법

```tsx
function insertAtBeginning<T>(array: T[], value: T) {
  const newArray = [value, ...array]
  return newArray
}

const demoArray = [1, 2, 3]
const updatedArray = insertAtBeginning(demoArray, -1) // number[]
// 또는 아래와 같이 명시할 수 있다.
const updatedArray = insertAtBeginning<number>(demoArray, -1) // number[]

// number 배열이라는 것을 인식해 에러 발생
updatedArray[0].split('')

const stringArray = insertAtBeginning(['a', 'b'], 'c') // string[]
```

함수 안에서만 사용할 수 있는 제네릭 타입 정의할 수 있다. 보통 Type의 `T`를 사용 타입 변수를 추가한다. 정의하면 타입을 함수와 매개변수 목록에서 사용할 수 있다. 함수를 호출하면 이 변수는 유저가 준 타입을 캡쳐하고 이 값을 이제 타입스크립트는 알 수 있다.

컴파일러는 `number[]`로 추론해 Type의 값을 자동으로 정하게 된다. 제네릭 타입을 사용해 타입스크립트에게 `any` 타입이 아니라는 것을 알려준다. 어떤 타입이든 사용할 수 있지만, 특정 타입을 사용해 함수를 실행하면 해당 타입으로 고정되어 동작하게 한다.

<br>

클래스나 인터페이스에서도 다양한 타입으로 재사용할 수 있다.

```tsx
// interface
interface Demo<T> {
  name: string
  options: T
}

const demo1: Demo<string> = {
  name: 'demo1',
  options: 'red',
}

const demo2: Demo<{ color: string; age: number }> = {
  name: 'demo2',
  options: {
    color: 'blue',
    age: 10,
  },
}
```

<br>

## 제네릭 제약조건 (extends)

```tsx
function showName<T>(data: T): string {
  return data.name
}
```

위의 예제에서 data의 `.name` 프로퍼티에 접근하려 했지만, 모든 타입에서 `.name`을 가지고 있는지 알 수 없다. `.name` 프로퍼티가 있는 모든 타입들에서 작동하는 것으로 제한하고 싶을 때 아래와 같이 Type에 제약조건을 줄 수 있다.

```tsx
function showName<T extends { name: string }>(data: T): string {
  return data.name
}

// 혹은 제약조건을 인터페이스로 정의

interface NameIn {
  name: string
}

function showName<T extends NameIn>(data: T): string {
  return data.name
}
```

제네릭 함수는 이제 제한되어있어 필요한 프로퍼티들이 있는 타입의 값을 전달해야한다.

```tsx
interface User {
  name: string
  age: number
}

interface Car {
  name: boolean
}

interface Book {
  price: number
}

const user: User = { name: 'abc', age: 10 }
const car: Car = { name: false }
const book: Book = { price: 1000 }

function showName<T extends { name: string }>(data: T): string {
  return data.name
}

showName(user)
showName(car) // car의 name 타입이 boolean이기 때문에 타입 에러 발생
showName(book) // name이 없기 때문에 에러 발생
```

다른 타입의 매개변수로 제한된 타입 매개변수를 선언할 수 있다.

```tsx
// 공식문서 예제
// obj에 존재하지 않는 프로퍼티를 가져오지 않도록 하기 위해 제약조건을 걸었다.
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key]
}

let x = { a: 1, b: 2, c: 3, d: 4 }

getProperty(x, 'a')
getProperty(x, 'm')
```

<br>

## Axios.get()

axios를 이용하면서 get의 타입을 보고 제네릭에 대한 개념을 명확하게 잡고 싶어 정리했다.

아래와 같이 타입 변수에 response에 대한 타입을 설정해주고 있다.

```tsx
export interface DivisionMatchParams {
  accessId: string
  startMatchCount: number
  nickname: string
}

export const getDivisionMatchApi = (params: DivisionMatchParams) =>
  axios.get<DivisionMatchResponse>(`${BASE_URL}matches`, {
    params: {
      ...params,
    },
  })
```

<br>

[https://github.com/axios/axios/blob/v1.x/index.d.ts](https://github.com/axios/axios/blob/v1.x/index.d.ts)

```tsx
Axios.get<T = any, R = AxiosResponse<T, any>, D = any>(url: string, config?: AxiosRequestConfig<D> | undefined): Promise<R>

export interface AxiosResponse<T = any, D = any>  {
  data: T;
  status: number;
  statusText: string;
  headers: RawAxiosResponseHeaders | AxiosResponseHeaders;
  config: AxiosRequestConfig<D>;
  request?: any;
}
```

T에 DivisionMatchResponse 타입을 주면서 response의 data 타입을 설정한다.

```tsx
export interface AxiosRequestConfig<D = any> {
  url?: string
  method?: Method | string
  baseURL?: string
  transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[]
  transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[]
  headers?: RawAxiosRequestHeaders
  params?: any
  paramsSerializer?: ParamsSerializerOptions
  data?: D
  timeout?: number
  timeoutErrorMessage?: string
  withCredentials?: boolean
  adapter?: AxiosAdapter
  auth?: AxiosBasicCredentials
  responseType?: ResponseType
  responseEncoding?: responseEncoding | string
  xsrfCookieName?: string
  xsrfHeaderName?: string
  onUploadProgress?: (progressEvent: AxiosProgressEvent) => void
  onDownloadProgress?: (progressEvent: AxiosProgressEvent) => void
  maxContentLength?: number
  validateStatus?: ((status: number) => boolean) | null
  maxBodyLength?: number
  maxRedirects?: number
  maxRate?: number | [MaxUploadRate, MaxDownloadRate]
  beforeRedirect?: (
    options: Record<string, any>,
    responseDetails: { headers: Record<string, string> }
  ) => void
  socketPath?: string | null
  httpAgent?: any
  httpsAgent?: any
  proxy?: AxiosProxyConfig | false
  cancelToken?: CancelToken
  decompress?: boolean
  transitional?: TransitionalOptions
  signal?: GenericAbortSignal
  insecureHTTPParser?: boolean
  env?: {
    FormData?: new (...args: any[]) => object
  }
  formSerializer?: FormSerializerOptions
}
```

config 타입 정의도 보면 params의 타입은 any로 지정된 것을 볼 수 있다.

<br>

## Reference

[https://www.typescriptlang.org/docs/handbook/2/generics.html](https://www.typescriptlang.org/docs/handbook/2/generics.html)

[https://www.udemy.com/course/best-react/](https://www.udemy.com/course/best-react/)

[https://www.youtube.com/watch?v=pReXmUBjU3E](https://www.youtube.com/watch?v=pReXmUBjU3E)
