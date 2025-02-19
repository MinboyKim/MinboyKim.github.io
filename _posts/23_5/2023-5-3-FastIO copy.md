---
title : "Fast I/O 정리"
comments : true
sidebar_main: true
categories:
  - Cpp
tags:
  - Cpp
toc: true
toc_sticky: true
---

PS를 하다보면 반드시 마주치는 것이 있다. 바로 Fast I/O이다. 

이 글에서는 맨날 써오던 ios::sync_with_stdio(false)와 cin.tie(nullptr)를 왜 사용해야 하고 어떻게 쓰는지 설명하겠다.

## ios::sync_with_stdio(false)

이 코드는 C와 C++의 표준 stream의 동기화를 비활성화 시켜준다. 원래 C++은 동기화가 디폴트로 되어있는데, 이때는 cin, cout (C++ 스타일), scanf, printf (C 스타일)를 혼용해도 문제가없지만, 이 코드를 작성하면 동기화가 비활성화 되면서 입출력 시간이 절약되는 것이다.

## cin.tie(nullptr)

이 코드를 사용하면 cin과 cout을 묶어주는 과정을 수행하지 않도록 할 수 있다. 평소에는 cin과 cout이 하나로 묶여있어 

```cpp
cout << "input : ";
cin >> name;
```

과 같은 코드가 있다고 할 때 반드시 “input : “ 문자열이 먼저 출력되고 name을 입력받는다. 하지만 cin.tie(nullptr)를 작성해주면 문자열 출력 전에 입력을 받을 수 있는 것이다. PS를 할 때는 입출력 순서가 크게 상관이 없기 때문에 위 코드를 작성하여 시간을 절약하는 것이다.

<details>
<summary>buffer란?</summary>
<div markdown="1">       

프로그램은 자체적으로 입출력을 할 수 있는 기능이 존재하지 않는다. 따라서 운영체제(OS)에게 입출력을 요청하는데, 이렇게 OS에게 무언가를 해달라고 요청하는 것을 system call이라고 한다. 하지만 system call은 다른 일반적인 함수들에 비해 많은 시간을 요구하는 단점이 존재한다. 
    
만약 ‘a’ ‘b’ ‘c’ ‘d’ ‘e’를 출력한다고 가정해보자. 다섯글자를 화면에 출력하기 위해 system call이 다섯번 호출될 것이다. 이는 많은 시간을 잡아먹기 때문에, 내부적으로 buffer라는 공간에 다섯글자를 모두 담아두고, system call은 한번만 호출하게 되면 같은 결과를 보여주면서도 system call 호출 횟수를 최소화 할 수 있을 것이다. (입력도 마찬가지이다) Buffer는 이렇게 입출력을 저장할 수 있는 공간이고, 이를 통해 system call의 호출 횟수를 최소화 할 수 있게해주는 역할을 수행한다.

</div>
</details>

<details>
<summary>자세한 설명</summary>
<div markdown="1">       

iostream은 cstdio보다 다양한 입출력을 지원하기 위해 자체적인 buffer를 이용하는데, 이 buffer를 거쳐 출력할 때 printf 등이 이용하는 cstdio의 buffer를 거쳐서 출력하게 된다. 

따라서 만약 iostream 계열의 함수 (cin, cout)를 cstdio의 함수(printf, scanf)와 섞어쓰면 함수 호출 순서와 무관하게 입출력이 섞이게 되는데, 이를 방지하기 위해 system call들을 사용하게 된다. system call들은 무지막지하게 느리므로 iostream계열 함수들이 cstdio 계열 함수들보다 입출력 속도가 떨어지는 것이다. 

이것을 해결하기 위해 cstdio의 buffer 눈치를 보지말고 입출력을 하도록 하는 것이ios::sync_with_stdio(false)의 의미이다.

---

cout과 cin을 혼용하여 사용할 때, 입력을 기다리는 동안 buffer에 출력이 저장되어 있으면 프로그램이 멈춘것처럼 보인다. 이를 방지하기 위해 iostream은 cout으로 buffer에 저장된 값을 cin을 할 때 flush (buffer를 비우는 것)해준다. flush는 system call을 유발하고 이또한 많은 시간을 소요한다. cin.tie(nullptr)은 이를 신경쓰지 말고 입출력을 알아서 하도록 하는 코드인 것이다.

---

결론 적으로 두 코드를 이용하면 iostream의 입출력을 본인들의 buffer를 통해 눈치보지않고, system call의 호출 수를 줄여 시간을 절약할 수 있도록 해준다.

</div>
</details>

## endl  VS  ‘\n’

사실 위의 **자세한 설명**을 읽었다면 더 설명이 필요하지 않은 부분인데, endl은 해당 줄을 끝내고 buffer를 비우는 행동(flush)을 취한다. 따라서 system call이 호출되고 많은 시간을 잡아먹는다. 반면에 ‘\n’은 그저 개행 문자 하나일뿐이므로 system call을 호출하지 않으므로 훨씬 적은 시간을 사용한다. 

## 사용 tip

```cpp
ios::sync_with_stdio(false);
cin.tie(nullptr);
```

위 두코드에서 false와 nullptr모두 0으로 대체할 수 있다. 따라서 다음과 같이 작성가능하다.

```cpp
ios::sync_with_stdio(0);
cin.tie(0);
```

더 짧게 한줄로도 작성할 수 있다. 다음과 같다.

```cpp
cin.tie(0)->sync_with_stdio(0);
```

이렇게 작성이 가능한 이유는, cin.tie(0)의 반환값이 ostream* (ostream 포인터)이기 때문이다. 

![image](/images/2023-5/FastIO/tie.png)

[cplusplus reference](https://cplusplus.com/reference/ios/ios/tie/)

(반환값에 ostream*)

또한 ostream 객체는 sync_with_stdio를 멤버함수로 갖고 있다. 

![image](/images/2023-5/FastIO/sync_with_stdio.png)

[cplusplus reference](https://cplusplus.com/reference/ostream/ostream/?kw=ostream)

따라서 위의 한줄로 작성된 코드는 cin.tie(0)을 수행하고, 리턴된 ostream 포인터를 통해 객체의 멤버함수 sync_with_stdio(0)을 수행하라는 의미로 해석할 수 있다. 

## 결론

PS를 할 때는 fast I/O를 사용하지않으면 시간초과를 받는 문제들이 많다. 따라서 fast I/O를 꼭 사용해야하는 경우들이 생기는데, 위 내용들을 이해했다면 알겠지만 일반적인 프로그램에서는 fast I/O를 쓴다고 꼭 좋은 것만은 아니다. 상황에 따라 목적에 맞게 사용해야하는 것이지 빠르다고 좋은것만은 아니라는 것이다. PS라는 입출력에 크게 제한이없고 시간제한이 중요한 특수한 상황이 fast I/O를 쓰도록 한것이다.

이 글로 fast I/O에 대한 의문들을 많이 해소하셨길 바랍니다.

### 세 줄 요약

1. 입출력 빠르게 하려고 **cin.tie(0)→sync_with_stdio(0)** 씀
2. endl 말고 **‘\n’** 써야함
3. fast I/O는 PS와 같은 특정상황에만 쓰는거지 **빠르다고 좋은게 아님**