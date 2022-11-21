---
layout : posts
title : "C++ 조합(Combination) - next_permutation"
typora-copy-images-to : ../../images/2022-11-21
comments : true
---

[백준 23057번 - 도전 숫자왕](https://www.acmicpc.net/problem/23057) 을 풀다가 알게된 사실. C++ 에서 순열(Permutation)을 구할 때 사용하는 next_permutaion 을 이용하면 조합(Combination)을 쉽게 구할 수 있다.

> 순열(Permutation) : 서로 다른 n개에서 r개를 뽑아서 **정렬**하는 경우의 수
>
> 조합(Combination) : 서로 다른 n개에서 **순서없이** r개를 뽑는 경우의수



## 헤더파일 

``` #include <algorithm>```

## 사용법

* 순열(Permutation)

```c++
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

int main(void) {
  int myints[] = {1,2,3};
  cout << "The 3! possible permutation with 3 elements \n";
  do{
    cout << myints[0] << myints[1] << myints[2] << "\n";
  } while(next_permutation(myints, myints+3));
  cout << "After loop : " << myints[0] << myints[1] << myints[2] << "\n";
  return 0;
}
```

1, 2, 3 이 존재하는 배열을 이용해 만들수 있는 모든 순열을 출력하는 프로그램이다. do-while 문을 통해 이용하고 next_permutation의 파라미터로 순열을 생성할 배열의 시작주소와 끝 주소를 전달한다. 

* 실행 결과

  ![스크린샷 2022-11-21 오후 4.35.01](../images/2022-11-21/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202022-11-21%20%EC%98%A4%ED%9B%84%204.35.01.png)

​	1, 2, 3 을 이용한 모든 순열을 출력한다. 루프가 끝난 뒤에는 원래의 순서를 유지하는 모습을 볼 수 있다.

---

* 조합(Combination)

``` c++
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

int main(void) {
  int myints[] = {1, 2, 3};
  int len = sizeof(myints) / sizeof(int);

  for (int i = 1; i <= len; i++) {
    vector<bool> v(len - i, false);
    v.insert(v.end(), i, true);
    cout << "---------------- " << i << "개 -----------------\n";
    do {
      for (int j = 0; j < len; j++) {
        if (v[j]) cout << myints[j] << " ";
      }
      cout << "\n";
    } while (next_permutation(v.begin(), v.end()));
  }

  return 0;
}
```

3개의 원소로 만들 수 있는 모든 조합을 보여주는 프로그램이다. 이 때 *vector v* 는 배열(myints)와 같은 크기를 가지며, bool 값을 통해 해당 인덱스의 원소가 조합에 뽑힐지 뽑히지 않을지 판단할 수 있게 해준다.

1. false false **true**  => 3개 중 1개 선택하는 경우
2. false **true** **true** => 3개 중 2개 선택하는 경우
3. **true** **true** **true** => 3개 중 3 선택하는 경우

next_permutation 을 통해 순서가 바뀌며 조합을 생성한다.

* 실행 결과

  ![스크린샷 2022-11-21 오후 4.55.39](../images/2022-11-21/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202022-11-21%20%EC%98%A4%ED%9B%84%204.55.39.png)

  

---

라이브러리 함수를 통해 순열과 조합을 쉽게 구현할 수 있다. 배열 뿐만 아니라 벡터나 문자열에도 이용할 수 있을 것이다.
