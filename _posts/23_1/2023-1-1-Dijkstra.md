---
title : "다익스트라 알고리즘(Dijkstra Algorithm)"
comments : true
sidebar_main: true
categories:
  - Algorithm
tags:
  - Cpp
classes: wide
---

## 개요
Dijkstra Algorithm은 최단경로(shortest path)를 탐색하는 알고리즘이다. 

## 동작
알고리즘은 다음 순서로 동작한다. <br/>
1 ) 거리 테이블을 충분히 큰 값으로 초기화한다. <br/>
2 ) 거리 테이블의 시작 정점 부분에 0을 저장하면서 시작 정점을 우선 순위 큐에 넣는다. <br/>
3 ) 우선 순위 큐에서 정점을 하나 꺼내 인접한 간선들을 확인한다. 만약 현재 큐에서 꺼낸 정점까지의 비용과 간선의 비용을 더한 값이 거리 테이블의 해당 정점부분보다 작다면 거리 테이블을 업데이트하고 해당 정점을 큐에 넣는다.<br/>
4 ) 큐가 빌 때까지 3 )을 반복한다. 알고리즘이 끝나면 거리 테이블에 시작 정점으로 부터 모든 정점까지의 최소 비용이 저장되어있다.<br/>

## 구현
우선 순위 큐(Priority Queue)를 이용해 구현한다.
> [문제 : 백준 1238번 - 파티](https://www.acmicpc.net/problem/1238)

```c++
#include <iostream>
#include <queue>
#include <vector>

#define MAX 99'999'999;

using namespace std;

int N, M, X, ans;
vector<pair<int, int>> Edges[2][1010];  // Edges[0] : forward, Edges[1] : reverse
int dist[2][1010];  // dist[0] : forward, dist[1] : reverse
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>>
    pq;

void dijkstra(int id) {  // dijkstra algorithm
  dist[id][X] = 0;

  pq.push({0, X});
  while (!pq.empty()) {
    int curCost = pq.top().first;
    int curVertex = pq.top().second;
    pq.pop();
    if (curCost > dist[id][curVertex]) continue;
    for (auto &i : Edges[id][curVertex]) {
      int nextCost = i.second;
      int nextVertex = i.first;
      if (curCost + nextCost < dist[id][nextVertex]) {
        dist[id][nextVertex] = curCost + nextCost;
        pq.push({curCost + nextCost, nextVertex});
      }
    }
  }
}

int main(void) {
  ios::sync_with_stdio(false);
  cin.tie(NULL);

  int a, b, c;
  cin >> N >> M >> X;

  for (int i = 1; i <= N; i++) {  // initialize
    dist[0][i] = MAX;
    dist[1][i] = MAX;
  }

  for (int i = 0; i < M; i++) {
    cin >> a >> b >> c;
    Edges[0][a].push_back({b, c});  // forward graph
    Edges[1][b].push_back({a, c});  // reverse graph
  }

  dijkstra(0);
  dijkstra(1);

  for (int i = 1; i <= N; i++)  // find max value
    if (dist[0][i] + dist[1][i] > ans) ans = dist[0][i] + dist[1][i];

  cout << ans << "\n";

  return 0;
}
```

이 문제에서는 돌아가는 최소 경로까지 구해야 하여 간선의 방향들을 뒤집은 그래프에 대해 알고리즘을 한번 더 진행한다.

## 정확성 증명
다익스트라 알고리즘의 정확성은 수학적 귀납법을 통해 증명한다.

명제 : 다익스트라 알고리즘은 모든 정점에 대해 최단 경로를 보장한다.
V : 정점들의 집합, S : s(시작 정점)으로부터 현재까지 최단경로를 찾은 정점들의 집합
1. base : \|S\| = 1 일때 정점이 하나만 존재하기 때문에 최단경로의 가중치는 0 또는 self loop의 가중치로 최단경로임이 자명하다.
2. step : \|S\| = k - 1일 때 S의 모든 정점 v에 대해 s로부터 v까지의 최단경로를 dist[v]라고 하자. \|S\| = k일 때를 생각하면 V-S의 정점 중 dist[]의 값을 최소로 하는 정점을 u라고 하자. dist[u]가 s로부터 u까지 가는 최단경로(P)임을 증명하면 된다. 만약 dist[u]가 P가 아니라고 가정하면 dist[u] > P 가 성립해야한다. 하지만 그럴 경우 P는 dist[u]와 다른 경로여야 하기 때문에 dist[u]가 거치지 않는 최소 1개의 V-S에 속하는 정점을 거쳐간다. 이 정점을 u'이라고 하면 dist[u'] <= P < dist[u]가 성립해야하지만 dist[u]는 최소를 선택한 것이므로 이는 모순이고 따라서 dist[u] = P가 성립한다.

## 시간복잡도 : O(ElogV) (E : 간선의 수, V : 정점의 수)
모든 정점들에 대해서 최소 힙으로 추출하는데 O(logV), 각 정점들마다 인접한 모든 간선을 확인하므로 O(E)만큼 걸리므로 총 O(ElogV)의 시간복잡도를 가진다.

## 한계
다익스트라 알고리즘은 모든 간선의 비용이 양수일때만 최소비용을 찾을 수 있다. 음의 가중치를 가진 간선이 있다면 "벨만-포드 알고리즘"을 사용해야한다.