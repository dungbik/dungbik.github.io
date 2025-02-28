---
title: 내일배움캠프 - 10일차
description:
slug: nb-til-10
date: 2025-02-28 00:00:00+0000
image:
weight: 1
categories:
  - til
tags:
  - 내일배움캠프
  - 본캠프
---

## 코드카타

### 나 잡아 봐라
> Q. 연인 코니와 브라운은 광활한 들판에서 ‘나 잡아 봐라’ 게임을 한다.  
> 이 게임은 브라운이 코니를 잡거나, 코니가 너무 멀리 달아나면 끝난다.  
> 게임이 끝나는데 걸리는 최소 시간을 구하시오. 
> 
> 조건은 다음과 같다.  
> 코니는 처음 위치 C에서 1초 후 1만큼 움직이고,  
> 이후에는 가속이 붙어 매 초마다 이전 이동 거리 + 1만큼 움직인다.  
> 즉 시간에 따른 코니의 위치는 C, C + 1, C + 3, C + 6, …이다.  
> 
> 브라운은 현재 위치 B에서 다음 순간 B – 1, B + 1, 2 * B 중 하나로 움직일 수 있다.  
> 코니와 브라운의 위치 p는 조건 0 <= x <= 200,000을 만족한다.  
> 브라운은 범위를 벗어나는 위치로는 이동할 수 없고, 코니가 범위를 벗어나면 게임이 끝난다

게임이 종료되는데 걸리는 최소 시간을 구하려면 모든 경우의 수를 확인해봐야한다. 이 문제의 경우 최소 시간을 구해야하기 떄문에 DFS 를 사용하기로 결정했다.  
그리고 잡았는지 여부는 같은 시간에 같은 위치에 있는 것이기 떄문에 시간에 따른 위치를 기록해야한다. 
따라서 key를 위치, value를 도달한 시간 집합 형태를 가지는 딕셔너리를 사용했다. 
여기서 딕셔너리와 집합을 이용한 이유는 데이터에 접근이 효율적이기 떄문이다. 

그 외에 신경쓸 것은 2가지가 더 있다.  
하나는 위치가 200000 보다 클 경우 잡지못한 채로 게임이 끝난 부분을 처리해주는 것이다.  
또 하나는 1초마다 움직이는 것이기 때문에 큐를 while queue: 가 아닌 for i in range(len(queue)): 를 사용한 것이다.  

``` python
from collections import deque

def catch_me(cony_loc, brown_loc):
    time = 0
    queue = deque()
    queue.append((brown_loc, 0))
    visited = [{} for _ in range(200001)]

    while cony_loc < 200000:
        cony_loc += time
        if time in visited[cony_loc]:
            return time

        for i in range(len(queue)):
            cur_loc, cur_time = queue.popleft()

            next_loc = cur_loc - 1
            next_time = cur_time + 1
            if next_loc >= 0 and next_time not in visited[next_loc]:
                visited[next_loc][next_time] = True
                queue.append((next_loc, next_time))

            next_loc = cur_loc + 1
            if next_loc <= 200000 and next_time not in visited[next_loc]:
                visited[next_loc][next_time] = True
                queue.append((next_loc, next_time))

            next_loc = cur_loc * 2
            if next_loc <= 200000 and next_time not in visited[next_loc]:
                visited[next_loc][next_time] = True
                queue.append((next_loc, next_time))

        time += 1

print("정답 = 3 / 현재 풀이 값 = ", catch_me(10,3))
print("정답 = 8 / 현재 풀이 값 = ", catch_me(51,50))
print("정답 = 28 / 현재 풀이 값 = ", catch_me(550,500))
```

## 외부 API 호출

외부 API를 호출이 필요하게 되어 예전에 개발할 때 자주 썼던 HttpClient로 코드를 작성하였다.  
되게 간단한 API를 호출하는데 코드가 복잡하고 가독성이 좋지 않았다.  
사용하기 편한 형태로 커스텀 메서드를 만들어서 사용할 수 있었으나, 더 좋은 대안이 있지 않을까 하고 찾아보았다.  
많은 대안이 있었는데 그 중에서 많이 검색되는 몇 가지 방법만 정리해보았다.

### FeignClient

- 인터페이스와 애노테이션으로 간단하게 사용 가능
- Spring Cloud와 잘 통합되며, 마이크로서비스 환경에서 많이 사용
- 동기 호출만 가능  

### WebClient (Spring WebFlux)

- 비동기 및 논블로킹 HTTP 클라이언트
- Mono와 Flux를 사용해 반응형 프로그래밍을 지원
- 높은 성능과 확장성
- 반응형 프로그래밍에 익숙하지 않다면 학습 곡선이 존재
- 설정이 상대적으로 복잡  

### RestTemplate

- 사용법이 간단
- Spring 5 이후로 deprecated
- 낮은 성능과 확장성  

### HttpClient (Apache)

- 세부적인 HTTP 요청을 컨트롤 가능
- 상대적으로 복잡한 코드 작성 필요

### OkHttp

- 경량화된 HTTP 클라이언트
- 높은 성능과 간단한 사용법
- 비동기 요청을 간단히 처리 가능
- Spring Boot에 사용시 추가 설정 필요

### Retrofit

- OkHttp 위에 선언형 방식으로 동작함
- Feign과 비슷한 스타일
- Spring Boot에 사용시 추가 설정 필요

현재 상황에서 간단한 API 동기 호출만 필요하고 간단하게 사용할 수 있는 것이 중요하기 때문에에 FeignClient를 사용하기로 결정하였다.
