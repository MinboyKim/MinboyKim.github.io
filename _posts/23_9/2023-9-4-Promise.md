---
title: "프로미스(Promise) 패턴"
comments: true
sidebar_main: true
categories:
  - JS
tags:
  - promise
  - asynchronous
  - js
toc: true
toc_sticky: true
date: "2023-09-04 22:16:00 +0900"
---

# Promise

Promise 객체는 비동기 연산과 결과의 완료 또는 실패를 나타낸다. Promise 객체에 대해 자세하게 알아보고 사용법을 익혀보자.

# 왜 필요할까?

게시판 웹 서비스를 만들고 있다고 생각해보자. 서버에서 데이터를 받아와 그 데이터들을 페이지에 출력하려고 한다. 이를 위해 서버에 데이터를 요청하고 받아와 리턴하는 함수를 만들어 볼 수 있다.

```jsx
function getPosts() {
  const posts = callAPI();
  return posts;
}
```

위와 같은 함수는 우리가 예상한대로 동작하지 않을 것이다. 왜냐하면 JS는 네트워크 요청과 같은 블로킹 코드를 비동기적으로 처리하기 때문이다. 따라서 posts에 callAPI함수의 결과가 도달하기 전에 getPosts함수는 리턴되어버리고 우리가 원하는 데이터를 받아볼 수 없게 될 것이다. JS의 비동기 작업에 관한 더 자세한 이야기는 다음 글을 참고하자. [JavaScript의 비동기 작업](https://minboykim.github.io/js/JSAsync/)

이러한 문제를 해결하기 위해서 Promise 객체를 이용할 수 있다.

# 사용법

먼저 Promise 객체의 사용법부터 살펴보도록하자. 위 예시를 Promise 객체를 활용해 수정하면 다음과 같다.

```jsx
function getPosts() {
	return new Promise((resolve, reject) => {
		// 여기서 API 요청
		// if success
		resolve(posts);
		// if fail
		reject(new Error("err...."));
	});
}

getPosts()
	.then((posts) => {
		// 데이터를 페이지에 출력 (성공 시)
	})
	.catch((err) => {
		// 에러처리 코드 (실패 시)
	})
	.finally(() => {
		// 성공, 실패 상관없이 실행
	}
```

Promise 객체는 new Promise 키워드로 만들 수 있다. 이때, 콜백함수를 하나 인자로 받는데, 이 콜백함수에 비동기적인 일과 우리가 원하는 코드들을 작성해주면 된다. 이 콜백함수는 **resolve와 reject 두 개의 콜백함수**를 인자로 받는다. r**esolve는 작업이 성공 시에, reject는 작업이 실패시**에 호출해주면 된다.

이때 호출된 함수가 resolve냐 reject에 따라 이후의 작업이 결정된다. getPosts함수는 Promise 객체를 반환함으로써 반환된 Promise 객체를 통해 여러 메소드들을 추가적으로 `체이닝`할 수 있다.

**then()** 메소드에는 작업이 성공하여 resolve 함수가 호출될 시에 순차적으로 실행될 작업들을 담은 콜백함수를 전달한다. 콜백함수의 인자로는 resolve함수에 전달된 파라미터가 전달된다.

**catch()** 메소드에는 작업이 실패하여 reject 함수가 호출될 시에 순차적으로 실행될 작업들을 담은 콜백함수를 전달한다. 콜백함수의 인자로는 reject 함수에 전달된 파라미터가 전달된다.

**finally()** 메소드에는 작업이 성공하든, 실패하든 상관없이 실행될 작업들을 담은 콜백함수를 전달한다.

# 좀 더 자세히..

Promise 객체는 `pending`, `fulfilled`, `rejected` 3가지 state를 가진다.

- pending : 초기 상태, fulfiled와 rejected가 아닌 상태이다.
- fulfilled : 작업이 성공적으로 끝난 상태, resolve가 호출된 상태이다.
- rejected : 작업이 실패한 상태, reject가 호출된 상태이다.

![img](/images/2023-9/Promise/PromiseState.png)

then과 catch도 Promise 객체를 반환한다. 따라서 아래와 같이 여러개의 then을 체이닝 할 수도 있다.

```jsx
myPromise
  .then((value) => `${value} and bar`)
  .then((value) => `${value} and bar again`)
  .then((value) => `${value} and again`)
  .then((value) => `${value} and again`)
  .then((value) => {
    console.log(value);
  })
  .catch((err) => {
    console.error(err);
  });
```

중간의 then에서 오류가 발생(reject)한다면 아래의 then들은 모두 건너뛰고 마지막의 catch로 이동된다.

# Promise concurrency

Promise는 여러 비동기 작업을 동시에 실행할 수 있는 메소드들을 지원한다. 인자로 원하는 여러 Promise 객체들을 담은 배열을 전달해주면 된다.

- Promise.all() : 모든 Promise들이 fulfilled 상태가 되면 fulfilled 상태의 Promise 객체를 반환한다. 하나의 Promise라도 reject 된다면 rejected 상태의 Promise 객체를 반환한다.
- Promise.any() : 하나의 Promise라도 fulfilled 상태가 되면 fulfilled 상태의 Promise 객체를 반환한다.모든 Promise들이 reject 되면 rejected 상태의 Promise 객체를 반환한다.
- Promise.race() : 하나의 Promise가 settled되는 즉시 그 Promise의 state에 따른 Promise 객체를 반환한다.

참고 내용 및 사진 출처
![MDN - Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
