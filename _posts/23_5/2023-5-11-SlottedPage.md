---
title : "File Organization, Fixed-Length Records & Variable-Length Records(Slotted Page)"
comments : true
sidebar_main: true
categories:
  - Database
tags:
  - database
  - db
toc: true
toc_sticky: true
---

데이터 베이스는 file의 집합체로 저장된다. 각각의 파일은 일정한 크기의 disk block(page)로 나뉘어져있다. block은 records를 포함하고 있고 records는 entity와 속성으로 정의된다.

## Fixed-Length Records

![image](/images/2023-5/SlottedPage/record.png)

위와 같은 고정 크기 레코드의 크기는 5 + 20 + 20 + 8 = 53과 같이 쉽게 구할 수 있다. 길이가 고정되어있는 레코드를 이용하면 가장 간단하게 데이터베이스를 구현할 수 있을 것이다. 

![image](/images/2023-5/SlottedPage/fixed.jpeg)

i 번째 레코드의 위치를 알고 싶다면, 단순히 byte 계산을 해주면 된다. 

`i번째 레코드의 위치는 N * ( i - 1 ) byte`

예를 들어 위의 그림에서 레코드의 크기가 100byte라고 가정하면, 8번 레코드의 위치는 100 * ( 8 - 1 ) = 700byte이다.

고정 길이 레코드는 구현하기 쉽다는 장점이 있지만, 한가지 단점이 존재한다. 레코드를 삭제하고 싶은 경우를 생각해보자. 3번 레코드를 삭제하고 싶다고 하면, 4번부터 마지막 레코드를 한칸씩 앞으로 옮겨주어야 할 것이다. 

![image](/images/2023-5/SlottedPage/delete.jpeg)

(3번 레코드 삭제후 뒤의 레코드들을 한칸 땡겨준 모습)

최악의 경우에는 delete 한번에 N개의 레코드를 모두 옮겨주어야 하는 상황이 발생할 수도 있다.

이 단점을 보완하기 위해, 먼저 빈 공간을 마지막 레코드로 채워주는 방법을 생각해볼 수 있다.

![image](/images/2023-5/SlottedPage/delete2.jpeg)

(마지막 레코드인 11번 레코드를 3번 레코드를 삭제하고 생긴 빈공간으로 옮긴 모습)

또 한가지 방법으로는 free list를 이용하는 것이다. free list는 빈 공간의 정보를 담고 있으며, linked list 형식으로 이루어져 있다.

![image](/images/2023-5/SlottedPage/delete3.jpeg)

옆의 화살표가 linked list 형식으로 모든 빈공간의 정보를 담고 있는 free list의 모습이다. 이렇게 모든 빈공간의 정보를 갖고 있다가 새로운 레코드가 들어오면 빈공간에 할당하고, 레코드를 삭제해야할 일이 생기면 빈공간의 정보를 free list에 추가하는 것이다.

## Variable-Length Records

모든 레코드가 고정 크기 레코드라면 좋겠지만, 현실의 데이터들은 가변 길이 정보를 갖고 있을 수도 있다. (ex varchar) 그럴 때는 가변 길이 레코드를 이용해 데이터를 관리해주어야한다. 

![image](/images/2023-5/SlottedPage/varlength.jpeg)

Variable-Length record는 위와 같은 구조로 구성되어있다.

- 헤더 : 가변 속성의 시작 주소 (offset), 길이 (length)
- Null bitmap : 몇번째 속성이 null value인지 나타냄 (ex 0110 - 첫번째, 두번째 속성이 null)

## Slotted Page Structure

Variable-Length Record를 저장하는 방법의 한 예로 Slotted Page Structure가 있다. 

![image](/images/2023-5/SlottedPage/slottedpage.jpeg)

Slotted Page Structure의 구조를 살펴보자. Variable-Length Records의 구조와 비슷하다.

Slotted Page Structure의 헤더는 다음의 정보를 포함한다.

- Record의 개수
- Free space의 끝부분을 가리키는 포인터
- 각 record의 위치와 크기

예시를 살펴보자.

![image](/images/2023-5/SlottedPage/slottedpage2.jpeg)

비어있는 slotted page의 모습이다. 이 slotted page에 크기가 8인 레코드를 하나 추가하면

![image](/images/2023-5/SlottedPage/slottedpage3.jpeg)

위와같은 모습이 된다. 

**레코드 추가시**

- 헤더의 레코드 개수 카운트가 증가
- free space pointer가 옮겨짐
- 레코드의 정보(크기, 위치)가 free space의 앞에 추가
- free space의 뒤에 실제 레코드 추가

![image](/images/2023-5/SlottedPage/slottedpage4.jpeg)

하나더 추가하면 위와 같은 모습이 된다.

이때 레코드 1번을 삭제하는 경우를 생각해보자.

![image](/images/2023-5/SlottedPage/slottedpage5.jpeg)

레코드가 삭제되면 헤더의 레코드 정보 부분에 -1을 기록한다. 이후에 레코드가 추가될 때 만약 삭제된 빈 공간보다 크기가 같거나 작은 레코드가 들어오면 -1로 기록된 빈 공간에 할당하고, 크기가 큰 레코드가 들어오면 원래 레코드가 추가될 때처럼 Free space에 할당한다. 

![image](/images/2023-5/SlottedPage/slottedpage6.jpeg)

만약 빈공간의 크기보다 작은 레코드가 들어오면 남은 빈공간은 낭비된다. 위 그림에서 크기가 5인 레코드가 추가된다고 생각해보자.

빈공간이 8이었던 곳에 크기가 5인 레코드를 저장해 3만큼의 공간이 낭비되는 것을 확인할 수 있다.