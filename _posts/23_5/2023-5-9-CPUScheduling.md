---
title : "CPU Scheduing"
comments : true
sidebar_main: true
categories:
  - OS
tags:
  - 운영체제
  - CPU
toc: true
toc_sticky: true
---

CPU는 하나의 프로세스 작업을 마치고 나면, 다음 프로세스를 선택하여 작업을 수행한다. 이 때 다음 프로세스로 어떤 프로세스를 선택할지 결정하는 알고리즘을 CPU Scheduling Algorithm이라고 한다. 이 글에서는 여러 CPU Scheduling Algorithm을 살펴보고, 각각을 비교해보려고 한다.

먼저, 각각의 CPU Scheduling Algorithm을 비교하기 위한 척도로 두가지 속성을 정하자.

1. Turnaround time
    
    Turnaround time은 job이 도착한 시점에서부터 job이 완료되는 시점까지 걸린 시간이다. 다음과 같이 구할 수 있다.
    
    ![image](/images/2023-5/CPUScheduling/turnaround.jpeg)
    
2. Response time
    
    Response time은 job이 도착한 시점에서부터 job이 실행되는 시점까지 걸린 시간이다. 다음과 같이 구할 수 있다.
    
    ![image](/images/2023-5/CPUScheduling/response.jpeg)
    

Turnaround time과 Response time이 모두 짧을 수록 좋은 CPU Scheduling Algorithm이라는 것을 알 수 있다.

## 1. First In, First Out (FIFO) == First Come, First Served (FCFS)

먼저 가장 간단한 방법인 First In, First Out 방법이다. 먼저 도착한 job을 먼저 수행하는 방법으로 가장 간단하고 구현하기도 쉽다. 

![image](/images/2023-5/CPUScheduling/FCFS.jpeg)

(모든 job은 0초에 동시에 도착했다고 가정)

평균 Turnaround time은 20이다.

FIFO 알고리즘은 간단하지만 치명적인 단점이 존재한다. 오래걸리는 job이 앞에 위치하면 효율이 급격하게 떨어진다는 것이다. 다음의 예시를 보자.

![image](/images/2023-5/CPUScheduling/FCFS2.jpeg)

이렇게 되면 오래걸리는 job을 처리하는 동안 뒤에 있는 job들도 수행하지 못하기 때문에 평균 Turnaround time이 매우 나빠지게 된다.

## 2. Shortest Job First (SJF)

FIFO의 위와같은 단점을 해결하는 알고리즘으로, job들이 동시에 도착했을 때 짧은 job을 먼저 수행하는 알고리즘이다.

![image](/images/2023-5/CPUScheduling/SJF.jpeg)

같은 경우에 짧은 job인 파란색 job과 초록색 job을 먼저 수행하여 평균 Turnaround time이 훨씬 좋아진 것을 볼 수 있다.

하지만 SJF 알고리즘도 단점이 존재한다. 현실의 job들은 항상 동시에 도착하지 않기 때문에, 오래걸리는 job이 먼저 도착한다면 또다시 FIFO에서 보았던 단점을 드러낸다는 것이다. 

![image](/images/2023-5/CPUScheduling/SJF2.jpeg)

오래걸리는 job인 A가 먼저 도착하자 또다시 Turnaround time이 나빠지는 것을 볼 수 있다.

## Preemptive VS Non-Preemptive

위의 문제를 해결하기 위해, Preemptive(선점)이라는 개념이 필요하다. 

- Non-Preemptive Scheduler (비선점)
    - 지금까지 봐온 Scheduling 알고리즘들처럼 하나의 job이 완료되기 전까지는 다른 job을 수행하지 않는다.
    - 옛날의 batch computing에서는 일반적으로 매우 오래 실행되는 작업들이 많았고, context switch의 오버헤드가 높았었기 때문에 Non-Preemptive scheduler를 이용하는 경우가 많았다.
- Preemptive Scheduler (선점)
    - 다른 job을 수행하기 위해 현재 process를 중단한다.
    - 이렇게 현재 process를 다른 process로 바꾸는 작업을 context switch라고 한다.

## 3. Shortest Time-to Completion First (STCF) == Preemptive Shortest Job First (PSJF)

이름에서 볼 수 있듯이 SJF를 Preemptive하게 변형한 알고리즘이다. 현재 수행하고 있는 job보다 더 짧은 수행시간이 걸리는 job이 대기중이면 그 job으로 context switch해 job을 수행한다.

![image](/images/2023-5/CPUScheduling/PSJF.jpeg)

이런 방식을 이용하면 아까 SJF 알고리즘의 단점을 상쇄할 수 있다.

만능 같아 보이는 PSJF에도 한가지 약점은 존재한다. PSJF는 Turnaround time은 최적일 수 있지만, Response time은 최적이 아닐 수 있다. 위 경우에서 평균 Response time을 계산해보면 

Response time of A : 0 - 0 = 0

Response time of B : 10 - 10 = 0

Response time of C : 20 - 10 = 10

Average response time = (0 + 0 + 10) / 3 = 3.33 이다. 꽤나 좋아보이지만 Response time을 더 줄일 수 있는 방법이 있다.

## 4. Round-Robin (RR) == Time slicing

Round-Robin 알고리즘은 job들을 정해진 Time slice로 쪼개어 번갈아가며 수행하는 알고리즘이다.

![image](/images/2023-5/CPUScheduling/RR.jpeg)

(Time slice는 2초라고 가정)

모든 job들을 2초단위로 쪼개어 번갈아가며 수행하는 모습이다. 이렇게 하면 평균 Response time은 (0 + 2 + 4) / 3 = 2 로 PSJF보다 더 나은 Response time을 가진다. 다만 RR 알고리즘은 Turnaround time에 있어서는 최악인 것을 생각해볼 수 있다. 

지금까지 살펴본 job들은 모두 CPU 작업만 수행하는 job들이었다. 하지만 현실의 job들은 CPU 작업 뿐만아니라 I/O 작업도 수행하고, 이에따라 job이 block 되는 경우도 있다. 

![image](/images/2023-5/CPUScheduling/IO.jpeg)

job이 I/O 작업을 위해 block되고 그동안 CPU가 놀고있는 모습이다. 우리는 이제 이렇게 job이 block 되어 CPU가 낭비되는 문제를 해결해야한다. 

![image](/images/2023-5/CPUScheduling/IO2.jpeg)

이렇게 하나의 job이 block되어 있는 동안 다른 job을 수행하게 해주어야한다. 

또한 사실 Scheduler는 어떤 job이 얼마나 수행될지 알 길이 없기 때문에 STJ 나 STCF 같은 알고리즘은 현실적으로 어렵다는 문제점이 존재한다.

이러한 문제점들을 해결하기 위해 현존하는 CPU Scheduler들은 Multi-Level Feedback Queue (MLFQ)라는 개념을 차용한다. 이는 다음 글에서 다뤄보겠다.