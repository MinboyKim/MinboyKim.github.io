---
title: "createConnection VS createPoo"
comments: true
sidebar_main: true
categories:
  - Diary
tags:
  - db
  - project
  - mysql
  - js
  - server
  - back-end
toc: true
toc_sticky: true
---

# 문제점

프로젝트 진행중에, DB와 연동한 후에 웹 퍼포먼스가 비정상적으로 느려진 모습을 확인했다. 그 이유를 찾고자 여러가지 방법을 테스트해보았는데, 가장 효과가 좋았던 방법을 기록하고자 한다.

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/1.png)

먼저 수정 전의 코드이다. createConnection을 이용해 DB와의 커넥션 객체를 만들어 이를 활용해 쿼리를 보내고 결과를 받아왔었다.

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/2.png)

export된 커넥션 객체를 import하여 쿼리를 보내는 모습이다. 이런 방식은 쿼리를 적게 사용하고 사용자가 적은 서버에서는 문제가 없지만, 여러 쿼리가 동시에 실행되고 사용자가 한 명이 아닌 환경에서는 적절하지 않다고 한다.

현재 상황은 createConnection으로 생성된 단 하나의 커넥션을 종료하지 않은채로 계속해서 돌려쓰고 있는 것이다. 이때 만약 여러개의 쿼리가 동시에 실행된다면, 커넥션이 하나밖에 존재하지 않기 때문에 한 쿼리가 끝나기 전까지 나머지 쿼리들은 기다릴 수 밖에없다. 따라서 웹 퍼포먼스가 안좋아진 것이다. 또한 모든 쿼리가 끝나도 커넥션을 close하는 부분이 존재하지 않아 계속해서 리소스를 잡아먹고 있었던 문제도 존재했다.

# 해결책

위와 같은 문제를 해결하기 위해서는 createConnection대신, createPool을 이용해야한다. **간단하게 말해서, createConnection이 하나의 커넥션 객체를 만들어낸다면 createPool은 여러 커넥션을 담을 수 있는 pool을 반환해준다.** 쿼리를 보낼때는 pool로부터 커넥션을 받아와 쿼리를 보내고 다시 pool로 커넥션을 반환하면 된다.

이 pool방식의 장점은 여러 커넥션 객체가 존재하므로 쿼리 수행을 병렬적으로 수행할 수 있고, 커넥션을 close하는 대신 open상태로 계속 두고 이 커넥션을 재활용함으로써 DB연결 시간의 오버헤드를 줄일 수 있다는 것이다. 이는 새로운 커넥션을 만드는 것보다 많은 시간을 줄일 수 있다고 한다.

주의할 점은, createPool이라고 미리 커넥션을 만들어 놓지는 않는다고 한다. 쿼리를 수행하기 위해 pool로 부터 커넥션을 받아오려고 하는데, pool에 남는 커넥션이 없다면 그때 pool에 커넥션이 생기는 것이다.(on demand)

또한 createPool을 할 때 option으로 connectionLimit을 설정하는데, pool의 커넥션 개수가 이 limit에 도달하면 더이상 커넥션을 생성하지 않고, 이 상황에서 커넥션을 받아가려는 시도가 발생한다면 대기가 필요하다. limit는 너무 많으면 많은 리소스를 차지하고, 너무 적으면 대기 시간이 길어지므로 서비스의 트래픽을 잘 파악하여 설정해주어야한다.

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/3.png)

위와 같이 pool을 생성하여 export 해주었다.

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/4.png)

쿼리를 보내는 부분에서는 pool을 import하여 pool로부터 커넥션을 받아오고 쿼리를 보낸 후 커넥션을 release해주는 것으로 pool에 커넥션을 반환해주었다.

쿼리를 보내는 부분은 자주 사용할 것 같아

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/5.png)

이와 같이 wrapper함수를 만들어 사용하기로 했다.

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/6.png)

쿼리를 보내는 부분이 한층 더 간결해진 모습이다.

# 비교

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/7.png)

먼저 createPool을 사용할 때의 모습이다. 로드 되는데 20초가 넘게 걸린 모습이다.

![스크린샷 2023-09-18 오후 7.57.23.png](/images/2023-9/DBConnection/8.png)

createPool을 적용한 모습이다. 로드가 훨씬 더 빨라진 모습을 확인할 수 있다.

# 정리

createConnection은 하나의 커넥션을 만들어 쿼리를 수행한다. 이는 다른 쿼리를 병렬적으로 실행할 수 없어 시간이 지연되거나, 다른 커넥션을 만들어야하는 부담 등의 문제를 만들 수 있다.

createPool은 여러 커넥션 객체를 만들어 재사용이 가능하다. 따라서 createConnection의 비효율을 해결할 수 있으나, 연결 개수를 잘 설정해주어야한다.

**참고내용**

[npm: mysql2](https://www.npmjs.com/package/mysql2)

[Node.js / MySQL - createConnection vs createPool](https://dirask.com/posts/Node-js-MySQL-createConnection-vs-createPool-DKg7nD)

[What is the difference between mysql.createConnection and mysql.createPool in Node.js MySQL module?](https://stackoverflow.com/questions/26432178/what-is-the-difference-between-mysql-createconnection-and-mysql-createpool-in-no)
