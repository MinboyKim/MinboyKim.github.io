---
title: "V8 엔진이 코드를 실행하는 방법"
comments: true
sidebar_main: true
categories:
  - JS
tags:
  - V8
toc: true
toc_sticky: true
date: "2023-07-16 18:04:00 +0900"
---

V8은 크롬과 node.js에서 사용하는 고성능 오픈소스 js 웹어셈블리 엔진이다. 이 글에서 V8엔진의 아키텍쳐를 살펴보자.

# V8엔진 소개

![V8logo.png](/images/2023-7/V8Architecture/V8logo.png)

[사진 출처](https://ko.wikipedia.org/wiki/V8_%28%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8_%EC%97%94%EC%A7%84%29)

V8엔진은 구글이 주도하여 `C++` 로 작성된 엔진이다. 오픈소스이기에 [여기](https://github.com/v8/v8)에서 소스코드를 살펴볼 수도 있다. V8엔진은 원래 자동차에 사용되는 8기통 엔진을 의미하는 단어이다.

그래서인지 `Igniton(점화기)`, `TurboFan(냉각팬)` 등의 용어들도 실제 자동차 엔진의 부품에서 이름을 따온것으로 보인다. 뒤에서 더 살펴보자.

V8엔진이 js 코드를 processing 하는 과정은 세 단계를 거친다.

1. 코드를 parse하기
2. 코드를 compile하기
3. 코드를 execute하기

하나씩 살펴보도록 하자.

# Parsing 단계

Parsing단계에서, V8엔진은 코드를 각각의 `token` 으로 분리한다. 코드를 token으로 분리해내고 나면, 토큰들은 syntax parser로 넘겨져 AST(Abstract Syntax Tree), 추상 구문 트리로 변환한다.

```jsx
const sum = 5 + 7;
```

예를 들어 위의 코드를 AST로 변환한 결과는 다음과 같다.

![스크린샷 2023-07-16 오후 4.14.01.png](/images/2023-7/V8Architecture/AST.png)

[사진 출처](https://www.geeksforgeeks.org/how-v8-compiles-javascript-code/)

# Compilation 단계

![v8-ignition.svg](/images/2023-7/V8Architecture/v8-ignition.svg)

Igniton 로고, [사진 출처](https://v8.dev/blog/launching-ignition-and-turbofan)

![v8-turbofan.svg](/images/2023-7/V8Architecture/v8-turbofan.svg)

TurboFan 로고, [사진 출처](https://v8.dev/blog/launching-ignition-and-turbofan)

컴파일 단계는 human-readable 코드를 machine 코드로 변환하는 단계이다. 다른 언어들과 다르게, V8엔진은 컴파일러와 인터프리터를 둘 다 사용한다.

- 컴파일 언어 - 코드 실행 전 컴파일러가 모든 코드를 기계어로 번역해 실행 (C, C++…)
- 인터프리터 언어 - 인터프리터가 코드를 한줄 한줄 바이트 코드로 변환하여 실행 (python..)

V8의 인터프리터 `Igniton`은 AST를 입력으로 받아 바이트 코드를 출력으로 내놓는다. V8의 컴파일러인 `TurboFan` 은 인터프리터의 바이트코드를 입력으로 받아 최적화된 기계어를 출력으로 내놓는다.

파서(Parser)에서 만들어진 AST를 `Ignition`이 바이트 코드로 변환하면, 이 바이트 코드를 실행함으로써 우리의 소스 코드는 작동하게 되는 것이다. V8엔진은 이 중에 자주 사용되는 함수나 변수들의 패턴을 찾아 `TurboFan`으로 보낸다.

- TurboFan의 최적화 기법에는 대표적으로 Hidden class, Inline caching 등의 기법이 있다. 이러한 기법들은 차후에 알아보겠다.
- 만약 코드가 덜 자주 불려지는 등 성능이 저하되거나 함수에 전달된 매개변수가 타입을 변경하는 등의 변화가 생기면 V8은 컴파일 된 코드를 디컴파일해 다시 인터프리터로 보낸다.

Ignition은 엔진에 시동걸때 사용하는 점화기를 의미한다. 이 점화기를 통해 우리의 js 코드가 실행되기 시작하는 것이다. 이 중에 자주 호출되어 열을 많이 받는 부분은 TurboFan, 냉각팬으로 보내져 최적화되는 것이다. 네이밍 센스가 재미있다.

# Execution 단계

컴파일 단계에서 만들어진 바이트 코드는 V8엔진의 런타임 환경의 Memory Heap과 Call Stack에 의해 실행된다.

- Memory Heap

메모리가 할당된 모든 변수와 함수가 존재하는 곳이다.

- Call Stack

각각의 함수가 호출될 때 스택에 push되고, 함수가 실행이 끝나면 스택에서 pop되는 공간이다.

인터프리터가 코드를 해석할 때, key는 바이트 코드이고 value로는 해당 바이트 코드를 처리하는 함수를 갖는 객체 구조를 사용한다. V8엔진은 이러한 값들을 메모리에 리스트 형태로 정렬하여 저장하며, 이를 맵(Map)에 저장하여 많은 메모리를 절약한다.

V8엔진의 메모리 구조와 GC(Garbage Collection)에 대해서는 다음 글에서 살펴보겠다.

참고 내용

[How V8 compiles JavaScript code ? - GeeksforGeeks](https://www.geeksforgeeks.org/how-v8-compiles-javascript-code/)

[V8 엔진은 어떻게 내 코드를 실행하는 걸까?](https://evan-moon.github.io/2019/06/28/v8-analysis/)

[구글 V8 엔진 살펴보기](https://jaehyeon48.github.io/javascript/google-v8-engine/)
